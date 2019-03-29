---
title: "scheduling"
date: 2019-03-11 18:25
---



# 资源数据同步
目前集群资源数据的管理和同步，Hadoop 从开始到 Yarn 基本都采用的是 Master-Slave-Heartbeat 模式，数据存储在 Master 内存中，数据更新并不一定是及时实时的，比如某个 Slave 上的container运行完毕，这个时候不是立即通知 Master，而是通过心跳来通知的。

集群数据资源的表示，在Master内部需要通过容器结构来维护所有的Slave资源数据信息，在Yarn中才通过 RMNode 结构来表示每个 NodeManager，RMContainer 来表示每个 NodeManager 上的 Container. 

ApplicationMasterProtocol#allocate, 这个函数取名叫`分配`，同时负责获取和释放容器资源。

# DRF 算法


# Yarn FairScheduler


## FairScheduler
1. 资源公平共享， 支持应用按照 FIFO, Fair, DRF 策略调度。
2. 支持资源抢占， 调度器有一个update线程定期检查是否进行资源抢占，当某个队列中有剩余资源时，调度器会将这些资源共享给其他队列，当该队列中有新的应用程序提交时，调度器要为它收回资源。同时为了尽可能降低不必要的资源浪费，调度器采用 先等待再强制回收 的策略，即如果等待一段时间后尚有资源未归还，则会进行资源抢占，从那些超额使用的队列中杀死一部分任务，进而释放资源。
3. 负载均衡, 提供一个基于任务数目的负载均衡机制，尽可能将系统中的任务均匀分配到各个节点上。且用户可以设计自己的负载均衡。
4. 调度策略配置灵活，每个队列可单独配置调度策略， FIFO, Fair, DRF
5. 提高小应用的相应时间。采用max-min 公平算法，小作业可快速获取资源并运行完成。


FairScheduler
```java
synchronized void attemptScheduling(FSSchedulerNode node) {
    // 检查当前调度器是否可用
    if (rmContext.isWorkPreservingRecoveryEnabled()
        && !rmContext.isSchedulerReadyForAllocatingContainers()) {
      return;
    }

    final NodeId nodeID = node.getNodeID();
    // Node 可能刚刚被移出了
    if (!nodes.containsKey(nodeID)) {
      // The node might have just been removed while this thread was waiting
      // on the synchronized lock before it entered this synchronized method
      LOG.info("Skipping scheduling as the node " + nodeID +
          " has been removed");
      return;
    }

    // Assign new containers...
    // 1. Check for reserved applications
    // 2. Schedule if there are no reservations
    // Yarn开启了资源保留机制，首先检查该节点是否为某个APP保留资源，如果是，则直接选择该APP分配容器
    FSAppAttempt reservedAppSchedulable = node.getReservedAppSchedulable();
    if (reservedAppSchedulable != null) {
      Priority reservedPriority = node.getReservedContainer().getReservedPriority();
      FSQueue queue = reservedAppSchedulable.getQueue();
      // 如果Node保留的资源还是无法满足APP所在的queue，则释放资源在该Node的预留机制，然后重新选择APP调度
      if (!reservedAppSchedulable.hasContainerForNode(reservedPriority, node)
          || !fitsInMaxShare(queue,
          node.getReservedContainer().getReservedResource())) {
        // Don't hold the reservation if app can no longer use it
        LOG.info("Releasing reservation that cannot be satisfied for application "
            + reservedAppSchedulable.getApplicationAttemptId()
            + " on node " + node);
        reservedAppSchedulable.unreserve(reservedPriority, node);
        reservedAppSchedulable = null;
      } else {
        // Reservation exists; try to fulfill the reservation
        if (LOG.isDebugEnabled()) {
          LOG.debug("Trying to fulfill reservation for application "
              + reservedAppSchedulable.getApplicationAttemptId()
              + " on node: " + node);
        }
        // 预留资源已经可以满足APP的需求，直接分配容器
        node.getReservedAppSchedulable().assignReservedContainer(node);
      }
    }
    if (reservedAppSchedulable == null) {
      // No reservation, schedule at queue which is farthest below fair share
      int assignedContainers = 0;
      while (node.getReservedContainer() == null) {
        boolean assignedContainer = false;
        if (!queueMgr.getRootQueue().assignContainer(node).equals(
            Resources.none())) {
          assignedContainers++;
          assignedContainer = true;
        }
        if (!assignedContainer) { break; }
        if (!assignMultiple) { break; }
        if ((assignedContainers >= maxAssign) && (maxAssign > 0)) { break; }
      }
    }
    updateRootQueueMetrics();
  }

```

FSAppAteempt

```java
private Resource assignContainer(FSSchedulerNode node, boolean reserved) {
    if (LOG.isDebugEnabled()) {
      LOG.debug("Node offered to app: " + getName() + " reserved: " + reserved);
    }

    Collection<Priority> prioritiesToTry = (reserved) ?
        Arrays.asList(node.getReservedContainer().getReservedPriority()) :
        getPriorities();

    // For each priority, see if we can schedule a node local, rack local
    // or off-switch request. Rack of off-switch requests may be delayed
    // (not scheduled) in order to promote better locality.
    synchronized (this) {
      for (Priority priority : prioritiesToTry) {
        if (getTotalRequiredResources(priority) <= 0 ||
            !hasContainerForNode(priority, node)) {
          continue;
        }

        addSchedulingOpportunity(priority);

        // Check the AM resource usage for the leaf queue
        // 有可能App所在的队列已经无法使用更多的资源了，不予以分配
        if (getLiveContainers().size() == 0 && !getUnmanagedAM()) {
          if (!getQueue().canRunAppAM(getAMResource())) {
            return Resources.none();
          }
        }

        ResourceRequest rackLocalRequest = getResourceRequest(priority,
            node.getRackName());
        ResourceRequest localRequest = getResourceRequest(priority,
            node.getNodeName());

        if (localRequest != null && !localRequest.getRelaxLocality()) {
          LOG.warn("Relax locality off is not supported on local request: "
              + localRequest);
        }

        NodeType allowedLocality;
        if (scheduler.isContinuousSchedulingEnabled()) {
          allowedLocality = getAllowedLocalityLevelByTime(priority,
              scheduler.getNodeLocalityDelayMs(),
              scheduler.getRackLocalityDelayMs(),
              scheduler.getClock().getTime());
        } else {
          allowedLocality = getAllowedLocalityLevel(priority,
              scheduler.getNumClusterNodes(),
              scheduler.getNodeLocalityThreshold(),
              scheduler.getRackLocalityThreshold());
        }

        if (rackLocalRequest != null && rackLocalRequest.getNumContainers() != 0
            && localRequest != null && localRequest.getNumContainers() != 0) {
          return assignContainer(node, localRequest,
              NodeType.NODE_LOCAL, reserved);
        }

        if (rackLocalRequest != null && !rackLocalRequest.getRelaxLocality()) {
          continue;
        }

        if (rackLocalRequest != null && rackLocalRequest.getNumContainers() != 0
            && (allowedLocality.equals(NodeType.RACK_LOCAL) ||
            allowedLocality.equals(NodeType.OFF_SWITCH))) {
          return assignContainer(node, rackLocalRequest,
              NodeType.RACK_LOCAL, reserved);
        }

        ResourceRequest offSwitchRequest =
            getResourceRequest(priority, ResourceRequest.ANY);
        if (offSwitchRequest != null && !offSwitchRequest.getRelaxLocality()) {
          continue;
        }

        if (offSwitchRequest != null &&
            offSwitchRequest.getNumContainers() != 0) {
          if (!hasNodeOrRackLocalRequests(priority) ||
              allowedLocality.equals(NodeType.OFF_SWITCH)) {
            return assignContainer(
                node, offSwitchRequest, NodeType.OFF_SWITCH, reserved);
          }
        }
      }
    }
    return Resources.none();
  }

```
FSAppAteempt
```Java

private Resource assignContainer(
    FSSchedulerNode node, ResourceRequest request, NodeType type,
    boolean reserved) {

    // How much does this request need?
    Resource capability = request.getCapability();

    // How much does the node have?
    Resource available = node.getAvailableResource();

    // 新建一个可序列化记录Contaniner
    Container container = null;
    if (reserved) {
        container = node.getReservedContainer().getContainer();
    } else {
        container = createContainer(node, capability, request.getPriority());
    }

    // Can we allocate a container on this node?
    if (Resources.fitsIn(capability, available)) {
        // 为该请求创建一个RMContainer容器对象
        RMContainer allocatedContainer =
            allocate(type, node, request.getPriority(), request, container);
        if (allocatedContainer == null) {//分配不成功
            // Did the application need this resource?
            // 如果该App在该Node上预留，则释放开始保存的预留资源
            if (reserved) {
                unreserve(request.getPriority(), node);
            }
            return Resources.none();
        }
        // 分配成功，则释放开始保存的预留资源
        // If we had previously made a reservation, delete it
        if (reserved) {
            unreserve(request.getPriority(), node);
        }

        // 对Node的资源进行更新，包括资源量，容器数目，以及添加运行的容器，需要枷锁
        node.allocateContainer(allocatedContainer);

        // If this container is used to run AM, update the leaf queue's AM usage
        if (getLiveContainers().size() == 1 && !getUnmanagedAM()) {
            getQueue().addAMResourceUsage(container.getResource());
            setAmRunning(true);
        }

        return container.getResource();
    } else {
        if (!FairScheduler.fitsInMaxShare(getQueue(), capability)) {
            return Resources.none();
        }
        // 当前该App不适合Node，因此做预留操作
        // The desired container won't fit here, so reserve
        reserve(request.getPriority(), node, container, reserved);

        return FairScheduler.CONTAINER_RESERVED;
    }
}
synchronized public RMContainer allocate(NodeType type, FSSchedulerNode node,
    Priority priority, ResourceRequest request,
    Container container) {
    // 更新允许的局部感知粒度
    NodeType allowed = allowedLocalityLevel.get(priority);
    if (allowed != null) {
        if (allowed.equals(NodeType.OFF_SWITCH) &&
            (type.equals(NodeType.NODE_LOCAL) ||
                type.equals(NodeType.RACK_LOCAL))) {
        this.resetAllowedLocalityLevel(priority, type);
        }
        else if (allowed.equals(NodeType.RACK_LOCAL) &&
            type.equals(NodeType.NODE_LOCAL)) {
        this.resetAllowedLocalityLevel(priority, type);
        }
    }
    // 检查资源需求量，因为该需求量可能会被AM通过 allocate 更改
    // Required sanity check - AM can call 'allocate' to update resource 
    // request without locking the scheduler, hence we need to check
    if (getTotalRequiredResources(priority) <= 0) {
        return null;
    }

    // Create RMContainer
    RMContainer rmContainer = new RMContainerImpl(container, 
        getApplicationAttemptId(), node.getNodeID(),
        appSchedulingInfo.getUser(), rmContext);

    // 该列表中存放的就是已经被调度器调度好的容器，AM会通过 allocate 进行获取; 当然，还有一种情况是启动AM也需要容器，那是通过 RMAppAttempt 的 ScheduleTransition 也会调用 allocate 获取 AM Container。
    newlyAllocatedContainers.add(rmContainer);
    liveContainers.put(container.getId(), rmContainer);    

    // Update consumption and track allocations
    // 跟踪被分配的资源, 并更新该App的资源消耗
    List<ResourceRequest> resourceRequestList = appSchedulingInfo.allocate(
        type, node, priority, request, container);
    Resources.addTo(currentConsumption, container.getResource());

    // Update resource requests related to "request" and store in RMContainer
    ((RMContainerImpl) rmContainer).setResourceRequests(resourceRequestList);

    // 通知RMContainer 容器新建了
    rmContainer.handle(
        new RMContainerEvent(container.getId(), RMContainerEventType.START));
    return rmContainer;
}
public Container createContainer(
      FSSchedulerNode node, Resource capability, Priority priority) {

    NodeId nodeId = node.getRMNode().getNodeID();
    ContainerId containerId = BuilderUtils.newContainerId(
        getApplicationAttemptId(), getNewContainerId());

    // Create the container， 这里的container是一个可序列化的记录
    Container container =
        BuilderUtils.newContainer(containerId, nodeId, node.getRMNode()
            .getHttpAddress(), capability, priority, null);

    return container;
  }

  private void reserve(Priority priority, FSSchedulerNode node,
      Container container, boolean alreadyReserved) {
    if (!alreadyReserved) {
      getMetrics().reserveResource(getUser(), container.getResource());
      RMContainer rmContainer =
          super.reserve(node, priority, null, container);
      node.reserveResource(this, priority, rmContainer);
    } else {
      RMContainer rmContainer = node.getReservedContainer();
      super.reserve(node, priority, rmContainer, container);
      node.reserveResource(this, priority, rmContainer);
    }
  }

  /**
   * Remove the reservation on {@code node} at the given {@link Priority}.
   * This dispatches SchedulerNode handlers as well.
   */
  public void unreserve(Priority priority, FSSchedulerNode node) {
    RMContainer rmContainer = node.getReservedContainer();
    unreserveInternal(priority, node);
    node.unreserveResource(this);
    getMetrics().unreserveResource(
        getUser(), rmContainer.getContainer().getResource());
  }
```

SchedulerNode

```Java
public synchronized void allocateContainer(RMContainer rmContainer) {
    Container container = rmContainer.getContainer();
    deductAvailableResource(container.getResource());
    ++numContainers;

    launchedContainers.put(container.getId(), rmContainer);
  }

private synchronized void deductAvailableResource(Resource resource) {
    if (resource == null) {
      return;
    }
    Resources.subtractFrom(availableResource, resource);
    Resources.addTo(usedResource, resource);
  }
```



FairScheduler#UpdateThread
```Java
private class UpdateThread extends Thread {

    @Override
    public void run() {
      while (!Thread.currentThread().isInterrupted()) {
        try {
          Thread.sleep(updateInterval);
          long start = getClock().getTime();
          update();
          preemptTasksIfNecessary();
          long duration = getClock().getTime() - start;
          fsOpDurations.addUpdateThreadRunDuration(duration);
        } catch (InterruptedException ie) {
          LOG.warn("Update thread interrupted. Exiting.");
          return;
        } catch (Exception e) {
          LOG.error("Exception in fair scheduler UpdateThread", e);
        }
      }
    }
}
  protected synchronized void update() {
    long start = getClock().getTime();
    updateStarvationStats(); // Determine if any queues merit preemption

    FSQueue rootQueue = queueMgr.getRootQueue();

    // Recursively update demands for all queues
    rootQueue.updateDemand();

    rootQueue.setFairShare(clusterResource);
    // Recursively compute fair shares for all queues
    // and update metrics
    rootQueue.recomputeShares();
    updateRootQueueMetrics();

    if (LOG.isDebugEnabled()) {
      if (--updatesToSkipForDebug < 0) {
        updatesToSkipForDebug = UPDATE_DEBUG_FREQUENCY;
        LOG.debug("Cluster Capacity: " + clusterResource +
            "  Allocations: " + rootMetrics.getAllocatedResources() +
            "  Availability: " + Resource.newInstance(
            rootMetrics.getAvailableMB(),
            rootMetrics.getAvailableVirtualCores()) +
            "  Demand: " + rootQueue.getDemand());
      }
    }

    long duration = getClock().getTime() - start;
    fsOpDurations.addUpdateCallDuration(duration);
  }
```
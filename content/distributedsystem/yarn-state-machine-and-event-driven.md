---
title: "yarn-state-machine-and-event-driven"
date: 2019-03-02 13:49
---


# Event-Driven State Machine
Yarn 整个框架代码都建立在状态机和事件驱动的模型之上，

## Finite State Machine

![Finite-State Machines: Theory and Implementation](https://gamedevelopment.tutsplus.com/tutorials/finite-state-machines-theory-and-implementation--gamedev-11867)

## Event-Driven System

处理请Ⅾ 会作为事件进入系统， 由中央异步调度器（ AsyncDispatcher） 负责传递给相应事件调度器（ Event Handler）。 该事件调度器可能将该事件转发另外一个事件调度器， 也可能交给一个带有有限状态机的事件处理器，其处理结果也以事件的形式输出给中央异步调度器。 而新的事件会再次被中央异步调度器转发给下一个事件调度器，直到处理完成。


# Yarn Event-Driven 
YARN 采用了基于事件驱动的并发模型，该模型能够大大增强并发性，提供系统性能。Yarn 的事件驱动模型如下图所示:

<div align="center"><img src="/static/images/DistributedSystem/yarn-fsm-and-ed/YarnEventDrivenStateMachine.png" style="width:700px;height:500px;">
<caption><center> YarnEventDrivenStateMachine </center></caption></div>

在 YARN 中， 所有核心服务都是要使用中央异步调度器， 包括 ResourceManager、NodeManager、 MRAppMaster(MapReduce 应用程序的ApplicationMaster)等， 它们维护了事先注册的事件与事件处理器， 并根据接收的事件类型驱动服务的运行。


Yarn 的事件库位于 `org.apache.hadoop.yarn.event` 包下面。是整个Yarn的模板库，对外提供 抽象模板类 `AbstractEvent<TYPE extends Enum<TYPE>>`， 用户通过该抽象类实现自己的事件类(事件类型类是枚举类型), 另一个接口模板 `EventHandler<T extends Event>`，用户通过该接口实现自己的事件处理器， 接口 `Dispatcher`，该库下面自己实现了一个 `AsyncDispatcher` 可以直接使用。以下是事件库的类图结构：

<div align="center"><img src="/static/images/DistributedSystem/yarn-fsm-and-ed/YarnEventUML.jpg" style="width:700px;height:500px;">
<caption><center> YarnEventUML </center></caption></div>

当使用 YARN 事件库时， 通常先要定义一个中央异步调度器 AsyncDispatcher，负责事件的处理与转发， 然后根据实际业务定义一系列Event 和 EventHandler， 并注册到中央异步调度器中以实现事件统一管理和调度。 以 MRAppMaster为例，它内部包含一个中央异步调度器 AsyncDispatcher， 并注册TaskAttemptEvent/TaskAttemptImpl、 TaskEvent/TaskImpl、 JobEvent/JobImpl 等一系列事件 / 事件处理器，由中央异步调度器统一管理和调度。

使用Yarn库也很简单，比如 `org.apache.hadoop.yarn.server.resourcemanager.rmapp` 包下实现的就是 状态机 RMApp 的事件模型。

```Java
// 定义状态机处理器接口 RMApp
public interface RMApp extends EventHandler<RMAppEvent>

// 定义RMAppEvent 事件
public class RMAppEvent extends AbstractEvent<RMAppEventType>

public enum RMAppEventType {
  // Source: ClientRMService
}

```
我们来分析一下事件分发器 `AsynchDispatcher` 的实现。事件的处理都是通过分发器分发出去来处理的，分发器会把所有的事件和对应的处理器注册进来，当接收到相应的事件以后，就交给对应的处理器执行，事件产生和执行是异步的关系。那么如何使用该事件分发器呢？ 
 
 1. 需要注册事件类型类和该类型类对应的处理器。
 2. 产生事件后，通过 `AsyncDispatcher.getEventHandler()` 方法拿到一个 `GenericEventHandler` 实例，调用该实例的 `handle` 方法，即可将事件放入 `AsyncDispatcher` 进行分发处理。`AsyncDispatcher.getEventHandler().handle(Event);` 这是事件放入的地方。
## 事件处理器向中央异步处理器注册
以下是 ResourceManager 中注册各种事件处理器的代码。
```java
// Register event handler for NodesListManager
nodesListManager = new NodesListManager(rmContext);
rmDispatcher.register(NodesListManagerEventType.class, nodesListManager);
addService(nodesListManager);
rmContext.setNodesListManager(nodesListManager);

// Initialize the scheduler
scheduler = createScheduler();
scheduler.setRMContext(rmContext);
addIfService(scheduler);
rmContext.setScheduler(scheduler);

schedulerDispatcher = createSchedulerEventDispatcher();
addIfService(schedulerDispatcher);
rmDispatcher.register(SchedulerEventType.class, schedulerDispatcher);

// Register event handler for RmAppEvents
rmDispatcher.register(RMAppEventType.class,
    new ApplicationEventDispatcher(rmContext));

// Register event handler for RmAppAttemptEvents
rmDispatcher.register(RMAppAttemptEventType.class,
    new ApplicationAttemptEventDispatcher(rmContext));

// Register event handler for RmNodes
rmDispatcher.register(
    RMNodeEventType.class, new NodeEventDispatcher(rmContext));
......
rmAppManager = createRMAppManager();
// Register event handler for RMAppManagerEvents
rmDispatcher.register(RMAppManagerEventType.class, rmAppManager);

applicationMasterLauncher = createAMLauncher();
rmDispatcher.register(AMLauncherEventType.class,
    applicationMasterLauncher);
addService(applicationMasterLauncher);

```

## 中央异步处理器处理模型
`AsyncDispatcher` 的设计也是采用生产者-消费者模型，这里的消费者是内部的 `eventHandlingThread` 单线程，生产者是内部的 `GenericEventHandler`， 也就是上文所说的事件放入的入口，当然这里并不是真实的生产者，只是生产者的代理入口。下图是 `AsyncDispatcher` 的生产消费模型。

<div align="center"><img src="/static/images/DistributedSystem/yarn-fsm-and-ed/YarnAsyncDispatcher.jpg" style="width:700px;height:500px;">
<caption><center> YarnAsyncDispatcher </center></caption></div>

需要注意的是，这里的 EventHandler 其实不一定是一个真实的处理器，他可能只是对某种类型处理器的包装处理器，比如我们有很多的 RMAppImpl 处理器对象，从 `AsyncDispatcher` 的 `register` 接口来看，它只维护某种事件类型和对应事件处理器对象，但是我们知道 RMAppEventType 其实对应了很多个 RMAppImpl 对象的，这样怎么能把对应 RMAppImpl 的事件转发给它处理，显然，可以设计一个包装处理器，它接受 RMAppEeventType事件，但是处理的时候会根据 Event 中的 RMAppID 再次分发给对应的 RMAppImpl 来处理即可。在 Yarn ResourceManager 类中， RMApp，RMAppAttempt，RMNode， ResourceScheduler 都是被包装的，分别是 ApplicationEventDispatcher, ApplicationAttemptEventDispatcher, NodeEventDispatcher， SchedulerEventDispatcher。

这里比较有意思的是， AsyncDispatcher 中的 EventHandler 还可以是一个 AsyncDispatcher， 比如SchedulerEventDispatcher 其实也实现了一个 Dispatcher 的模型。这种结构是可以递归嵌套的，这种代码最好不要嵌套过多，否则影响可读性。

```java

  @Private
  public static class SchedulerEventDispatcher extends AbstractService
      implements EventHandler<SchedulerEvent> {

    private final ResourceScheduler scheduler;
    private final BlockingQueue<SchedulerEvent> eventQueue =
      new LinkedBlockingQueue<SchedulerEvent>();
    private volatile int lastEventQueueSizeLogged = 0;
    private final Thread eventProcessor;
    private volatile boolean stopped = false;
    private boolean shouldExitOnError = false;

    public SchedulerEventDispatcher(ResourceScheduler scheduler) {
      super(SchedulerEventDispatcher.class.getName());
      this.scheduler = scheduler;
      this.eventProcessor = new Thread(new EventProcessor());
      this.eventProcessor.setName("ResourceManager Event Processor");
    }

    @Override
    protected void serviceInit(Configuration conf) throws Exception {
      this.shouldExitOnError =
          conf.getBoolean(Dispatcher.DISPATCHER_EXIT_ON_ERROR_KEY,
            Dispatcher.DEFAULT_DISPATCHER_EXIT_ON_ERROR);
      super.serviceInit(conf);
    }

    @Override
    protected void serviceStart() throws Exception {
      this.eventProcessor.start();
      super.serviceStart();
    }

    private final class EventProcessor implements Runnable {
      @Override
      public void run() {

        SchedulerEvent event;

        while (!stopped && !Thread.currentThread().isInterrupted()) {
          try {
            event = eventQueue.take();
          } catch (InterruptedException e) {
            LOG.error("Returning, interrupted : " + e);
            return; // TODO: Kill RM.
          }

          try {
            scheduler.handle(event);
          } catch (Throwable t) {
            // An error occurred, but we are shutting down anyway.
            // If it was an InterruptedException, the very act of 
            // shutdown could have caused it and is probably harmless.
            if (stopped) {
              LOG.warn("Exception during shutdown: ", t);
              break;
            }
            LOG.fatal("Error in handling event type " + event.getType()
                + " to the scheduler", t);
            if (shouldExitOnError
                && !ShutdownHookManager.get().isShutdownInProgress()) {
              LOG.info("Exiting, bbye..");
              System.exit(-1);
            }
          }
        }
      }
    }

    @Override
    protected void serviceStop() throws Exception {
      this.stopped = true;
      this.eventProcessor.interrupt();
      try {
        this.eventProcessor.join();
      } catch (InterruptedException e) {
        throw new YarnRuntimeException(e);
      }
      super.serviceStop();
    }

    @Override
    public void handle(SchedulerEvent event) {
      try {
        int qSize = eventQueue.size();
        if (qSize != 0 && qSize % 1000 == 0
            && lastEventQueueSizeLogged != qSize) {
          lastEventQueueSizeLogged = qSize;
          LOG.info("Size of scheduler event-queue is " + qSize);
        }
        int remCapacity = eventQueue.remainingCapacity();
        if (remCapacity < 1000) {
          LOG.info("Very low remaining capacity on scheduler event queue: "
              + remCapacity);
        }
        this.eventQueue.put(event);
      } catch (InterruptedException e) {
        LOG.info("Interrupted. Trying to exit gracefully.");
      }
    }
  }

  public void handleTransitionToStandBy() {
    if (rmContext.isHAEnabled()) {
      try {
        // Transition to standby and reinit active services
        LOG.info("Transitioning RM to Standby mode");
        transitionToStandby(true);
        adminService.resetLeaderElection();
        return;
      } catch (Exception e) {
        LOG.fatal("Failed to transition RM to Standby mode.");
        ExitUtil.terminate(1, e);
      }
    }
  }

  @Private
  public static final class ApplicationEventDispatcher implements
      EventHandler<RMAppEvent> {

    private final RMContext rmContext;

    public ApplicationEventDispatcher(RMContext rmContext) {
      this.rmContext = rmContext;
    }

    @Override
    public void handle(RMAppEvent event) {
      ApplicationId appID = event.getApplicationId();
      RMApp rmApp = this.rmContext.getRMApps().get(appID);
      if (rmApp != null) {
        try {
          rmApp.handle(event);
        } catch (Throwable t) {
          LOG.error("Error in handling event type " + event.getType()
              + " for application " + appID, t);
        }
      }
    }
  }

  @Private
  public static final class ApplicationAttemptEventDispatcher implements
      EventHandler<RMAppAttemptEvent> {

    private final RMContext rmContext;

    public ApplicationAttemptEventDispatcher(RMContext rmContext) {
      this.rmContext = rmContext;
    }

    @Override
    public void handle(RMAppAttemptEvent event) {
      ApplicationAttemptId appAttemptID = event.getApplicationAttemptId();
      ApplicationId appAttemptId = appAttemptID.getApplicationId();
      RMApp rmApp = this.rmContext.getRMApps().get(appAttemptId);
      if (rmApp != null) {
        RMAppAttempt rmAppAttempt = rmApp.getRMAppAttempt(appAttemptID);
        if (rmAppAttempt != null) {
          try {
            rmAppAttempt.handle(event);
          } catch (Throwable t) {
            LOG.error("Error in handling event type " + event.getType()
                + " for applicationAttempt " + appAttemptId, t);
          }
        }
      }
    }
  }

  @Private
  public static final class NodeEventDispatcher implements
      EventHandler<RMNodeEvent> {

    private final RMContext rmContext;

    public NodeEventDispatcher(RMContext rmContext) {
      this.rmContext = rmContext;
    }

    @Override
    public void handle(RMNodeEvent event) {
      NodeId nodeId = event.getNodeId();
      RMNode node = this.rmContext.getRMNodes().get(nodeId);
      if (node != null) {
        try {
          ((EventHandler<RMNodeEvent>) node).handle(event);
        } catch (Throwable t) {
          LOG.error("Error in handling event type " + event.getType()
              + " for node " + nodeId, t);
        }
      }
    }
  }
```
 

## 中央异步处理器服务停止
`AsyncDispatcher` 在 Yarn 中也属于服务，服务关闭的时候，需要考虑是否清理正在排队的任务或者资源，`drainEventsOnStop` 是用来设置在服务停止的时候，是否将队列中的事件处理完毕再关闭。 如果设置为 true，那么就需要阻塞还在继续发生的事件进入队列 `blockNewEvents`，同时使用同步信号 `waitForDrained` 等待队列中剩余的事件被处理完成。这里需要注意的一点是，队列中的事件处理不一定能够顺利完成，这是不确定的，万一一直处理不完，则一直无法停止服务了，甚至导致无法退出服务进程。因此需要设置超时时间 `DISPATCHER_DRAIN_EVENTS_TIMEOUT`，如果等待事件过长，还是强制退出服务。

```java
class GenericEventHandler implements EventHandler<Event> {
    public void handle(Event event) {
      if (blockNewEvents) {
        return;
      }
      drained = false;

      /* all this method does is enqueue all the events onto the queue */
      int qSize = eventQueue.size();
      if (qSize != 0 && qSize % 1000 == 0
          && lastEventQueueSizeLogged != qSize) {
        lastEventQueueSizeLogged = qSize;
        LOG.info("Size of event-queue is " + qSize);
      }
      int remCapacity = eventQueue.remainingCapacity();
      if (remCapacity < 1000) {
        LOG.warn("Very low remaining capacity in the event-queue: "
            + remCapacity);
      }
      try {
        eventQueue.put(event);
      } catch (InterruptedException e) {
        if (!stopped) {
          LOG.warn("AsyncDispatcher thread interrupted", e);
        }
        // Need to reset drained flag to true if event queue is empty,
        // otherwise dispatcher will hang on stop.
        drained = eventQueue.isEmpty();
        throw new YarnRuntimeException(e);
      }
    };
  }
  
  protected void serviceStop() throws Exception {
    if (drainEventsOnStop) {
      blockNewEvents = true;
      LOG.info("AsyncDispatcher is draining to stop, igonring any new events.");
      long endTime = System.currentTimeMillis() + getConfig()
          .getLong(YarnConfiguration.DISPATCHER_DRAIN_EVENTS_TIMEOUT,
              YarnConfiguration.DEFAULT_DISPATCHER_DRAIN_EVENTS_TIMEOUT);

      synchronized (waitForDrained) {
        while (!drained && eventHandlingThread != null
            && eventHandlingThread.isAlive()
            && System.currentTimeMillis() < endTime) {
          waitForDrained.wait(1000);
          LOG.info("Waiting for AsyncDispatcher to drain. Thread state is :" +
              eventHandlingThread.getState());
        }
      }
    }
    stopped = true;
    if (eventHandlingThread != null) {
      eventHandlingThread.interrupt();
      try {
        eventHandlingThread.join();
      } catch (InterruptedException ie) {
        LOG.warn("Interrupted Exception while stopping", ie);
      }
    }

    // stop all the components
    super.serviceStop();
  }
```

# Yarn State Machine

Yarn 采用工厂模式和范型，也是一个模板类库，对外提供了一个底层的状态机工厂模版 `StateMachineFactory<OPERAND, STATE extends Enum<STATE>,EVENTTYPE extends Enum<EVENTTYPE>, EVENT>`，代码位于 `org.apache.hadoop.yarn.state` 包之下，对外提供了两个需要由用户实现的接口 `SingleArcTransition`和`MultipleArcTransition`, `StateMachine` 会通过调用这两个接口的实现类的 `transition` 方法，实现内部状态转移。 值得注意的是， `StateMachine` 接口也不是由用户实现的，而是定义在 `StateMachineFactory` 内部的一个内部类 `InternalStateMachine` 实现。用户仅仅需要实现 `SingleArcTransition`和`MultipleArcTransition` 两个接口。状态机的状态还是由 `StateMachineFactory#InternalStateMachine` 来维护。

下图是Yarn 状态机库的类图关系：

<div align="center"><img src="/static/images/DistributedSystem/yarn-fsm-and-ed/YarnStateMachineUML.jpg" style="width:700px;height:500px;">
<caption><center> YarnStateMachineUML </center></caption></div>

`SingleArcTransition`和`MultipleArcTransition` 以及 `StateMachine` 都是接口类，定义在 `StateMachineFactory` 外部，其他的各种类和接口都是定义在 `StateMachineFactory` 内部的。


`SingleInternalArc` 和  `MultipleInternalArc` 这两个实现 `Transition` 接口的类，内部组合了一个成员 `hook`，这个就是用户要具体实现的 `SingleArcTransition` 和 `MultipleArcTransition` 类。然后 这俩种 `Transition` 又被组合进入 `ApplicableSingleOrMultipleTransition` 这个实现 `ApplicableTransition` 接口的类，接着 `ApplicableTransition` 又被组合到了 `TransitionListNode` 内部，而 `TransitionListNode` 是一个链表结构。 最后和 `StateMachineFactory` 发生直接联系的是 `TransitionListNode`，他是 `StateMachineFactory` 的一个成员。除了该成员，`StateMachineFactory` 还有 `Map<STATE, Map<EVENTTYPE, Transition<OPERAND, STATE, EVENTTYPE, EVENT>>> stateMachineTable` 表和一个布尔变量 `optimized`.

可以看到这个嵌套组合关系非常奇葩，目前我还不懂为啥要写成这样。。。对大致的类的引用关系有了解以后，再来看看这些状态机库都是怎么使用的。

Yarn 中实现了各种各样的具体的状态机，比如 RMApp，RMAppAttempt, RMNode, RMContainer, 具体类型的状态机需要根据事件作出处理，所以状态机一般都是继承了 EventHandler 的事件处理器，根据多用组合，少用继承的设计原则，这些具体类型的状态机（处理器）并不需要继承 StateMachine， 只是将 StateMachine 当作成员使用即可，这样每种状态机内部都会生成自己的 StateMachine 对象，注册完各种 Transition 以后，事件的处理和状态的转移代码全部都是复用 yarn 实现的状态机代码。

以 RMAppImpl 为例，先看构造状态机代码：

```java
private static final StateMachineFactory<RMAppImpl,
                                           RMAppState,
                                           RMAppEventType,
                                           RMAppEvent> stateMachineFactory
                               = new StateMachineFactory<RMAppImpl,
                                           RMAppState,
                                           RMAppEventType,
                                           RMAppEvent>(RMAppState.NEW)


     // Transitions from NEW state
    .addTransition(RMAppState.NEW, RMAppState.NEW,
        RMAppEventType.NODE_UPDATE, new RMAppNodeUpdateTransition())
    .addTransition(RMAppState.NEW, RMAppState.NEW_SAVING,
        RMAppEventType.START, new RMAppNewlySavingTransition())
    .....// 有很多转移加入
    .installTopology();

private final StateMachine<RMAppState, RMAppEventType, RMAppEvent>
                                                                 stateMachine;
this.stateMachine = stateMachineFactory.make(this);
    
```

首先调用 StateMachineFactory 的唯一的公开构造函数，做些初始化工作，比如初始状态。然后就开始调用 addTransition 了，这里的 addTransition 有多个重载。`addTransition` 采用是 builder pattern 也就是链式调用法(简单的说是每次调用对象方法后返回对象自己)。但是这里的链式调用每次返回的都是新的工厂而不是原来的工厂。这个设计也很奇葩。。我目前的水平确实看不懂。。。根据状态转移图可以把转移分成 单弧 和 多弧的，hook 可以为空.

1. 单个终态，单个事件类型，无 hook 动作.
2. 单个终态，多个事件类型，无 hook 动作.
3. 单个终态，多个事件类型. SingleArcTransition.
4. 单个终态，单个事件类型，SingleArcTransition.
5. 多个终态，单个事件类型，MultiArcTransition.

```Java
//1.
  public StateMachineFactory
             <OPERAND, STATE, EVENTTYPE, EVENT>
          addTransition(STATE preState, STATE postState, EVENTTYPE eventType) {
    return addTransition(preState, postState, eventType, null);
  }
//2.
  public StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT> addTransition(
      STATE preState, STATE postState, Set<EVENTTYPE> eventTypes) {
    return addTransition(preState, postState, eventTypes, null);
  }
//3.
  public StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT> addTransition(
      STATE preState, STATE postState, Set<EVENTTYPE> eventTypes,
      SingleArcTransition<OPERAND, EVENT> hook) {
    StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT> factory = null;
    for (EVENTTYPE event : eventTypes) {
      if (factory == null) {
        factory = addTransition(preState, postState, event, hook);
      } else {
        factory = factory.addTransition(preState, postState, event, hook);
      }
    }
    return factory;
  }
//4.
  public StateMachineFactory
             <OPERAND, STATE, EVENTTYPE, EVENT>
          addTransition(STATE preState, STATE postState,
                        EVENTTYPE eventType,
                        SingleArcTransition<OPERAND, EVENT> hook){
    return new StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT>
        (this, new ApplicableSingleOrMultipleTransition<OPERAND, STATE, EVENTTYPE, EVENT>
           (preState, eventType, new SingleInternalArc(postState, hook)));
  }
//5.
  public StateMachineFactory
             <OPERAND, STATE, EVENTTYPE, EVENT>
          addTransition(STATE preState, Set<STATE> postStates,
                        EVENTTYPE eventType,
                        MultipleArcTransition<OPERAND, EVENT, STATE> hook){
    return new StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT>
        (this,
         new ApplicableSingleOrMultipleTransition<OPERAND, STATE, EVENTTYPE, EVENT>
           (preState, eventType, new MultipleInternalArc(postStates, hook)));
  }
```
这里可以看到前面说的几个类的组合关系，嵌套了四层，TransitionsListNode --> ApplicableTransition --> Transition --> hook。
还可以看到，这里都使用了私有构造函数。 这个函数其实是采用头插法逆序建立了一个链表。后面加入的 Transition 通过 next 指针连接 前面加入的 Transition。

```java
private StateMachineFactory
      (StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT> that,
       ApplicableTransition<OPERAND, STATE, EVENTTYPE, EVENT> t) {
    this.defaultInitialState = that.defaultInitialState;
    // 头插法建立链表
    this.transitionsListNode 
        = new TransitionsListNode(t, that.transitionsListNode);
        
    this.optimized = false;
    this.stateMachineTable = null;
 }

```
下图是链表建立过程:

<div align="center"><img src="/static/images/DistributedSystem/yarn-fsm-and-ed/YarnStateMachineTransitionListNode.jpg" style="width:700px;height:500px;">
<caption><center> YarnStateMachineTransitionListNode </center></caption></div>


通过链式调用 `addTransition`, 最后会构成一个 TransitionsListNode 的链表, 最后又通过 函数 `installTopology` 返回了一个新工厂。这里调用了 StateMachineFactory 的另一个私有构造函数. 

```java
private StateMachineFactory
      (StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT> that,
       boolean optimized) {
    this.defaultInitialState = that.defaultInitialState;
    this.transitionsListNode = that.transitionsListNode;
    this.optimized = optimized;
    if (optimized) {
      makeStateMachineTable();
    } else {
      stateMachineTable = null;
    }
}

public StateMachineFactory
             <OPERAND, STATE, EVENTTYPE, EVENT>
          installTopology() {
    return new StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT>(this, true);
}

```
可以看到，这里通过设置是否优化，会对这个新工厂建立 `stateMachineTable`. 这个表就是状态转移表，是一个hash表。以下是建表的代码：


```Java
private void makeStateMachineTable() {
    Stack<ApplicableTransition<OPERAND, STATE, EVENTTYPE, EVENT>> stack =
      new Stack<ApplicableTransition<OPERAND, STATE, EVENTTYPE, EVENT>>();

    Map<STATE, Map<EVENTTYPE, Transition<OPERAND, STATE, EVENTTYPE, EVENT>>>
      prototype = new HashMap<STATE, Map<EVENTTYPE, Transition<OPERAND, STATE, EVENTTYPE, EVENT>>>();

    prototype.put(defaultInitialState, null);

    // I use EnumMap here because it'll be faster and denser.  I would
    //  expect most of the states to have at least one transition.
    stateMachineTable
       = new EnumMap<STATE, Map<EVENTTYPE,
                           Transition<OPERAND, STATE, EVENTTYPE, EVENT>>>(prototype);

	// 链表是头插法逆序的，使用栈保持建表插入顺序和 addTransition 顺序一致
    for (TransitionsListNode cursor = transitionsListNode;
         cursor != null;
         cursor = cursor.next) {
      stack.push(cursor.transition);
    }

    while (!stack.isEmpty()) {
      stack.pop().apply(this);
    }
}

static private class ApplicableSingleOrMultipleTransition
             <OPERAND, STATE extends Enum<STATE>,
              EVENTTYPE extends Enum<EVENTTYPE>, EVENT>
          implements ApplicableTransition<OPERAND, STATE, EVENTTYPE, EVENT> {
    final STATE preState;
    final EVENTTYPE eventType;
    final Transition<OPERAND, STATE, EVENTTYPE, EVENT> transition;

    ApplicableSingleOrMultipleTransition
        (STATE preState, EVENTTYPE eventType,
         Transition<OPERAND, STATE, EVENTTYPE, EVENT> transition) {
      this.preState = preState;
      this.eventType = eventType;
      this.transition = transition;
    }

    @Override
    public void apply
             (StateMachineFactory<OPERAND, STATE, EVENTTYPE, EVENT> subject) {
      Map<EVENTTYPE, Transition<OPERAND, STATE, EVENTTYPE, EVENT>> transitionMap
        = subject.stateMachineTable.get(preState);
      if (transitionMap == null) {
        // I use HashMap here because I would expect most EVENTTYPE's to not
        //  apply out of a particular state, so FSM sizes would be 
        //  quadratic if I use EnumMap's here as I do at the top level.
        transitionMap = new HashMap<EVENTTYPE,
          Transition<OPERAND, STATE, EVENTTYPE, EVENT>>();
        subject.stateMachineTable.put(preState, transitionMap);
      }
      transitionMap.put(eventType, transition);
    }
  }
```
链表是头插法逆序的，建立Hash表的过程中，借助栈保持元素插入顺序和 addTransition 顺序一致。下图是 Hash 表的内部结构:

<div align="center"><img src="/static/images/DistributedSystem/yarn-fsm-and-ed/YarnStateMachineTable.jpg" style="width:700px;height:500px;">
<caption><center> YarnStateMachineTable </center></caption></div>

这里`stateMachineTable`采用的是 `EnumMap`. 注意是外层的 Map 是 `EnumMap`. 因为 key 是 `preState` 是 State 枚举类型，状态数目固定。适合采用 `EnumMap` 加速。但是 value 是一个真实的 `HashMap`, 因为value `transitionMap` 中的 key 是 EventType 枚举类型，但是状态表并不是状态和事件类型的迪卡尔集。比如，RMAppState 有 11 种， RMAppEventType 有 15 种，并不意味着转移表是 11 * 15， 因为事件类型往往只和特定的某个状态有关系，而不是和所有状态有关系。采用 HashMap 做value 反而节省空间。

Java 中的 `EnumMap` 的 key 是枚举类型，这样看来他的key是固定数目的，内部直接用数组实现了, 因为 key 的数目定了，每个枚举元素对应一个数组索引。 这个有点像 Python 中 `__slots__` 的用法，Python 对象的属性其实是动态的，被存放在一个词典对象里面，如果对象的字段不变的化，其实可以进行 slots 限定，这样 Python 对象就不会创建词典，而是限定该对象只含有 slots 中的属性，不可以再动态添加其他的属性了。这样可以减少对象内存消耗且更加快速。


最后，具体的状态机一般会调用 make 获取一个 StateMachine. 这是 StateMachineFactory 内部的一个 InternalStateMachine. 此后，具体的事件处理器就可以使用该状态机了。


```java
public StateMachine<STATE, EVENTTYPE, EVENT> make(OPERAND operand) {
    return new InternalStateMachine(operand, defaultInitialState);
} 
``` 

比如 RMAppImpl 中的 `handle` 函数，直接调用 `StateMachine.doTransition` 即可。其他具体类型状态机实现也是如此。


```java
public void handle(RMAppEvent event) {

    this.writeLock.lock();

    try {
      ApplicationId appID = event.getApplicationId();
      LOG.debug("Processing event for " + appID + " of type "
          + event.getType());
      final RMAppState oldState = getState();
      try {
        /* keep the master in sync with the state machine */
        this.stateMachine.doTransition(event.getType(), event);
      } catch (InvalidStateTransitonException e) {
        LOG.error("Can't handle this event at current state", e);
        /* TODO fail the application on the failed transition */
      }

      if (oldState != getState()) {
        LOG.info(appID + " State change from " + oldState + " to "
            + getState() + " on event=" + event.getType());
      }
    } finally {
      this.writeLock.unlock();
    }
}
```

InternalStateMachine 调用了 StateMchineFactory 的 doTransition 方法，因为只有工厂才存有 状态机表，原因在于某个类的状态机表是该类所有对象共享的代码，工厂在状态机类中的用法也是一个静态成员，并在状态机类加载的时候初始化，然后所有的状态机对象共享该状态机类的方法和状态机表。对于状态机而言，其实他的图结构在编译的时候就定了，但是每个状态机对象其实是处于图中不同的状态，每个状态机对象都关联一个操作对象，并记录当前状态。

```java
private class InternalStateMachine
        implements StateMachine<STATE, EVENTTYPE, EVENT> {
    private final OPERAND operand;
    private STATE currentState;

    InternalStateMachine(OPERAND operand, STATE initialState) {
      this.operand = operand;
      this.currentState = initialState;
      if (!optimized) {
        maybeMakeStateMachineTable();
      }
    }

    @Override
    public synchronized STATE getCurrentState() {
      return currentState;
    }

    @Override
    public synchronized STATE doTransition(EVENTTYPE eventType, EVENT event)
         throws InvalidStateTransitonException  {
      currentState = StateMachineFactory.this.doTransition
          (operand, currentState, eventType, event);
      return currentState;
    }
  }
  
 ```
 
 `StateMachineFactory` 通过传入的参数，直接查表即可找到对应的 `Transition`. 注意这个 `Transition` 是 `SingleInternalArc` 或者 `MultipleInternalArc`. 他们会调用 `hook.Transition`.
 
 ```Java
 private STATE doTransition
           (OPERAND operand, STATE oldState, EVENTTYPE eventType, EVENT event)
      throws InvalidStateTransitonException {
    // We can assume that stateMachineTable is non-null because we call
    //  maybeMakeStateMachineTable() when we build an InnerStateMachine ,
    //  and this code only gets called from inside a working InnerStateMachine .
    Map<EVENTTYPE, Transition<OPERAND, STATE, EVENTTYPE, EVENT>> transitionMap
      = stateMachineTable.get(oldState);
    if (transitionMap != null) {
      Transition<OPERAND, STATE, EVENTTYPE, EVENT> transition
          = transitionMap.get(eventType);
      if (transition != null) {
        return transition.doTransition(operand, oldState, event, eventType);
      }
    }
    throw new InvalidStateTransitonException(oldState, eventType);
  }
```
对于 `SingleInternalArc` 来说，只要 hook 不是空，直接调用即可。 对于 `MultipleInternalArc` 还需要对得到的 postState 进行验证。这里的 hook 就是用户实现的 `SingleArcTransition` 和 `MultipleArcTransition` 了。

```java
 private class SingleInternalArc
                    implements Transition<OPERAND, STATE, EVENTTYPE, EVENT> {

    private STATE postState;
    private SingleArcTransition<OPERAND, EVENT> hook; // transition hook

    SingleInternalArc(STATE postState,
        SingleArcTransition<OPERAND, EVENT> hook) {
      this.postState = postState;
      this.hook = hook;
    }

    @Override
    public STATE doTransition(OPERAND operand, STATE oldState,
                              EVENT event, EVENTTYPE eventType) {
      if (hook != null) {
        hook.transition(operand, event);
      }
      return postState;
    }
  }

  private class MultipleInternalArc
              implements Transition<OPERAND, STATE, EVENTTYPE, EVENT>{

    // Fields
    private Set<STATE> validPostStates;
    private MultipleArcTransition<OPERAND, EVENT, STATE> hook;  // transition hook

    MultipleInternalArc(Set<STATE> postStates,
                   MultipleArcTransition<OPERAND, EVENT, STATE> hook) {
      this.validPostStates = postStates;
      this.hook = hook;
    }

    @Override
    public STATE doTransition(OPERAND operand, STATE oldState,
                              EVENT event, EVENTTYPE eventType)
        throws InvalidStateTransitonException {
      STATE postState = hook.transition(operand, event);

      if (!validPostStates.contains(postState)) {
        throw new InvalidStateTransitonException(oldState, eventType);
      }
      return postState;
    }
  }
```

operand 就是我们需要操作的具体的状态机，比如 RMApp。下面是一个具体的 hook， `RMAppTransition`，比如 `RMAppNodeUpdateTransition`, 当 `RMAppNodeUpdateEvent` 事件发生，就会触发相应的状态转移。
```Java
private static class RMAppTransition implements
      SingleArcTransition<RMAppImpl, RMAppEvent> {
    public void transition(RMAppImpl app, RMAppEvent event) {
    };

  }

  private static final class RMAppNodeUpdateTransition extends RMAppTransition {
    public void transition(RMAppImpl app, RMAppEvent event) {
      RMAppNodeUpdateEvent nodeUpdateEvent = (RMAppNodeUpdateEvent) event;
      app.processNodeUpdate(nodeUpdateEvent.getUpdateType(),
          nodeUpdateEvent.getNode());
    };
  }
```

Yarn 还提供了可视化状态机的工具类，`VisualizeStateMachine`， 通过反射机制，拿到状态机类的 `stateMachineFactory` 实例，然后调用 `StateMachineFactory.generateStateGraph()`方法构造一个状态机图，让后通过 `Graph.save()` 将图保存为 gz 格式。通过 linux graphviz 工具可以生成图片。

```java
 public static Graph getGraphFromClasses(String graphName, List<String> classes)
      throws Exception {
    Graph ret = null;
    if (classes.size() != 1) {
      ret = new Graph(graphName);
    }
    for (String className : classes) {
      Class clz = Class.forName(className);
      Field factoryField = clz.getDeclaredField("stateMachineFactory");
      factoryField.setAccessible(true);
      StateMachineFactory factory = (StateMachineFactory) factoryField.get(null);
      if (classes.size() == 1) {
        return factory.generateStateGraph(graphName);
      }
      String gname = clz.getSimpleName();
      if (gname.endsWith("Impl")) {
        gname = gname.substring(0, gname.length()-4);
      }
      ret.addSubGraph(factory.generateStateGraph(gname));
    }
    return ret;
  }

  public static void main(String [] args) throws Exception {
    if (args.length < 3) {
      System.err.printf("Usage: %s <GraphName> <class[,class[,...]]> <OutputFile>%n",
          VisualizeStateMachine.class.getName());
      System.exit(1);
    }
    String [] classes = args[1].split(",");
    ArrayList<String> validClasses = new ArrayList<String>();
    for (String c : classes) {
      String vc = c.trim();
      if (vc.length()>0) {
        validClasses.add(vc);
      }
    }
    Graph g = getGraphFromClasses(args[0], validClasses);
    g.save(args[2]);
  }
```

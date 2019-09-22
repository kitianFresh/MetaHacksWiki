---
title: "docker"
date: 2019-08-28 20:35
---
$ sudo cat /var/lib/docker/containers/7b0daafdf04e524422669e29358ec860f9516488532feb0e2e0e26c8bb259738/config.v2.json|jq

```json
{
  "StreamConfig": {},
  "State": {
    "Running": true,
    "Paused": false,
    "Restarting": false,
    "OOMKilled": false,
    "RemovalInProgress": false,
    "Dead": false,
    "Pid": 95697,
    "ExitCode": 0,
    "Error": "",
    "StartedAt": "2019-08-27T12:53:52.084452351Z",
    "FinishedAt": "0001-01-01T00:00:00Z",
    "Health": null
  },
  "ID": "7b0daafdf04e524422669e29358ec860f9516488532feb0e2e0e26c8bb259738",
  "Created": "2019-08-27T12:53:51.720863284Z",
  "Managed": false,
  "Path": "/var/sankuai/hulk/init",
  "Args": [
    "-start_cmd",
    "entry"
  ],
  "Config": {
    "Hostname": "inf-hulk-demo-ebs",
    "Domainname": "",
    "User": "0",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "HULK_LSB_RELEASE=centos6",
      "HULK_APPKEYS=com.sankuai.inf.hulk.demo",
      "HULK_CONTAINER_TYPE=k8s",
      "HULK_ENV=beta",
      "TASK_ID=32583",
      "HULK_DISK=10",
      "HULK_CORE_NUM=1",
      "HULK_INIT_PORT=6710",
      "HULK_LOCAL_ENV=online",
      "HULK_CONFIG_SERVER=http://config.hulk.dev.sankuai.com",
      "HULK_SSHD_PORT=22",
      "ENABLE_FLEXVOLUME=on",
      "HULK_IDC=gh",
      "HULK_K8S_WORKLOAD_TYPE=pod",
      "HULK_DISK_NUM=10",
      "HULK_K8S_WORKLOAD_NAME=inf-hulk-demo-ebs",
      "HULK_THRIDSERVICE_ENV=online",
      "HULK_MEM_NUM=1024",
      "HULK_APPKEY=com.sankuai.inf.hulk.demo",
      "HULK_DOCKER_COMMAND=",
      "HULK_TASK_ID=26124",
      "HULK_MEM=1024",
      "HULK_CONTAINER_NAME=app",
      "KUBERNETES_PORT_443_TCP_PROTO=tcp",
      "KUBERNETES_PORT_443_TCP_PORT=443",
      "KUBERNETES_PORT_443_TCP_ADDR=172.17.0.1",
      "KUBERNETES_SERVICE_HOST=172.17.0.1",
      "KUBERNETES_SERVICE_PORT=443",
      "KUBERNETES_SERVICE_PORT_HTTPS=443",
      "KUBERNETES_PORT=tcp://172.17.0.1:443",
      "KUBERNETES_PORT_443_TCP=tcp://172.17.0.1:443",
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "container=docker",
      "LC_ALL=en_US.UTF-8",
      "LANG=en_US.UTF-8",
      "TERM=xterm-256color",
      "IMAGE=offline_java_190801_centos6"
    ],
    "Cmd": [
      "-start_cmd",
      "entry"
    ],
    "Image": "docker.io/sankuai/centos@sha256:27bca559e2557c5dcc1d6125ffd12e97f682f2d4a4b100ccf61542a3f1d745ed",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": [
      "/var/sankuai/hulk/init"
    ],
    "OnBuild": null,
    "Labels": {
      "annotation.io.kubernetes.container.hash": "8ab533d4",
      "annotation.io.kubernetes.container.restartCount": "0",
      "annotation.io.kubernetes.container.terminationMessagePath": "/dev/termination-log",
      "annotation.io.kubernetes.container.terminationMessagePolicy": "File",
      "annotation.io.kubernetes.pod.terminationGracePeriod": "30",
      "build-date": "2019-08-01 12:14:00",
      "io.kubernetes.container.logpath": "/var/log/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/app_0.log",
      "io.kubernetes.container.name": "app",
      "io.kubernetes.docker.type": "container",
      "io.kubernetes.pod.name": "inf-hulk-demo-ebs",
      "io.kubernetes.pod.namespace": "default",
      "io.kubernetes.pod.uid": "b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab",
      "io.kubernetes.sandbox.id": "9bea248d2e7805648028af50b738a220d946399c19c9db64a55786dfb7527833",
      "license": "GPLv2",
      "name": "Meituan CentOS6/Centos7 java Image",
      "vendor": "CentOS"
    }
  },
  "Image": "sha256:e2c44c2e1b1d9b573e37ca2a35704bffa49dfe968c1cd9fd4cab2f57a3f49c08",
  "NetworkSettings": {
    "Bridge": "",
    "SandboxID": "",
    "HairpinMode": false,
    "LinkLocalIPv6Address": "",
    "LinkLocalIPv6PrefixLen": 0,
    "Networks": null,
    "Service": null,
    "Ports": null,
    "SandboxKey": "",
    "SecondaryIPAddresses": null,
    "SecondaryIPv6Addresses": null,
    "IsAnonymousEndpoint": false,
    "HasSwarmEndpoint": false
  },
  "LogPath": "/var/lib/docker/containers/7b0daafdf04e524422669e29358ec860f9516488532feb0e2e0e26c8bb259738/7b0daafdf04e524422669e29358ec860f9516488532feb0e2e0e26c8bb259738-json.log",
  "Name": "/k8s_app_inf-hulk-demo-ebs_default_b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab_0",
  "Driver": "devicemapper",
  "MountLabel": "",
  "ProcessLabel": "",
  "RestartCount": 0,
  "HasBeenStartedBefore": true,
  "HasBeenManuallyStopped": false,
  "MountPoints": {
    "/dev/termination-log": {
      "Source": "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/containers/app/bb73d8d2",
      "Destination": "/dev/termination-log",
      "RW": true,
      "Name": "",
      "Driver": "",
      "Type": "bind",
      "Propagation": "rprivate",
      "Spec": {
        "Type": "bind",
        "Source": "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/containers/app/bb73d8d2",
        "Target": "/dev/termination-log"
      }
    },
    "/docker": {
      "Source": "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/volumes/hulk~ebs/hulkebs",
      "Destination": "/docker",
      "RW": true,
      "Name": "",
      "Driver": "",
      "Type": "bind",
      "Propagation": "rprivate",
      "Spec": {
        "Type": "bind",
        "Source": "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/volumes/hulk~ebs/hulkebs",
        "Target": "/docker"
      }
    },
    "/etc/hosts": {
      "Source": "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/etc-hosts",
      "Destination": "/etc/hosts",
      "RW": true,
      "Name": "",
      "Driver": "",
      "Type": "bind",
      "Propagation": "rprivate",
      "Spec": {
        "Type": "bind",
        "Source": "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/etc-hosts",
        "Target": "/etc/hosts"
      }
    }
  },
  "SecretReferences": null,
  "AppArmorProfile": "",
  "HostnamePath": "/var/lib/docker/containers/9bea248d2e7805648028af50b738a220d946399c19c9db64a55786dfb7527833/hostname",
  "HostsPath": "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/etc-hosts",
  "ShmPath": "/var/lib/docker/containers/9bea248d2e7805648028af50b738a220d946399c19c9db64a55786dfb7527833/shm",
  "ResolvConfPath": "/var/lib/docker/containers/9bea248d2e7805648028af50b738a220d946399c19c9db64a55786dfb7527833/resolv.conf",
  "SeccompProfile": "unconfined",
  "NoNewPrivileges": false
}
```

$ sudo cat /var/lib/docker/containers/7b0daafdf04e524422669e29358ec860f9516488532feb0e2e0e26c8bb259738/hostconfig.json|jq
```json
{
  "Binds": [
    "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/volumes/hulk~ebs/hulkebs:/docker",
    "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/etc-hosts:/etc/hosts",
    "/var/lib/kubelet/pods/b0f9d4ca-c8c9-11e9-aa09-0022dfb3e5ab/containers/app/bb73d8d2:/dev/termination-log"
  ],
  "ContainerIDFile": "",
  "LogConfig": {
    "Type": "json-file",
    "Config": {
      "max-size": "100m"
    }
  },
  "NetworkMode": "container:9bea248d2e7805648028af50b738a220d946399c19c9db64a55786dfb7527833",
  "PortBindings": null,
  "RestartPolicy": {
    "Name": "",
    "MaximumRetryCount": 0
  },
  "AutoRemove": false,
  "VolumeDriver": "",
  "VolumesFrom": null,
  "CapAdd": [
    "SYS_ADMIN",
    "SYS_PTRACE",
    "NET_ADMIN",
    "NET_RAW"
  ],
  "CapDrop": null,
  "Dns": [],
  "DnsOptions": [],
  "DnsSearch": [],
  "ExtraHosts": null,
  "GroupAdd": null,
  "IpcMode": "container:9bea248d2e7805648028af50b738a220d946399c19c9db64a55786dfb7527833",
  "Cgroup": "",
  "Links": [],
  "OomScoreAdj": 993,
  "PidMode": "",
  "Privileged": false,
  "PublishAllPorts": false,
  "ReadonlyRootfs": false,
  "SecurityOpt": [
    "seccomp=unconfined"
  ],
  "UTSMode": "",
  "UsernsMode": "",
  "ShmSize": 67108864,
  "Runtime": "docker-runc",
  "ConsoleSize": [
    0,
    0
  ],
  "Isolation": "",
  "CpuShares": 102,
  "Memory": 1073741824,
  "NanoCpus": 0,
  "CgroupParent": "kubepods-burstable-podb0f9d4ca_c8c9_11e9_aa09_0022dfb3e5ab.slice",
  "BlkioWeight": 0,
  "BlkioWeightDevice": null,
  "BlkioDeviceReadBps": [
    {
      "Path": "/dev/sde",
      "Rate": 52428800
    }
  ],
  "BlkioDeviceWriteBps": [
    {
      "Path": "/dev/sde",
      "Rate": 52428800
    }
  ],
  "BlkioDeviceReadIOps": [
    {
      "Path": "/dev/sde",
      "Rate": 500
    }
  ],
  "BlkioDeviceWriteIOps": [
    {
      "Path": "/dev/sde",
      "Rate": 500
    }
  ],
  "CpuPeriod": 100000,
  "CpuQuota": 100000,
  "CpuRealtimePeriod": 0,
  "CpuRealtimeRuntime": 0,
  "CpusetCpus": "8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39",
  "CpusetMems": "",
  "Devices": [],
  "DiskQuota": 0,
  "KernelMemory": 0,
  "MemoryReservation": 0,
  "MemorySwap": 3221225472,
  "MemorySwappiness": -1,
    {
      "Name": "nofile",
      "Hard": 1048576,
      "Soft": 1048576
    },
    {
      "Name": "memlock",
      "Hard": -1,
      "Soft": -1
    }
  ],
  "CpuCount": 0,
  "CpuPercent": 0,
  "IOMaximumIOps": 0,
  "IOMaximumBandwidth": 0
}
```


 sudo docker run --name demo --memory 1073741824 --cpuset-cpus "8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39" --cpu-period 100000 --cpu-quota 100000 -d nginx
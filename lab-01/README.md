# Lab 01 - Nodes

A node is a worker machine in Kubernetes, previously known as a minion. A node 
may be a VM or physical machine, depending on the cluster. Each node contains 
the services necessary to run pods and is managed by the master components. The 
services on a node include the container runtime, kubelet and kube-proxy. 

## Task 1: Listing nodes

To see which nodes are part of your Kubernetes cluster run the 
`kubectl get nodes` command:

```
kubectl get nodes

NAME                                        STATUS    ROLES     AGE       VERSION
gke-kbc-steven-default-pool-b82ee1c9-5n9j   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-6wnx   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-x13z   Ready     <none>    20m       v1.11.7-gke.4
```

We can use the `kubectl get nodes -o wide` command to get some additional 
information about the nodes:

```
kubectl get nodes -o wide
NAME                                        STATUS    ROLES     AGE       VERSION         EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-kbc-steven-default-pool-b82ee1c9-5n9j   Ready     <none>    21m       v1.11.7-gke.4   35.188.203.43   Container-Optimized OS from Google   4.14.89+         docker://17.3.2
gke-kbc-steven-default-pool-b82ee1c9-6wnx   Ready     <none>    21m       v1.11.7-gke.4   35.192.143.33   Container-Optimized OS from Google   4.14.89+         docker://17.3.2
gke-kbc-steven-default-pool-b82ee1c9-x13z   Ready     <none>    21m       v1.11.7-gke.4   23.251.156.4    Container-Optimized OS from Google   4.14.89+         docker://17.3.2
```

## Task 2: Getting detailed node information

If we want more detailed information about a node we have to use the 
`kubectl describe node <node_name>` command:

```
kubectl describe node gke-kbc-steven-default-pool-b82ee1c9-5n9j

Name:               gke-kbc-steven-default-pool-b82ee1c9-5n9j
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/fluentd-ds-ready=true
                    beta.kubernetes.io/instance-type=n1-standard-1
                    beta.kubernetes.io/os=linux
                    cloud.google.com/gke-nodepool=default-pool
                    cloud.google.com/gke-os-distribution=cos
                    failure-domain.beta.kubernetes.io/region=us-central1
                    failure-domain.beta.kubernetes.io/zone=us-central1-a
                    kubernetes.io/hostname=gke-kbc-steven-default-pool-b82ee1c9-5n9j
Annotations:        container.googleapis.com/instance_id=2496827992163718317
                    node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
CreationTimestamp:  Mon, 11 Mar 2019 10:30:17 +0100
Taints:             <none>
Unschedulable:      false
Conditions:
  Type                          Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                          ------  -----------------                 ------------------                ------                       -------
  FrequentKubeletRestart        False   Mon, 11 Mar 2019 11:01:14 +0100   Mon, 11 Mar 2019 10:33:53 +0100   FrequentKubeletRestart       kubelet is functioning properly
  FrequentDockerRestart         False   Mon, 11 Mar 2019 11:01:14 +0100   Mon, 11 Mar 2019 10:33:54 +0100   FrequentDockerRestart        docker is functioning properly
  FrequentContainerdRestart     False   Mon, 11 Mar 2019 11:01:14 +0100   Mon, 11 Mar 2019 10:33:55 +0100   FrequentContainerdRestart    containerd is functioning properly
  CorruptDockerOverlay2         False   Mon, 11 Mar 2019 11:01:14 +0100   Mon, 11 Mar 2019 10:33:53 +0100   CorruptDockerOverlay2        docker overlay2 is functioning properly
  KernelDeadlock                False   Mon, 11 Mar 2019 11:01:14 +0100   Mon, 11 Mar 2019 10:28:52 +0100   KernelHasNoDeadlock          kernel has no deadlock
  ReadonlyFilesystem            False   Mon, 11 Mar 2019 11:01:14 +0100   Mon, 11 Mar 2019 10:28:52 +0100   FilesystemIsNotReadOnly      Filesystem is not read-only
  FrequentUnregisterNetDevice   False   Mon, 11 Mar 2019 11:01:14 +0100   Mon, 11 Mar 2019 10:33:53 +0100   UnregisterNetDevice          node is functioning properly
  NetworkUnavailable            False   Mon, 11 Mar 2019 10:30:29 +0100   Mon, 11 Mar 2019 10:30:29 +0100   RouteCreated                 RouteController created a route
  OutOfDisk                     False   Mon, 11 Mar 2019 11:01:32 +0100   Mon, 11 Mar 2019 10:30:17 +0100   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure                False   Mon, 11 Mar 2019 11:01:32 +0100   Mon, 11 Mar 2019 10:30:17 +0100   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure                  False   Mon, 11 Mar 2019 11:01:32 +0100   Mon, 11 Mar 2019 10:30:17 +0100   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure                   False   Mon, 11 Mar 2019 11:01:32 +0100   Mon, 11 Mar 2019 10:30:17 +0100   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                         True    Mon, 11 Mar 2019 11:01:32 +0100   Mon, 11 Mar 2019 10:30:37 +0100   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.128.0.37
  ExternalIP:  35.188.203.43
  Hostname:    gke-kbc-steven-default-pool-b82ee1c9-5n9j
Capacity:
 cpu:                1
 ephemeral-storage:  98868448Ki
 hugepages-2Mi:      0
 memory:             3787656Ki
 pods:               110
Allocatable:
 cpu:                940m
 ephemeral-storage:  47093746742
 hugepages-2Mi:      0
 memory:             2702216Ki
 pods:               110
System Info:
 Machine ID:                 6ea81fc2b497faf269c516c42061b928
 System UUID:                6EA81FC2-B497-FAF2-69C5-16C42061B928
 Boot ID:                    fcf8db1e-f20a-47af-933d-7886cbfa3fa3
 Kernel Version:             4.14.89+
 OS Image:                   Container-Optimized OS from Google
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://17.3.2
 Kubelet Version:            v1.11.7-gke.4
 Kube-Proxy Version:         v1.11.7-gke.4
PodCIDR:                     10.24.1.0/24
ExternalID:                  gke-kbc-steven-default-pool-b82ee1c9-5n9j
ProviderID:                  gce://certain-nexus-865/us-central1-a/gke-kbc-steven-default-pool-b82ee1c9-5n9j
Non-terminated Pods:         (4 in total)
  Namespace                  Name                                                    CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                                                    ------------  ----------  ---------------  -------------
  kube-system                fluentd-gcp-v3.2.0-st25d                                100m (10%)    1 (106%)    200Mi (7%)       500Mi (18%)
  kube-system                heapster-v1.6.0-beta.1-575d556fcd-dxfz6                 138m (14%)    138m (14%)  301856Ki (11%)   301856Ki (11%)
  kube-system                kube-proxy-gke-kbc-steven-default-pool-b82ee1c9-5n9j    100m (10%)    0 (0%)      0 (0%)           0 (0%)
  kube-system                metrics-server-v0.2.1-fd596d746-5qpdw                   53m (5%)      148m (15%)  154Mi (5%)       404Mi (15%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits    Memory Requests  Memory Limits
  ------------  ----------    ---------------  -------------
  391m (41%)    1286m (136%)  664352Ki (24%)   1227552Ki (45%)
Events:
  Type    Reason                     Age                From                                                        Message
  ----    ------                     ----               ----                                                        -------
  Normal  Starting                   31m                kubelet, gke-kbc-steven-default-pool-b82ee1c9-5n9j          Starting kubelet.
  Normal  NodeHasSufficientDisk      31m (x2 over 31m)  kubelet, gke-kbc-steven-default-pool-b82ee1c9-5n9j          Node gke-kbc-steven-default-pool-b82ee1c9-5n9j status is now: NodeHasSufficientDisk
  Normal  NodeHasSufficientMemory    31m (x2 over 31m)  kubelet, gke-kbc-steven-default-pool-b82ee1c9-5n9j          Node gke-kbc-steven-default-pool-b82ee1c9-5n9j status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure      31m (x2 over 31m)  kubelet, gke-kbc-steven-default-pool-b82ee1c9-5n9j          Node gke-kbc-steven-default-pool-b82ee1c9-5n9j status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID       31m (x2 over 31m)  kubelet, gke-kbc-steven-default-pool-b82ee1c9-5n9j          Node gke-kbc-steven-default-pool-b82ee1c9-5n9j status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced    31m                kubelet, gke-kbc-steven-default-pool-b82ee1c9-5n9j          Updated Node Allocatable limit across pods
  Normal  NodeReady                  30m                kubelet, gke-kbc-steven-default-pool-b82ee1c9-5n9j          Node gke-kbc-steven-default-pool-b82ee1c9-5n9j status is now: NodeReady
  Normal  Starting                   30m                kube-proxy, gke-kbc-steven-default-pool-b82ee1c9-5n9j       Starting kube-proxy.
  Normal  FrequentKubeletRestart     27m                systemd-monitor, gke-kbc-steven-default-pool-b82ee1c9-5n9j  Node condition FrequentKubeletRestart is now: False, reason: FrequentKubeletRestart
  Normal  CorruptDockerOverlay2      27m                docker-monitor, gke-kbc-steven-default-pool-b82ee1c9-5n9j   Node condition CorruptDockerOverlay2 is now: False, reason: CorruptDockerOverlay2
  Normal  UnregisterNetDevice        27m                kernel-monitor, gke-kbc-steven-default-pool-b82ee1c9-5n9j   Node condition FrequentUnregisterNetDevice is now: False, reason: UnregisterNetDevice
  Normal  FrequentDockerRestart      27m                systemd-monitor, gke-kbc-steven-default-pool-b82ee1c9-5n9j  Node condition FrequentDockerRestart is now: False, reason: FrequentDockerRestart
  Normal  FrequentContainerdRestart  27m                systemd-monitor, gke-kbc-steven-default-pool-b82ee1c9-5n9j  Node condition FrequentContainerdRestart is now: False, reason: FrequentContainerdRestart
```

Read through the output above to see all the information that is available about 
the node.

The `kubectl describe node <node_name>` is a very useful tool when 
troubleshooting a node that is failing.  Going through the output will give a 
good indication why the node is not behaving as it should.
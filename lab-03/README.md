# Lab 03 - Namespaces

Kubernetes supports multiple virtual clusters backed by the same physical
cluster. These virtual clusters are called namespaces.

Namespaces make it possible to run different environments on a single Kubernetes
cluster, such as DEV, TST and UAT.  Namespace can also be used to set different
access controls per namespace.

Most objects in Kubernetes can be namespaced (pods, services, pvc,...), keep in
mind however that some objects however cannot be namespaced and are cluster-wide
(pv for example).

## Task 1: Listing namespaces

To see which namespaces are available use the `kubectl get namespaces` command:

```
kubectl get namespaces

NAME          STATUS    AGE
default       Active    40m
kube-public   Active    40m
kube-system   Active    40m
```

## Task 2: Creating a new namespace

Creating a namespace is easy, `kubectl create namespace test`:

```
kubectl create namespace test-${USERNAME}

namespace "test-trescst" created
```

Check that your namespace has been created:

```
kubectl get ns

NAME           STATUS    AGE
default        Active    45m
kube-public    Active    45m
kube-system    Active    45m
test           Active    1m
```

> NOTE: as you can see we abreviated `namespace` to `ns`, most of the objects in
> Kubernetes have abreviations that you can use on the command line

## Task 3: Specifying namespaces

When working with namespaces it is important to know that if you do not specify
a specific namepace when issuing a `kubectl` command the default namespace of
your current context is assumed.  In our case this will be `default` but you can
of course create new and/or customize your context.

This means that running the following command:

```
kubectl get all

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.27.240.1   <none>        443/TCP   53m
```

Is exactly the same as running:

```
kubectl get all -n default

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.27.240.1   <none>        443/TCP   53m
```

But if we do this for a different namespace we of course get a completely
different result:

```
kubectl get all -n kube-system
NAME                                                       READY     STATUS    RESTARTS   AGE
pod/event-exporter-v0.2.3-85644fcdf-brk52                  2/2       Running   0          55m
pod/fluentd-gcp-scaler-8b674f786-6wt6v                     1/1       Running   0          55m
pod/fluentd-gcp-v3.2.0-gb9qz                               2/2       Running   0          54m
pod/fluentd-gcp-v3.2.0-hmbnp                               2/2       Running   0          54m
pod/fluentd-gcp-v3.2.0-st25d                               2/2       Running   0          54m
pod/heapster-v1.6.0-beta.1-575d556fcd-dxfz6                3/3       Running   0          54m
pod/kube-dns-7df4cb66cb-5tn89                              4/4       Running   0          55m
pod/kube-dns-7df4cb66cb-c52g6                              4/4       Running   0          55m
pod/kube-dns-autoscaler-67c97c87fb-xp9xt                   1/1       Running   0          55m
pod/kube-proxy-gke-kbc-steven-default-pool-b82ee1c9-5n9j   1/1       Running   0          55m
pod/kube-proxy-gke-kbc-steven-default-pool-b82ee1c9-6wnx   1/1       Running   0          55m
pod/kube-proxy-gke-kbc-steven-default-pool-b82ee1c9-x13z   1/1       Running   0          55m
pod/l7-default-backend-7ff48cffd7-w4nrz                    1/1       Running   0          55m
pod/metrics-server-v0.2.1-fd596d746-5qpdw                  2/2       Running   0          54m
NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
service/default-http-backend   NodePort    10.27.243.2    <none>        80:31515/TCP    55m
service/heapster               ClusterIP   10.27.246.39   <none>        80/TCP          55m
service/kube-dns               ClusterIP   10.27.240.10   <none>        53/UDP,53/TCP   55m
service/metrics-server         ClusterIP   10.27.255.78   <none>        443/TCP         55m
NAME                                      DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                                  AGE
daemonset.apps/fluentd-gcp-v3.2.0         3         3         3         3            3           beta.kubernetes.io/fluentd-ds-ready=true       55m
daemonset.apps/metadata-proxy-v0.1        0         0         0         0            0           beta.kubernetes.io/metadata-proxy-ready=true   55m
daemonset.apps/nvidia-gpu-device-plugin   0         0         0         0            0           <none>                                         55m
NAME                                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/event-exporter-v0.2.3    1         1         1            1           55m
deployment.apps/fluentd-gcp-scaler       1         1         1            1           55m
deployment.apps/heapster-v1.6.0-beta.1   1         1         1            1           55m
deployment.apps/kube-dns                 2         2         2            2           55m
deployment.apps/kube-dns-autoscaler      1         1         1            1           55m
deployment.apps/l7-default-backend       1         1         1            1           55m
deployment.apps/metrics-server-v0.2.1    1         1         1            1           55m
NAME                                                DESIRED   CURRENT   READY     AGE
replicaset.apps/event-exporter-v0.2.3-85644fcdf     1         1         1         55m
replicaset.apps/fluentd-gcp-scaler-8b674f786        1         1         1         55m
replicaset.apps/heapster-v1.6.0-beta.1-575d556fcd   1         1         1         54m
replicaset.apps/heapster-v1.6.0-beta.1-7d4677f9c6   0         0         0         55m
replicaset.apps/kube-dns-7df4cb66cb                 2         2         2         55m
replicaset.apps/kube-dns-autoscaler-67c97c87fb      1         1         1         55m
replicaset.apps/l7-default-backend-7ff48cffd7       1         1         1         55m
replicaset.apps/metrics-server-v0.2.1-597c89dc98    0         0         0         55m
replicaset.apps/metrics-server-v0.2.1-fd596d746     1         1         1         54m
```

## Task 4: All namespaces

It could happen that you do not really know in which namespace a Kubernetes
object is running.  If that is the case you can always add the
`--all-namespaces` option to your command to list objects from all of the
namespaces.

```
kubectl get pods --all-namespaces

NAMESPACE     NAME                                                   READY     STATUS    RESTARTS   AGE
default       nginx                                                  1/1       Running   0          43m
kube-system   event-exporter-v0.2.3-85644fcdf-brk52                  2/2       Running   0          2h
kube-system   fluentd-gcp-scaler-8b674f786-6wt6v                     1/1       Running   0          2h
kube-system   fluentd-gcp-v3.2.0-gb9qz                               2/2       Running   0          2h
kube-system   fluentd-gcp-v3.2.0-hmbnp                               2/2       Running   0          2h
kube-system   fluentd-gcp-v3.2.0-st25d                               2/2       Running   0          2h
kube-system   heapster-v1.6.0-beta.1-575d556fcd-dxfz6                3/3       Running   0          2h
kube-system   kube-dns-7df4cb66cb-5tn89                              4/4       Running   0          2h
kube-system   kube-dns-7df4cb66cb-c52g6                              4/4       Running   0          2h
kube-system   kube-dns-autoscaler-67c97c87fb-xp9xt                   1/1       Running   0          2h
kube-system   kube-proxy-gke-kbc-steven-default-pool-b82ee1c9-5n9j   1/1       Running   0          2h
kube-system   kube-proxy-gke-kbc-steven-default-pool-b82ee1c9-6wnx   1/1       Running   0          2h
kube-system   kube-proxy-gke-kbc-steven-default-pool-b82ee1c9-x13z   1/1       Running   0          2h
kube-system   l7-default-backend-7ff48cffd7-w4nrz                    1/1       Running   0          2h
kube-system   metrics-server-v0.2.1-fd596d746-5qpdw                  2/2       Running   0          2h
test          hello-world                                            1/1       Running   0          1h
```

## Task 5: Deleting namespaces

Deleting a namespace is very easy, keep in mind however that when you delete a
namespace *all* the objects in that namespace will be deleten. So always verify
that all the objects in that namespace can be deleted:

```
kubectl delete ns test

namespace "test" deleted
```
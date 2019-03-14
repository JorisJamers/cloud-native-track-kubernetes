# Lab 10 - Deployments (more advanced topics)

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-10

namespace "lab-10" created
```

## Task 1:

## Task 2: Blue / green deployment

For this task we are creating 2 versions of 1 deployment. This means that we
can use the service to piont to either of the deployments. This can be used for
example if you want to update your application. We are going to use the
container-info application again to demo this feature.

First we will create a blue deployment. This is the first and initial version
of our application.

Create a file `lab-10-deployment-blue.yml` with the following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info-blue
  labels:
    app: container-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
        version: "blue"
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
```

As you can see we have added a new label called `version` with this label we can
select the correct deployment in the service we are creating in this task.

```
kubectl apply -f lab-10-deployment-blue.yml -n lab-10
```

Now create the file `lab-10-service-blue-green.yml` with the following content.

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
    version: "blue"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Notice that in the `spec:selector:version` we chose for `blue` this specifies
the version of the deployment we are going to expose.

```
kubectl apply -f lab-10-service-blue-green.yml -n lab-10
```

Confirm that the application is running by doing to following command.

```
minikube service container-info -n lab-10
```

You will see that our `blue` version of the deployment is now running on our
minikube instance.

After you exposed the `blue` version we are going to create a `green` version
of the application. This is done by creating a file
`lab-10-deployment-green.yml` with the following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info-green
  labels:
    app: container-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
        version: "green"
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:green
        ports:
        - containerPort: 80
```

We changed some parameters from `blue` to `green`, notice that the image changed
as well. We are now deploying a `green` version.

```
kubectl -f lab-10-deployment-green.yml -n lab-10
```

If you list the deployments in your namespace you will see that we now have 2
deployments.

```
kubectl get deployment -n lab-10

NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
container-info-blue    3         3         3            3           12m
container-info-green   3         3         3            3           12m
```

A `blue` and a `green` version. Remember that our service is still pointing at
the `blue` version. In the next step we will update the service in a way that it
will point to the `green` version of our deployment.

Edit the `lab-10-service-blue-green.yml` file. You can also just clear the file
and copy the following content.

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
    version: "green"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

The only thing that is updated in this file is basically the `version` this is
set to `green` now.

```
kubectl apply -f lab-10-service-blue-green.yml -n lab-10
```

Now you can run the service command again to check your newly updated version of
the deployment.

```
minikube service container-info -n lab-10
```

When you did these steps succesfully you have done a `blue / green` on `minikube`

Delete the namespace so we can start off with a clean namespace for the
following tasks.

```
kubectl delete namespace lab-10
```

## Task 3: Liveness / readiness

To test the liveness and the readiness of a pod we are going to create the following
file. Name the file `probes.yml` and fill with the following content.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: probes
  name: probes
spec:
  containers:
  - name: probes
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 6000
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/ready
      initialDelaySeconds: 5
      periodSeconds: 5
```

Apply the file in your namespace to create the `probes` pod.

```
kubectl apply -f probes.yml -n lab-10

pod "probes" created
```

Now we need to check the state of the pod.

```
kubectl get pods -n lab-10

NAME      READY     STATUS    RESTARTS   AGE
probes    0/1       Running   0          45s
```

We see that the pod is running but not ready yet.

Before we are going to fix this readiness check we are going to create a service.
When a pod is not ready you will not get an endpoint in that service.

Create the file `probes-service.yml` with this content.

```
kind: Service
apiVersion: v1
metadata:
  name: probes-service
spec:
  selector:
    app: probes
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Apply the file and check out the service.

```
kubectl apply -f probes-service.yml -n lab-10

service "probes-service" created
```

If we describe the service at this moment we won't see an endpoint.

```
kubectl describe service probes-service -n lab-10

Name:                     probes-service
Namespace:                lab-10
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"probes-service","namespace":"lab-10"},"spec":{"ports":[{"port":80,"protocol":"...
Selector:                 app=probes
Type:                     NodePort
IP:                       10.105.221.153
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32748/TCP
Endpoints:
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

If we fix the readiness check on the pod, we will get the endpoint. We are going to
fix it with creating the `/tmp/ready` file in the pod.

```
kubectl exec probes touch /tmp/ready -n lab-10
```

Now you will see that the pod is ready.

```
kubectl get pods -n lab-10

NAME      READY     STATUS    RESTARTS   AGE
probes    1/1       Running   0          6m
```

And if we describe the service once again we will see that an endpoint has appeared.

```
kubectl describe service probes-service -n lab-10

Name:                     probes-service
Namespace:                lab-10
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"probes-service","namespace":"lab-10"},"spec":{"ports":[{"port":80,"protocol":"...
Selector:                 app=probes
Type:                     NodePort
IP:                       10.105.221.153
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32748/TCP
Endpoints:                172.17.0.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

At this moment if we remove the `/tmp/healthy` the pod will become unhealhty.

```
kubernetes exec probes rm /tmp/healthy -n lab-10
```

If we describe the pod we will see in the recent events that the pod became unhealhty.

```
kubernetes describe pod probes -n lab-10


Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  21m                default-scheduler  Successfully assigned lab-10/probes to minikube
  Normal   Pulling    21m                kubelet, minikube  pulling image "busybox"
  Normal   Pulled     21m                kubelet, minikube  Successfully pulled image "busybox"
  Normal   Created    21m                kubelet, minikube  Created container
  Normal   Started    21m                kubelet, minikube  Started container
  Warning  Unhealthy  1m (x2 over 2m)    kubelet, minikube  Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

## Task 4: Node labeling

Create a fresh namespace for this task.

```
kubectl create namespace lab-10
```

The basic node labeling will give you the option to restrict pods to specific nodes.
This could be very usefull if you are running multiple environments. Imagine that
we are pushing our application from `test` to `uat` to `prod`. With node labeling
you can specify for example that all the pods of the `test` environment are going
to be scheduled on the `test` node.

Because we are using `minikube` in this lab, we only have 1 node. This example will
show you how you can label the `minikube` node and make sure that the `application` is running on this `minikube` node.

First we need to find the name of the node.

```
kubectl get nodes

NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    2d        v1.13.3
```

Now you can add a label to the `minikube` node with the following command.

```
kubectl label nodes minikube environment=test
```

This labels the `minikube` node with the `environment=test` label. You can confirm
that the node is labeled with the command.

```
kubectl get nodes --show-labels

NAME       STATUS    ROLES     AGE       VERSION   LABELS
minikube   Ready     master    2d        v1.13.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,environment=test,kubernetes.io/hostname=minikube,node-role.kubernetes.io/master=
```

Create a new deployment file `label-deployment.yml` and add this content to it.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info-blue
  labels:
    app: container-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
        version: "blue"
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
      nodeSelector:
        environment: test
```

This is basically the `blue.yml` deployment. We added the

```
nodeSelector:
  environment: test
```

And this will define that the deployment is going to run on this specific node.
Apply the deployment.

```
kubectl apply -f label-deployment.yml -n lab-10
```
### Task 4.1: Pod Affinity

Pod Affinity means that you can schedule pods with the same label on the same
pod. If available. So imagine that we want our `test` pod to run next to other
pods with the label `test` on a node. We need to define the `podAffinity` like
this.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: environment
            operator: In
            values:
            - test
  containers:
  - name: container-info
    image: gluobe/container-info:blue
```

We are using `requiredDuringSchedulingIgnoredDuringExecution` this means that
this is a requirement while the pod is getting scheduled. It's also possible to
define `preferredDuringSchedulingIgnoredDuringExecution`. This means that he
will try to schedule the pod with these conditions if possible. Otherwise it
will schedule the pod on a other node.

### Task 4.2: Pod Anti Affinity

The oposite from what we did above can be done with `antiAffinity`. If we do the
following in the pod yml.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: environment
            operator: In
            values:
            - test
  containers:
  - name: container-info
    image: gluobe/container-info:blue
```

The pod will `not` schedule next to the already running `test` environment pod
on any node. This means that the scheduler is going to look for another pod
where no pod is running with the label `environment=test`.

### Task 4.3: Taints

It's also possible to taint a node. This means that the node will explicitly
reject nodes with a defined label.

```
kubectl taint nodes minikube environment=test:NoSchedule
```

If you use this commando the `minikube` node will reject all pods with the label
`environment=test`.

TODO
* list
* green/blue
* liveness/readiness
* node labeling (affinity, anti-affinity, taints,...)

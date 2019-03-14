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

Create a file `blue.yml` with the following content.

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
kubectl apply -f blue.yml -n lab-10
```

Now create the file `service.yml` with the following content.

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
kubectl apply -f service.yml -n lab-10
```

Confirm that the application is running by doing to following command.

```
minikube service container-info -n lab-10
```

You will see that our `blue` version of the deployment is now running on our
minikube instance.

After you exposed the `blue` version we are going to create a `green` version
of the application. This is done by creating a file `green.yml` with the following
content.

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
kubectl -f green.yml -n lab-10
```

If you list the deployments in your namespace you will see that we now have 2
deployments.

```
kubectl get deployment -n lab-10

NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
container-info-blue    3         3         3            3           12m
container-info-green   3         3         3            3           12m
```

A `blue` and a `green` version. Remember that our service is still pointing at the
`blue` version. In the next step we will update the service in a way that it will
point to the `green` version of our deployment.

Edit the `service.yml` file. You can also just clear the file and copy the following
content.

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

The only thing that is updated in this file is basically the `version` this is set
to `green` now.

```
kubectl apply -f service.yml -n lab-10
```

Now you can run the service command again to check your newly updated version of
the deployment.

```
minikube service container-info -n lab-10
```

When you did these steps succesfully you have done a `blue / green` on `minikube`

Delete the namespace so we can start off with a clean namespace for the following
tasks.

```
kubectl delete namespace lab-10
```


## Task 3: Liveness / readiness



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

TODO
* list
* green/blue
* liveness/readiness
* node labeling (affinity, anti-affinity, taints,...)

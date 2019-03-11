# Lab 06 - Services

A Kubernetes Service is an abstraction which defines a logical set of Pods and a 
policy by which to access them - sometimes called a micro-service. The set of 
Pods targeted by a Service is (usually) determined by a Label Selector.

As an example, consider an image-processing backend which is running with 3 
replicas. Those replicas are fungible - frontends do not care which backend they 
use. While the actual Pods that compose the backend set may change, the frontend 
clients should not need to be aware of that or keep track of the list of 
backends themselves. The Service abstraction enables this decoupling.

To make copy/pasting easier we will again export our username first:

```
export USERNAME=<username>
```

## Task 1: Creating your first service

Before we can create our service we will first have to create our namespace and 
our deployment (with 3 replica's):

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Namespace
metadata:
  name: lab-06-${USERNAME}
EOF
```

```
cat <<EOF | kubectl create -n lab-06-${USERNAME} -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info
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
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
EOF
```

Verfiy that this is working:

```
kubectl get pods -n lab-06-${USERNAME}

NAME                              READY     STATUS    RESTARTS   AGE
container-info-5998b79944-gh57z   1/1       Running   0          21s
container-info-5998b79944-t4h6n   1/1       Running   0          21s
container-info-5998b79944-vkmcg   1/1       Running   0          21s
```

Now create the service:

```
cat <<EOF | kubectl create -n lab-06-${USERNAME} -f -
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
EOF
```

Check that the service has been created:

```
kubectl get svc -n lab-06-${USERNAME}

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
container-info   ClusterIP   10.27.247.106   <none>        80/TCP    7m
```

## Task 2: Inspecting your first service

To see more information about the service we can again use the 
`kubectl describe` command:

```
kubectl describe service container-info -n lab-06-${USERNAME}

Name:              container-info
Namespace:         lab-06-trescst
Labels:            <none>
Annotations:       <none>
Selector:          app=container-info
Type:              ClusterIP
IP:                10.27.247.106
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.24.0.15:80,10.24.1.13:80,10.24.2.19:80
Session Affinity:  None
Events:            <none>
```

From the output you can see that a service looks a lot like a loadbalancer, you 
can see the VIP (ClusterIP) and the different backends (Endpoints).

## Task 3: Cleaning up

Clean up any namespaces you might have created during this lab: 
`kubectl delete ns ...`
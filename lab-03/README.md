# Lab 03 - Pods

In this lab we will run our first, very basic, application on Kubernetes.  We 
will basically simply run a single pod as our application.

To make copy/pasting easier we will again export our username first:

```
export USERNAME=<username>
```

## Task 1: Creating a namespace

Before we start with this lab we will create a new namespace:

```
kubectl create namespace lab-03-${USERNAME}
```

Verify that your namespace was created:

```
kubectl get namespaces

NAME             STATUS    AGE
default          Active    1h
kube-public      Active    1h
kube-system      Active    1h
lab-03-trescst   Active    7s
```

## Task 2: Starting your first pod

To run your first pod (the official nginx Docker image), run the following 
command:

```
kubectl run --generator=run-pod/v1 --image=nginx nginx -n lab-03-${USERNAME}

pod "nginx" created
```

The above command will run a single pod that is based on the official nginx 
Docker image.  Run the following command to verify that the pods has been 
created and is in the running state (if the pod is not yet in the running state 
wait a couple of seconds and try to issue the command again):

```
kubectl get pods -n lab-03-${USERNAME}

NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          22s
```

As we have not yet configured any services and/or ingresses we will use a litte 
"hack" to access our pod we just created.

Run the following command to forward the port of the pod (in our case port 80) 
running in our Kubernetes cluster to a port on your laptop (in this case port 
8080).

```
kubectl port-forward nginx 8080:80 -n lab-03-${USERNAME}
```

Now go to your browser and surf to http://localhost:8080, you should be greeted 
with the default nginx welcome page.

If that works you can close the port-foward connection by pressing `CTRL+c`.

## Task 3: Connecting to your pod

To connect to your pod, you can use the following command (notice how it 
resembles the `docker exec` command):

```
kubectl exec -ti nginx bash -n lab-03-${USERNAME}

root@nginx:/#
```

Notice how the prompt changes.  `exec`-ing into a pod is very powerful for 
troubleshooting, but keep in mind that by defaults pods/containers are immutable 
so do not make any changes inside the pods/container.

To exit run the `exit` command.

## Task 4: Pod logs

Again similar to when working with Docker containers, Kubernetes has a built-in 
feature that exposes all stdout/stderr output into logs.  To access those logs 
issue the following command:

```
kubectl logs nginx -n lab-03-${USERNAME}

127.0.0.1 - - [11/Mar/2019:11:40:47 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.52.1" "-"
127.0.0.1 - - [11/Mar/2019:11:40:48 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.52.1" "-"
127.0.0.1 - - [11/Mar/2019:11:40:49 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.52.1" "-"
127.0.0.1 - - [11/Mar/2019:11:40:50 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.52.1" "-"
```

A very handy option of `kubectl logs` is that you can follow them using the `-f` 
option, this is extremely useful when troubleshooting:

```
kubectl logs nginx -n lab-03-${USERNAME} -f
```

Hit `CTRL+c` to exit the logs.

## Task 5: Cleaning up

Because we are working with namespaces it is very easy to clean everything up, 
simple issue the command below:

```
kubectl delete namespace lab-03-${USERNAME}
```
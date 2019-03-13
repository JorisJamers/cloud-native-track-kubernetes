# Lab 09 - Persistent Volumes

Persistent volumes can be used to store stateful/persistent data, this means 
that when a pods is replaced its successor will still have the data available. 
This can be used when you want to run persistent databases in pods and/or when 
you want to share data between different pods.

In this lab we will make a persistent volume that we fill with images, those 
images will then be served by our application (pods).

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-09

namespace "lab-09" created
```

## Task 1 : Generating some stateful/persistent data

First we will need to ssh into the minikube instance.

```
minikube ssh
```

When you are in the minikube instance make sure that you are root. Otherwise you 
might not be have permission to create the required directories and/or files:

```
sudo su -
```

Now we need to create a directory to store our images.

```
mkdir -p /mnt/data/images
```

Let's download a couple of images into our persistent images directory:

```
echo 'Hello the minikube instance' > /mnt/data/index.html
```

Now `exit` the minikube instance.

```
exit

exit
```

## Task 2 : Create a persistent volume

In this task we are going to create the persistent volume. This will put the volume
in a `Available` state. It's not yet bound to a PersistentVolumeClaim at this point.

Create a file `pv-lab-09.yaml` with the following content.

```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: lab-09-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

You can now create the persistent volume with the kubectl command. It will look
like this.

```
kubectl apply -f pv-lab-09.yaml -n lab-09
```

At this point the persistent volume is created. Be sure that it its by using the
get command.

```
kubectl get pv -n lab-09
```

## Task 3 : Claim a persistent Volume

Now we need to create the claim. This way we can bind the persistent volume
claim to a deployment. First we need to create the file `pv-claim-lab-09.yaml`
with the following content.

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: lab-09-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

And apply it with the following command.

```
kubectl create -f pv-claim-lab-09.yaml -n lab-09
```

You can confirm that it is created with the following command.

```
kubectl get pvc -n lab-09
```

## Task 4 : Mount persistent volume in a pod

Now we are able to use the PersistentVolumeClaim in a pod. The following
steps will show you how you can do this.

First we need to create a file that describes our pod. This will be `pv-lab-09-pod.yaml`
with the following content.

```
kind: Pod
apiVersion: v1
metadata:
  name: pv-lab-09-pod
spec:
  volumes:
    - name: lab-09-volume
      persistentVolumeClaim:
       claimName: lab-09-claim
  containers:
    - name: pv-lab-09-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: lab-09-volume
```

Apply the file with the kubectl command.

```
kubectl create -f pv-lab-09-pod.yaml -n lab-09
```

Be sure that the pod is running.

```
kubectl get pod pv-lab-09-pod -n lab-09
```

When you see your pod running you can do a quick port-forward to check out the webserver running in the pod. This will be with the `index.html` file we created at the start of this lab.

```
kubectl port-forward pv-lab-09-pod 8080:80 -n lab-09
```

Visit `localhost:8080` and the message we created in the `/mnt/data/index.html` file will appear!

Clean up the environment.

```
kubectl delete ns lab-09
```

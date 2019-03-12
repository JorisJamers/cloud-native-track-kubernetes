# Lab 07 - Config Maps & Secrets

## Task 1: Create a configmap

With a configmap we are able to pass environment variables to the Deployment we
are creating.

Create a file `container-info.properties` with the following content :

```
# container-info.properties

IMAGE_COLOR=green
```

This will pass an environment variable to the pod. We are now able to create
a configmap with this file.

```
kubectl create configmap container-info-env --from-env-file='./container-info.properties'
```

## Task 2: Add the configmap in the deployment.

We will use the deployment from our previous labs. With some extra parameters
to include the configmap.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info
  labels:
    app: container-info
spec:
  replicas: 1
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
        envFrom:
          - configMapRef:
              name: container-info-env
```     
Copy the above into a file lab-07-deployment.yml

Now we create the new deployment with our environment variables.

```
kubectl create ns lab-07-${USERNAME}
kubectl apply -f lab-07-deployment.yml -n lab-07-${USERNAME}
```

We will expose the service again.

```
kubectl expose deployment container-info --type=NodePort --name=my-container-info-service
-n lab-07-${USERNAME}
```

And connect to the service in the browser.

```
minikube service my-container-info-service -n lab-07-${USERNAME}
```


TODO  --> Create a new application for the environment var? Tweak above ymls if necessary. 

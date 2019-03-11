# Lab 05 - Deployments

Deploying pods is not something you normally do when working with Kubernetes, 
instead of interacting with pods directly you would use a higher lever 
construct.  Deployments are such a higher level construct, they are usually the 
way you would describe your applications.

To make copy/pasting easier we will again export our username first:

```
export USERNAME=<username>
```

## Task 1: Creating a deployment

A basic deployment looks like this:

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
```

Copy the above into a file `lab-05-deployment.yml` and `kubectl apply` it (we 
will create a new namespace first):

```
kubectl create ns lab-05-${USERNAME}
kubectl apply -f lab-05-deployment.yml -n lab-05-${USERNAME}
```

Use `kubectl port-forward` to see the deployed application in your local 
browser:

```
kubectl port-forward container-info 8080:80 -n lab-05-${USERNAME}
```

Check out the page: http://localhost:8080

Kill the `kubectl port-forward` process by pressing `CTRL+c`.

## Task 2: Scaling a deployment

Because we are using a deployment we can very easily scale our application from 
a Kubernetes point of view (you of course need to ensure that your application 
is stateless so it can properly scale).

Scaling your running application is as simple as:

```
kubectl scale deployment container-info --replicas=3 -n lab-05-${USERNAME}

deployment.extensions "container-info" scaled
```

When you do a `kubectl get pods -n lab-05-${USERNAME}` you will see that there 
are 2 additional container-info pods being started.

Scaling down to 1 can be done by re-applying the original YAML:

```
kubectl apply -f lab-05-deployment.yml -n lab-05-${USERNAME}
```

When you do a `kubectl get pods -n lab-05-${USERNAME}` you will see 2 pods are 
being terminated.

> NOTE: scaling up can also be done by editing the "replica" field in the YAML

## Task 3: Changing the image of a deployment

Updating the image (tag) of a deployment is just as easy as scaling a deployment 
and it also can be done using the CLI or by editing the YAML.

```
kubectl set image deployment container-info containerinfo=gluobe/container-info:green -n lab-05-${USERNAME}
```

Or simply edit the YAML and `kubectl apply` it again:

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
        image: gluobe/container-info:green
        ports:
        - containerPort: 80
```

## Task 4: Cleaning up

Clean up any namespaces you might have created during this lab: 
`kubectl delete ns ...`
# Lab 08 - Config Maps & Secrets

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
Copy the above into a file lab-08-deployment.yml

Now we create the new deployment with our environment variables.

```
kubectl create ns lab-08-${USERNAME}
kubectl apply -f lab-08-deployment.yml -n lab-08-${USERNAME}
```

We will expose the service again.

```
kubectl expose deployment container-info --type=NodePort --name=my-container-info-service
-n lab-08-${USERNAME}
```

And connect to the service in the browser.

```
minikube service my-container-info-service -n lab-08-${USERNAME}
```

## Task 3 : Create a secret from literal

Secrets can be handled differently in a kubernetes cluster. There are 3 ways to
handle these secrets. From `literal`, from a `file` and from a `YAML` file.

If you want to create a secret from `literal` it looks like this.

```
kubectl create secret generic lab-08-secret --from-literal=username=${USERNAME}
--from-literal=password=<password> -n lab-08-${USERNAME}
```

You will get confirmation that the secret has been created.

```
secret "lab-08-secret" created
```

This secret can be listed by the command :

```
kubectl get secret -n lab-08-${USERNAME}-literal
```

## Task 4 : Create a secret from a file

The second option we have is creating the secret from a file. First of all create
the files we are going to use in the command.

```
echo -n "${USERNAME}" > ./username.txt
echo -n "<password>" > ./password.txt
```  

If these 2 files are created we can use these to create the secret.

```
kubectl create secret generic lab-08-${USERNAME}-file --from-file=./username.txt --from-file=./password.txt
```

## Task 5 : Create a secret from yaml

Our last option we have is to create the secret directly from a yaml. This looks
like the services, deployments, ... we already created in these labs. First we need
to encrypt our username and password. The following credentials are going to be used
as an example. Create your own username and password for this lab.

```
 echo -n 'admin' | base64
```

This will give you the following output : `YWRtaW4=`

Now do the password.

```
echo -n '1f2d1e2e67df' | base64
```

This will give you the following output : `MWYyZDFlMmU2N2Rm`

The following
content of a yaml will create a secret when we apply it. Let's name the yaml
`secret.yaml`

```
apiVersion: v1
kind: Secret
metadata:
  name: lab-08-secret-yaml
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

In this yaml file we specify the output we generated from the base64 encode. At
this moment you are able to apply the yaml file and the secret will be created.

```
kubectl create -f ./secret.yaml -n lab-08-${USERNAME}
```

We will get the confirmation that the secret has been created.

```
secret "lab-08-secret-yaml" created
```

## Task 6 : Check content of a secret

We can describe a secret just like any other object in kubernetes.

```
kubectl get secret lab-08-secret-yaml -o yaml -n lab-08-${USERNAME}
```

You will see the encrypted username and password in the yaml. This can be decoded with the following command.

```
echo 'YWRtaW4=' | base64 --decode
```

This will output : `admin`

You can do the same for the password.

```
echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
```

This will give you the expected : `1f2d1e2e67df`

This is the password we used before in this lab. 

## Task 6 : Cleanup

```
kubectl delete ns lab-08-${USERNAME}
```
TODO
- Create a new application for the environment var? Tweak above ymls if necessary.
- Create an example to use the secrets

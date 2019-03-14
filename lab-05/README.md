# Lab 05 - YAML

Although relatively easy in concept, this lab is a very important one when 
working with Kubernetes.  In this lab we introduce YAML as a way to describe 
objects in Kubernetes.

If you are not very familiar with YAML check out the website below for a basic
introduction and its relevance for Kubernetes:
https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/

## Task 1: Creating objects using YAML

In the previous labs we have used `kubectl create namespace` to create a
namespace object in Kubernetes, instead we could (and probably should) have also
used YAML.  So what would this look like?

The YAML code to replace `kubectl create namespace lab-05` looks like
this:

```
apiVersion: v1
kind: Namespace
metadata:
  name: lab-05
```

To create the object in Kubernetes simply copy the above content into a file,
for example `lab-05-namespace.yml`.  After that simply issue the following
commmand:

```
kubectl apply -f lab-05-namespace.yml

namespace/lab-05 created
```

## Task 2: Deleting objects using YAML

Deleting an object is even easier, only requirement is that you have the orignal
YAML file at your disposal:

```
kubectl delete -f lab-05-namespace.yml

namespace "lab-05" deleted
```

## Task 3: Multiple objects

When you have multiple objects you have different options:
* you can put multiple objects in the same YAML (using lists)
* you can have multiple YAML files inside a directory and `kubectl apply` the
directory (for example: `kubectl apply -f directory_full_of_yaml/`)

## Task 4: YAML and namespaces

When working with YAML you can use namespaces in different ways:
* metadata in YAML
* option on the command line

When working with metadata your YAML will look like this:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: lab-05
spec:
  containers:
  - name: nginx
    image: nginx
```

You will create the object using:

```
kubectl apply -f lab-05-metadata_namespace.yml

namespace "lab-05" deleted
```

Or you can omit the namespace in the metadata and provide it at the command
line:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

In this case you will create the object using:

```
kubectl apply -f lab-05-without_metadata_namespace.yml -n lab-05
```

Both options have their specific use-cases.

## Task 5: Cleaning up

Clean up the namespace for this lab:

```
kubectl delete ns lab-05

namespace "lab-05" deleted
```
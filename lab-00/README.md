# Lab 00 - Prerequisites

In this lab we will be installing the Kubernetes CLI tools, better know as the 
`kubectl` command.

## Task 1: Downloading the binary

Download the binary from [here](https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl) into 
`/usr/local/bin`:

```
curl -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
```

## Task 2: Make binary executable

In order to properly use the binary we need to make it executable:

```
chmod +x /usr/local/bin/kubectl
```

## Task 3: Basic CLI configuration

TODO => zien hoe we dit fixen met OpenShift login (mogelijks eerst oc installeren en hiervan de oc login gebruiken...)
# Lab 00 - Prerequisites

In this lab we will be installing the Kubernetes CLI tools, better know as the 
`kubectl` command.

## Task 1: Downloading the binary

Download the binary from [here](curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl) into 
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

The easiest way to initially setup the OpenShift CLI is to use the
`oc login` command. It’ll interactively ask you a server URL, username
and password. The information is automatically saved in a CLI configuration file 
that is then used for subsequent commands.

```
oc login
```
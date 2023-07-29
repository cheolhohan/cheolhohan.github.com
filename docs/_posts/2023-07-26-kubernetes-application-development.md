---
layout: post
title:  "Kubernetes Application Development"
date:   2023-07-26 21:14:54
categories: study
tags: kubernetes
---

# Kubernetes Application Development

CKAD Course: https://www.udemy.com/course/certified-kubernetes-application-developer/

## Kubernetes Architecture Recap

### Main Concepts

- Nodes: VMs or physical machines that run the applications and other Kubernetes components
- Cluster: A set of nodes grouped together
- Master: A node with a set of programs to manage the cluster

### Components

- etcd: A distributed key-value store that stores the cluster state
- controller: A control loop that watches the shared state of the cluster through the API server and makes changes to move the current state towards the desired state
- container runtime: A software to run containers, such as Docker
- kubelet: An agent that runs on each node in the cluster. It makes sure that containers are running in a pod

### Master vs Worker Nodes

- Master
  - kube-apiserver
  - etcd
  - controller manager
  - scheduler
- Worker
  - kubelet

### kubectl

Kubernetes command line tool

## Docker vs ContainerD

### CRI (Container Runtime Interface)

Kubernetes is orchestration tool running containers such as Docker, Rocket, ContainerD, etc. So it needs CRI to communicate with container runtime.

### dockershim

The `dockershim` component of Kubernetes allows the use of Docker as a Kubernetes's container runtime instead of using CRI. Now it is deprecated since 1.24 and recommended to use CRI instead by using ContainerD.

### ContainerD

ContainerD is a container runtime that is designed to be embedded into a larger system, such as Kubernetes. It is a daemon that manages the complete container lifecycle of its host system, from image transfer and storage to container execution and supervision to low-level storage to network attachments and beyond.

- ctr
  - ContainerD CLI
  - Not user friendly
  - Only for debugging
- nerdctl
  - ContainerD CLI
  - Recommended interface for controlling ContainerD
  - Very similar to Docker CLI
- crictl
  - CRI CLI
  - Can be used for all CRI-supported runtime

## Pods

- Pod: a collection of one or more Linux® containers, and is the smallest unit of a Kubernetes application running on worker nodes
- Multi-Container Pod: Pod having multiple containers that share the same resources. This is especially useful helper containers such as log collectors, debuggers, sidecar etc.

### Kubectl for Pods

What does really `kubectl` do?

For example of this command, `kubectl run nginx --image nginx`

- Pulling image
- Create container in a pod
- Running the pod on worker node

### Pods with YAML

```yaml
# pod-definition.yaml
apiVersion: v1 # API Version of Kubernetes
kind: Pod # Type of Kubernetes object
metadata: # Metadata of the object
  name: myapp-pod # Name of the object
  labels:
    app: myapp # Label for the object
    type: front-end
spec: # Specification of the object
  containers: # List of containers in the pod. Because pod can have multiple containers
    - name: nginx-container # Name of the container
      image: nginx # Image of the container
```

```bash
kubectl create -f pod-definition.yaml # Create a pod defined in yaml

kubectl get pods # Get all pods you created

kubectl describe pod myapp-pod # Get detailed information about the pod
```

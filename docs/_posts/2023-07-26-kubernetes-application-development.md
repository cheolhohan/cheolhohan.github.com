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

### ReplicaSets

- Replication Controller: A replication controller is a supervisor for long-running pods. It helps to make sure that a specified number of pods are running at any given time. It also helps in scaling up or down the number of replicas of a pod.
- Replica: A copy of a pod for providing high availability
- ReplicationController vs ReplicaSet
  - ReplicationController is older and only supports equality-based selector
  - ReplicaSet is newer and supports set-based selector
  - apiVersion is different, ReplicationController is v1 and ReplicaSet is apps/v1

```yaml
# ReplicationController definition
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```

```yaml
# ReplicaSet definition
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

### Labels and Selectors

- These are required for ReplicaSet to work, but optional for ReplicationController
- The role of ReplicaSet is to monitor the pods and if any of them were to fail, deploy new ones. But how does it know which pods to monitor? This is where labels and selectors come in.

## Kubernetes Deployment

Deployment format is similar to ReplicaSet

```yaml
# Deployment definition
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

## Namespace

### Definitions

- Namespace: A virtual cluster backed by the same physical cluster
  - Default Namespace: The default namespace for objects with no other namespace
  - kube-system Namespace: The namespace for objects created by Kubernetes system
  - kube-public Namespace: The namespace is readable by all users
- Why use Namespace?
  - To separate resources
  - To allocate resources
  - To improve performance
  - To assign access rights to resources
- For examples, you can create a namespace for each team in your organization
  - Dev, QA, Staging, Production, etc.

For connecting to a service in specific namespace, you can use `service-name.namespace-name.svc.cluster.local`
- `mysql.connect("db-service")` will connect to `db-service` in the same namespace
- `mysql.connect("db-service.dev.svc.cluster.local")` will connect to `db-service` in `dev` namespace
  - db-service: service name
  - dev: namespace name
  - svc: service (subdomain)
  - cluster.local: cluster domain

```yaml
# Namespace definition
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
kubectl create -f namespace-definition.yaml # Create a namespace defined in yaml
```

### How to set namespace

`kubectl get pods --namespace=dev`

`kubectl get pods --all-namespaces`

`kubectl config set-context $(kubectl config current-context) --namespace=dev`

### Resource Quota

- Resource Quota: A tool for limiting resource consumption and the number of objects in a namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

## Service

In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster in your cluster. A key aim of Services in Kubernetes is that you don't need to modify your existing application to use an unfamiliar service discovery mechanism.

## Docker commands and arguments recap

```bash
docker run ubuntu
docker ps # Show running containers
docker ps -a # Show all containers
```

### Difference between ENTRYPOINT and CMD

- ENTRYPOINT: The command that is always executed when the container starts
- CMD: The default command that is executed only when you don't specify a command

Let's say you create an docker image called ubuntu-sleeper with the following Dockerfile

```Dockerfile
FROM ubuntu
CMD ["sleep", "5"]
```

```bash
docker run ubuntu-sleeper # This will run sleep 5 command
```

This will start ubuntu image and run sleep program by passing 5 as an argument. So when you run `docker run ubuntu-sleeper`, it will run `sleep 5` command. But what if you want to specify an argument when you run the container? You can do it like this

```Dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
```

```bash
docker run ubuntu-sleeper 10 # This will run sleep 10 command
docker run ubuntu-sleeper # This will return an error because you didn't specify an argument
```

Now, you can pass the parameter. But you will have an error because sleep command always requires an argument. So you can fix it like this

```Dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```bash
docker run ubuntu-sleeper # This will run sleep 5 command because CMD 5 is specified by default when you don't specify an argument
```

## Commands and Arguments in Kubernetes

In order to run the docker with Kubernetes pod, you will need to specify yaml file like this

```yaml
# Pod definition
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  containers:
    - name: ubuntu
      image: ubuntu-sleeper
      args: ["10"] # Overwriting CMD ["5"]
```

What if you want to specify the entrypoint? You can do it like this

```yaml
# Pod definition
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  containers:
    - name: ubuntu
      image: ubuntu-sleeper
      command: ["sleep2.0"] # Overwriting ENTRYPOINT ["sleep"]
      args: ["10"] # Overwriting CMD ["5"]
```
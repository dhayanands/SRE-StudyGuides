#concept #kubernetes

## Overview

- Kubernetes is an orchestration system to deploy and manage **containers**
- Containers are not managed individually; instead, they are part of a larger object called a **Pod**
- A Pod consists of one or more containers which share an IP address, access to storage and namespace
- Kubernetes uses **namespaces** to keep objects distinct from each other, for resource control and multi-tenant considerations.
- Some objects are cluster-scoped, others are scoped to one namespace at a time.
- As the namespace is a segregation of resources, pods would need to leverage **services** to communicate.
- Orchestration is managed through a series of watch-loops, also called **controllers** or operators.
- Each controller interrogates the kube-apiserver for a particular object state, then modifying the object until the declared state matches the current state.
- These controllers are compiled into the **kube-controller-manager**, but others can be added using custom resource definitions.
- The default and feature-filled operator for containers is a **Deployment**.
- A Deployment does not directly work with pods. Instead it manages **ReplicaSets**.
- The ReplicaSet is an operator which will create or terminate pods by sending out a **podSpec**.
- The podSpec is sent to the **kubelet**, which then interacts with the container engine, Docker by default, to spawn or terminate a container until the requested number is running.
- The **service** operator requests existing IP addresses and endpoints and will manage the network connectivity based on labels. These are used to communicate between pods, namespaces, and outside the cluster.
- There are also **Jobs** and **CronJobs** to handle single or recurring tasks, among others.
- To easily manage thousands of Pods across hundreds of nodes can be a difficult task. To make management easier, we can use **labels**, arbitrary strings which become part of the object metadata. These can then be used when checking or changing the state of objects without having to know individual names or UIDs.
- Nodes can have **taints** to discourage Pod assignments, unless the Pod has a **toleration** in its metadata
- There is also space in metadata for **annotations** which remain with the object but cannot be used by Kubernetes commands. This information could be used by third-party agents or other tools
- **DaemonSet** makes sure the local Pod is deleted. DaemonSets are often used for logging, metrics and security pods, and can be configured to avoid nodes.
- **StatefulSet** is the workload API object used to manage stateful applications. Pods deployed using a StatefulSet use the same Pod specification. How this is different than a Deployment is that a StatefulSet considers each Pod as unique and provides ordering to Pod deployment. The default deployment scheme is sequential, starting with 0, such as app-0, app-1, app-2, etc. A following Pod will not launch until the current Pod reaches a running and ready state. They are not deployed in parallel.
- **Horizontal Pod Autoscalers (HPA)** automatically scale Replication Controllers, ReplicaSets, or Deployments based on a target of 50% CPU usage by default. The usage is checked by the kubelet every 30 seconds, and retrieved by the Metrics Server API call every minute. HPA checks with the Metrics Server every 30 seconds. Should a Pod be added or removed, HPA waits 180 seconds before further action.
- **Cluster Autoscaler (CA)** adds or removes nodes to the cluster, based on the inability to deploy a Pod or having nodes with low utilization for at least 10 minutes.
- **Vertical Pod Autoscaler** will adjust the amount of CPU and memory requested by Pods. -- still under development
- **Jobs** are part of the batch API group. They are used to run a set number of pods to completion. If a pod fails, it will be restarted until the number of completion is reached.
- **Cronjobs** work in a similar manner to Linux jobs, with the same time syntax. There are some cases where a job would not be run during a time period or could run twice; as a result, the requested Pod should be idempotent.

  

## Core-Components Quick Recap

- Control Pane consists of:
	  - API Server
	  - etc (cluster store)
	  - Scheduler
	  - Control manager

### API Server

- API Server is the primary access point for administrative and cluster operations
- It is based on client server architecture
- Works on RESTful API over HTTP using JSON
- Client submits requets over HTTP/HTTPS and server reponds to the requests

### etcd

- API Server is Stateless and all configurations are stored in etcd
- etcd is a key-value data store

### Scheduler

- Scheduler tells kubernetes which nodes to start the Pods on based on the Pods resource requirements

### Controller Manager

- Controller Manager has the job of implementing lifecycle funtions of controllers
- Controller are responsible for keeping things in the desired state


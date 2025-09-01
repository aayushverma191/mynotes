---
longform:
  format: single
  title: Design
---
![[k8s-architecture.png]]

How do you interact with cluster ?

How to 
	- schedule pod ?
	- monitor ?
	- re-schedule / restart pod ?
	- join a new node ?
# Cluster:

A Kubernetes cluster is a set of machines (physical or virtual) that work together to run containerized applications. The cluster is managed by Kubernetes and consists of a control plane and one or more worker nodes.

# Control Plane:

The control plane is responsible for managing the Kubernetes cluster. It maintains the desired state of the cluster, such as which applications are running and their configurations. The control plane consists of several key components:

#### **API Server (kube-apiserver):** 

This is the front end of the Kubernetes control plane. It exposes the Kubernetes API and is the entry point for commands and configurations. All communication with the cluster goes through the API server.

#### **etcd:**

A distributed key-value store used to store all cluster data, including configurations and state. It is a highly available and consistent storage layer.

If your Kubernetes cluster uses etcd as its backing store, make sure you have a [back up](https://v1-32.docs.kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster) plan for the data.

#### **Controller Manager (kube-controller-manager):** 

This component runs controllers, which are responsible for ensuring that the cluster's state matches the desired state. Controllers handle tasks like replication, scaling, and other routine operations.

#### **Scheduler (kube-scheduler):** 

The scheduler watches for newly created pods and assigns them to nodes based on resource availability and other constraints.

#### **Cloud Controller Manager (cloud-controller-manager):** 

This component interacts with the underlying cloud provider (if applicable) to manage resources like load balancers and storage volumes.

# Worker Nodes

Worker nodes are the machines where application containers run. Each worker node includes several components:

#### **Kubelet:** 

An agent that runs on each worker node. It ensures that containers are running in a Pod according to the specifications provided by the control plane.

-  It interacts with both - container and node
-  It starts the pod with a container inside

#### **Kube-Proxy:** 

A network proxy that maintains network rules on nodes. It allows communication between Pods and services by handling networking and load balancing.

#### **Container Runtime:** 

The software responsible for running containers, such as Docker, containerd, or CRI-O.


# Request flow for a new pod

![[Screenshot 2024-07-26 at 12.39.30 PM.png]]

# What happen when a pod die

![[Screenshot 2024-07-26 at 12.42.09 PM.png]]
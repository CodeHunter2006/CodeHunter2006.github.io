---
layout: post
title: "Kubernetes关键概念"
date: 2021-10-08 22:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/2021-10-08-Kubernetes_Concepts_1.svg)
记录 K8S 常用概念，基础请参考[Kubernetes 基础概念](/2019/02/01/Kubernetes_basic/)

- Control Plane
  The container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers.
  The control plane's components make global decisions about the cluster (for example, scheduling),
  as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfied).

- kube-apiserver
  The API server is a component of the Kubernetes control plane that exposes the Kubernetes API.
  The API server is the front end for the Kubernetes control plane.

- etcd
  Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

- kube-scheduler
  Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.
  **Factors** taken into account for scheduling decisions include:

  - individual and collective resource requirements
  - hardware/software/policy constraints
  - affinity and anti-affinity specifications
  - data locality
  - inter-workload interference
  - deadlines

- Controllers
  In Kubernetes, controllers are control loops that watch the state of your cluster,
  then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.

- kube-controller-manager
  Control plane component that runs controller processes.
  Some types of these controllers are:

  - Node controller: Responsible for noticing and responding when nodes go down.
  - Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
  - Endpoints controller: Populates the Endpoints object (that is, joins Services & Pods).
  - Service Account & Token controllers: Create default accounts and API access tokens for new namespaces.

- cloud-controller-manager
  The following controllers can have cloud provider dependencies:

  - Node controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
  - Route controller: For setting up routes in the underlying cloud infrastructure
  - Service controller: For creating, updating and deleting cloud provider load balancers

- Container runtime
  The container runtime is the software that is responsible for running containers. like Docker.

- Kubernetes API
  The core of Kubernetes' control plane is the API server.
  The Kubernetes API lets you query and manipulate the state of API objects in Kubernetes (for example: Pods, Namespaces, ConfigMaps, and Events).
  Most operations can be performed through the kubectl command-line interface or other command-line tools, such as kubeadm, which in turn use the API.
  However, you can also access the API directly using REST calls.

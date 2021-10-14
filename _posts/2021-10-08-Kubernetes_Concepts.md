---
layout: post
title: "Kubernetes关键概念"
date: 2021-10-08 22:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/2021-10-08-Kubernetes_Concepts_1.svg)
记录 K8S 常用概念。

基础请参考[Kubernetes 基础概念](/2019/02/01/Kubernetes_basic/)

# Overview

- Control Plane
  The container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers.
  You can use finalizers to control garbage collection of resources. The control plane's components make global decisions about the cluster (for example, scheduling),
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

- API groups and versioning
  To make it easier to eliminate fields or restructure resource representations,
  Kubernetes supports multiple API versions, each at a different API path, such as `/api/v1` or `/apis/rbac.authorization.k8s.io/v1alpha1`.
  API groups make it easier to extend the Kubernetes API.
  The API group is specified in a REST path and in the apiVersion field of a serialized object.
  You can config API groups by setting flag to API server, like `--runtime-config=batch/v1=false`

- Kubernetes objects
  Kubernetes objects are persistent entities in the Kubernetes system.
  Kubernetes uses these entities to represent the state of your cluster.
  A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists.

  - Deployment
    an object that can represent an application running on your cluster.

- Object Spec
  Describe what state you desire for the object. The precise format of the object spec is different for every Kubernetes object

- Object Management

  - Imperative commands
    operates directly on live objects in a cluster.
  - Imperative object configuration
    The file specified must contain a **full** definition of the object in YAML or JSON format.
  - Declarative object configuration
    user operates on object configuration files stored locally, however the user does not define the operations to be taken on the files.
    Create, update, and delete operations are automatically detected per-object by kubectl.

- Object Names and IDs
  Each object in your cluster has a Name that is unique for that type of resource.
  Every Kubernetes object also has a UID that is unique(UUID) across your whole cluster(history).

- Namespaces
  Kubernetes supports multiple virtual clusters backed by the same physical cluster.
  These virtual clusters are called namespaces.

  - The Kubernetes control plane sets an immutable label `NamespaceDefaultLabelName`,
    The value of the label is the namespace name.

- Labels
  Labels are key/value pairs that are attached to objects, such as pods.
  Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system.
  Labels can be used to organize and to select subsets of objects.
  Labels enable users to map their own organizational structures onto system objects in a loosely coupled fashion, without requiring clients to store these mappings.

- Label selectors
  Unlike names and UIDs, labels do not provide uniqueness.
  In general, we expect many objects to carry the same label(s).

  - two types of selectors:
    - equality-based
      `=,==,!=`
    - set-based
      `in,notin,exists`
  - Similarly the comma separator acts as an AND operator.
    Set-based requirements can be mixed with equality-based requirements.
    For example: `partition in (customerA, customerB),environment!=qa`.

- Set references in API objects
  Some Kubernetes objects, such as services and replicationcontrollers, also use label selectors to specify sets of other resources, such as pods.

- Annotations
  You can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects.
  Clients such as tools and libraries can retrieve this metadata.
  You can use either labels or annotations to attach metadata to Kubernetes objects.
  abels can be used to select objects and to find collections of objects that satisfy certain conditions.
  In contrast, annotations are not used to identify and select objects.
  The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.
  Annotations, like labels, are key/value maps.

- Field Selectors
  Field selectors let you select Kubernetes resources based on the value of one or more resource fields.
  `=,==,!=`

- **???** Finalizers
  Finalizers are namespaced keys that tell Kubernetes to wait until specific conditions are met before it fully deletes resources marked for deletion.
  Finalizers alert controllers to clean up resources the deleted object owned.
  You can use finalizers to control garbage collection of resources.

- Owner references
  The Job controller also adds owner references to those Pods, pointing at the Job that created the Pods.
  If you delete the Job while these Pods are running, Kubernetes uses the owner references (not labels) to determine which Pods in the cluster need cleanup.
  Dependent objects have a metadata.ownerReferences field that references their owner object.
  Kubernetes sets the value of this field automatically for objects that are dependents of other objects like ReplicaSets, DaemonSets, Deployments, Jobs and CronJobs, and ReplicationControllers.

## Workloads

A workload is an application running on Kubernetes.
Whether your workload is a single component or several that work together, on Kubernetes you run it inside a set of pods.
In Kubernetes, a Pod represents a set of running containers on your cluster.

- Kubernetes provides several built-in workload resources:

  - Deployment and ReplicaSet
    Deployment is a good fit for managing a stateless application workload on your cluster.
  - StatefulSet
    StatefulSet lets you run one or more related Pods that do track state somehow.
    For example, if your workload records data persistently, you can run a StatefulSet that matches each Pod with a PersistentVolume.
  - DaemonSet
    DaemonSet defines Pods that provide node-local facilities.
    Every time you add a node to your cluster that matches the specification in a DaemonSet, the control plane schedules a Pod for that DaemonSet onto the new node.
  - Job and CronJob
    define tasks that run to completion and then stop.
    Jobs represent one-off tasks, whereas CronJobs recur according to a schedule.

## Services, Load Balancing, and Networking

- Kubernetes networking addresses four concerns:

  - Containers within a Pod use networking to communicate via loopback.
  - Cluster networking provides communication between different Pods.
  - The Service resource lets you expose an application running in Pods to be reachable from outside your cluster.
  - You can also use Services to publish services only for consumption inside your cluster.

- Services
  An abstract way to expose an application running on a set of Pods as a network service.These Pods are exposed through endpoints.
  Kubernetes gives Pods their own IP addresses and a single **DNS name** for a set of Pods, and can load-balance across them.

  - Supported protocols：TCP UDP SCTP HTTP PROXY
  - Discovering services："environment variables" and "DNS"

- Topology-aware traffic routing
  The label matching between the source and destination lets you, as a cluster operator, designate sets of Nodes that are "closer" and "farther" from one another.

- DNS
  Kubernetes creates DNS records for services and pods. You can contact services with consistent DNS names instead of IP addresses.

- Exposing the Service
  Kubernetes supports two ways of doing this:

  - NodePorts
  - LoadBalancers

![Ingress](/assets/images/2021-10-08-Kubernetes_Concepts_2.png)

- Ingress
  An API object that manages external access to the services in a cluster, typically HTTP.
  Ingress may provide load balancing, SSL termination and name-based virtual hosting.

- Ingress Controllers
  In order for the Ingress resource to work, the cluster must have an ingress controller running.

- EndpointSlices
  EndpointSlices provide a simple way to track network endpoints within a Kubernetes cluster.
  They offer a more scalable and extensible alternative to Endpoints.
  The control plane automatically creates EndpointSlices for any Kubernetes Service that has a selector specified.

- Service Internal Traffic Policy
  Service Internal Traffic Policy enables internal traffic restrictions to only route internal traffic to endpoints within the node the traffic originated from.
  The "internal" traffic here refers to traffic originated from Pods in the current cluster.
  This can help to reduce costs and improve performance.

## Storage

## Task

- Horizontal Pod Autoscaler
  The Horizontal Pod Autoscaler automatically scales the number of Pods in a replication controller, deployment, replica set or stateful set based on observed CPU utilization (or, with custom metrics support, on some other application-provided metrics).
  Note that Horizontal Pod Autoscaling does not apply to objects that can't be scaled, for example, DaemonSets.

## API

- GV GVK GVR
  - GV
    Api Group & Version
  - GVK
    Group Version Kind
  - GVR
    Group Version Resource

## Others

- The Metrics API
  Through the Metrics API, you can get the amount of resource **currently** used by a given node or a given pod.

- ConfigMap
  A ConfigMap is an API object used to store non-confidential data in key-value pairs.
  Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.
  (The data stored in a ConfigMap cannot exceed 1 MiB.)

- Secrets
  A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.
  Such information might otherwise be put in a Pod specification or in a container image.

- Garbage Collection
  Garbage collection is a collective term for the various mechanisms Kubernetes uses to clean up cluster resources.
  This allows the clean up of resources like the following:

  - Failed pods
  - Completed Jobs
  - Objects without owner references
  - Unused containers and container images
  - Dynamically provisioned PersistentVolumes with a StorageClass reclaim policy of Delete
  - Stale or expired CertificateSigningRequests (CSRs)
  - Nodes deleted in the following scenarios:
    - On a cloud when the cluster uses a cloud controller manager
    - On-premises when the cluster uses an addon similar to a cloud controller manager
  - Node Lease objects

- cordon
  Marking a node as unschedulable prevents the scheduler from placing new pods onto that Node but does not affect existing Pods on the Node.
  This is useful as a preparatory step before a node reboot or other maintenance.
  `kubectl cordon $NODENAME`

- drain
  You can use kubectl drain to **safely**(graceful termination) evict all of your pods from a node before you perform maintenance on the node.
  `kubectl drain <node name>`
  Once it returns (without giving an error), you can power down the node.
  If you leave the node in the cluster during the maintenance operation, you need to run `kubectl uncordon <node name>` resume scheduling new pods onto the node.

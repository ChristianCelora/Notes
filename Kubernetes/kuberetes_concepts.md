# Key elements

### Containers

Technology for packaging an application along with its runtime dependencies. A *container image* is a ready-to-run software package containing everything needed to run an application: the code and any runtime it requires, application and system libraries, and default values for any essential settings. Containers are intended to be stateless and immutable. If you want to make changes you need to build a new image

*Container runtimes* are a fundamental component that empowers Kubernetes to run containers effectively. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.

### Pod

A pod is an atomic unit that runs one or more containers. These containers share resources such as file volumes and network interfaces in common. Pods are the basic unit of scheduling in Kubernetes: all containers in a pod are guaranteed to run on the same node that the pod is scheduled on.

Each pod has its own IP address, and a pod on one node should be able to access a pod on another node using the pod’s IP. Containers on a single node can communicate easily through a local interface. Communication between pods is more complicated, however, and requires a separate networking component that can transparently route traffic from a pod on one node to a pod on another.

This functionality is provided by pod network plugins.

### Cluster

A Kubernetes cluster consists of a *control plane* plus a set of *worker* machines, called *nodes*, that run containerized applications. Every cluster needs at least one worker node in order to run Pods.

Below is the cluster architecture
[![Badge](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)](https://kubernetes.io/docs/concepts/architecture/)

Below are described each components:
- kube-apiserver: exposes the Kubernetes API. It scales horizontally adding new instances of it, and load balancing traffic between them.
- etcd: highly-available key value store
- kube-scheduler: watches for newly created Pods with no assigned node, and selects a node for them to run on
- kube-controller-manager: runs controller processes
- cloud-controller-manager: link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster
- kubelet: It makes sure that containers are running in a Pod
- kube-proxy (optional): maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster
- Container runtime (CRI): described in [[#Containers]]


# Key elements

## Cluster Architecture

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

### ReplicaSet

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. Replica sets objects are used when you require custom update orchestration or don't require updates at all. Otherwise, use high-level deployments to manage both pods and replica sets.

### Deployments

A Deployment provides declarative updates for Pods and ReplicaSets. You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate.

### StatefulSets

StatefulSet is the workload API object used to manage stateful applications. Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods. 
- Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec.
- Unlike a Deployment, a StatefulSet maintains a sticky identity for each of its Pods.

StatefulSets are valuable for applications that require one or more of the following:
- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.
- Persistence across Pod (re)scheduling

### Jobs

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

## Config

### Secret

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key

> Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd).
> In order to safely use Secrets, take at least the following steps:
>   1. Enable Encryption at Rest for Secrets.
>   2. Enable or configure RBAC rules with least-privilege access to Secrets.
>   3. Restrict Secret access to specific containers.
>   4. Consider using external Secret store providers.

### ConfigMap

A ConfigMap is an API object used to store non-confidential data in key-value pairs. It allows multiple values for a single key

```yaml
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
# file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5 
```

> ConfigMap does not provide secrecy or encryption. If the data you want to store are confidential use a Secret, or use additional (third party) tools to keep your data private.

There are four different ways that you can use a ConfigMap to configure a container inside a Pod:
1. Inside a container command and args
2. Environment variables for a container
3. Add a file in read-only volume, for the application to read
4. Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

# Examples

### Pod

Pod running nginx:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

To create the Pod shown above, run the following command:

```sh
kubectl apply -f <yaml_file_path>
```

Usually you don't need to create Pods directly, even singleton Pods. Instead, create them using workload resources such as Deployment or Job.

To retrieve the pods use 
```sh
kubectl get pods
kubectl describe pods
kubectl describe pods <pod_name>
```

Check another pod template in the [[#Jobs]] section below
### Jobs

Pod template for a simple Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

To create the job run

```sh
kubectl apply -f <yaml_file_path>
```

To check job

```sh
kubectl describe job hello
kubectl get job hello
kubectl logs jobs/hello # get logs of job named hello
```

To get all pods created by the job run
```sh
pods=$(kubectl get pods --selector=batch.kubernetes.io/job-name=hello --output=jsonpath='{.items[*].metadata.name}')
echo $pods
```
### Deployments

The following is an example of a Deployment. It creates a ReplicaSet to bring up three `nginx` Pods:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

In this example:
- We created a deployment named nginx-deployment. This name will become the basis for the ReplicaSets and Pods which are created later
- The Deployment creates a ReplicaSet that creates three replicated Pods, indicated by the `.spec.replicas` field.
- The `.spec.selector` field defines how the created ReplicaSet finds which Pods to manage. In this case, you select a label that is defined in the Pod template (`app: nginx`). More complex rules are possible
  - The `.spec.template` field contains the following sub-fields:
	- The Pods are labeled `app: nginx`using the `.metadata.labels` field.
    - The Pod template's specification, or `.spec` field, indicates that the Pods run one container, `nginx`
    - Create one container and name it `nginx` using the `.spec.containers[0].name` field.

To create the deployment run

```sh
kubectl apply -f <yaml_file_path>
```

To check the created deployment use

```sh
kubectl get deployments # summary table of all deployments
kubectl describe deployments # all deployments
kubectl describe deployments <deployment_name> # single deployment
```

### StatefulSet

nginx Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

A StatefulSet, named web, running 3 replicas of the service above

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.24
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

The volumeClaimTemplates will provide stable storage using PersistentVolumes provisioned by a PersistentVolume Provisioner.

For a StatefulSet with N replicas, each Pod in the StatefulSet will be assigned an integer ordinal, that is unique over the Set. By default, pods will be assigned ordinals from 0 up through N-1. Each Pod in a StatefulSet derives its hostname from the name of the StatefulSet and the ordinal of the Pod. The pattern for the constructed hostname is `$(statefulset name)-$(ordinal)`

### Secrets

Secrets are generated through cli o yaml config file
 - cli
```sh
kubectl create secret generic backend-user --from-literal=backend-username='backend-admin'
```

 - yaml
```yaml
apiVersion: v1
kind: Secret
type: Opaque # the secret type depends on its use case
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
```


### ConfigMap

Here's an example ConfigMap used as env variables:

The following ConfigMap (myconfigmap.yaml) stores two properties: username and access_level:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  username: k8s-admin
  access_level: "1"
```

The following command will create the ConfigMap object:

```shell
kubectl apply -f myconfigmap.yaml
```

The following Pod consumes the content of the ConfigMap as environment variables:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: myconfigmap
```

If you need use them as file, here an example of a Pod that mounts a ConfigMap in a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```

Modify your image or command line so that the program looks for files in that directory. Each key in the ConfigMap `data` map becomes the filename under `mountPath`
If there are multiple containers in the Pod, then each container needs its own `volumeMounts` block, but only one `.spec.volumes` is needed per ConfigMap.

# Features

## pod-to-pod communication

One of the key feature is the communication between pods. Each pod has its own IP, assigned by the network plugin. An example of a job with pod to pod communication is [here](https://kubernetes.io/docs/tasks/job/job-with-pod-to-pod-communication/)

## External access to services

### Ingress \[not-recommended]

Ingress makes your HTTP (or HTTPS) network service available using a protocol-aware configuration mechanism, that understands web concepts like URIs, hostnames, paths, and more. The Ingress concept lets you map traffic to different backends based on rules you define via the Kubernetes API.

Although Ingress is still available, it's recommended to use Gateway API, described below

### Gateway API

Gateway API is an official Kubernetes project focused on L4 and L7 routing in Kubernetes. This project represents the next generation of Kubernetes Ingress, Load Balancing, and Service Mesh APIs. It has been designed to be generic, expressive, and role-oriented.

![Badge](https://gateway-api.sigs.k8s.io/images/resource-model.png)

key elements:
 - **GatewayClass**: defines a set of gateways with common configuration and managed by a controller that implements the class
 - **Gateway**: defines a point of access at which traffic can be routed across multiple contexts
 - **HTTPRoute** is a Gateway API type for specifying routing behaviour of HTTP requests from a Gateway listener to an API object, i.e. Service
 - **GRPCRoute:** defines gRPC-specific rules for mapping traffic from a Gateway listener to a representation of backend network endpoints

A typical Gateway resource example:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
  namespace: example-namespace
spec:
  gatewayClassName: example-class
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    hostname: "www.example.com"
    allowedRoutes:
      namespaces:
        from: Same
```

A typical HTTPRoute example:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
  - name: example-gateway
  hostnames:
  - "www.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /login
    backendRefs:
    - name: example-svc
      port: 8080
```

In this example, HTTP traffic from Gateway `example-gateway` with the "Host" header set to `www.example.com` and the request path specified as `/login` will be routed to Service `example-svc` on port `8080`.

Here is a simple example of HTTP traffic being routed to a Service by using a Gateway and an HTTPRoute:
![Badge](https://kubernetes.io/docs/images/gateway-request-flow.svg)



# Kubernetes Quick Notes
I've been reading k8s official documents and it is a bit too sophisticated to read so I decided to write some quick notes for future lookup.
## Master node processes
### kube-apiserver
The Kubernetes API server validates and configures data for the api objects which include pods, services, replicationcontrollers, and others.
### kube-controller-manager
A daemon that embeds the core control loops shipped with Kubernetes, a control loop is a non-terminating loop that regulates the state of the system.
### kube-scheduler
A policy-rich, topology-aware, workload-specific function that significantly impacts availability, performance, and capacity.

## Non-master node processes
### kubelet
The primary node agent that runs on each node, it ensures the container is running as config expects.
### kube-proxy
Below.

## K8s objects
### pod
- A pod represents a running instance of ur application, it encapsulates an app container, storage resources, network config and other options. It is also the smallest deployable units.
[link](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
- Life cycle: pending, running, succeeded(in termination), failed(i t), unknown(networking), completed(terminatable), crashLoopBackOff.
### service
An abstraction which defines a logical set of pods and policy.
- kube-proxy: Every node in k8s cluster has one, responsible for implementing a form of virtual IPs. [link](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)
- service publishing types: ClusterIP(expose on a cluster IP), NodePort(expose at a static port on each IP), LoadBalancer(expose externally using cloud provider's LB), ExternalName(map service to a DNS)
- DNS: [link](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- Ingress: API object which manages external access to services in the cluster, such as giving service externally-reachable URLs, LB traffic, terminate SSL, name based virtual hosting. Ingress controller is responsible for fulfilling the ingress, usually with a LB(usually nginx) and deployed as a pod.

### volume
Pods are ephemeral, and data is wiped out when reclaimed. Volume is essentially a directory which mounts somewhere else on the disk, it is used to preserve data that should be kept upon pod restart.

[types of volumns](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes)
### namespace

## Controllers
Controllers can create and manage pods for you, handling replication, rollout and failover mechanisms at cluster scope.
### ReplicaSet
A ReplicaSet’s purpose is to maintain a stable set of replica Pods running at any given time. It creates and destroys Pods dynamically.
### Deployment
It provides declarative updates for pods and replicaSets.
[link](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
### StatefulSet
A workload API object used to manage stateful applications.
[link](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
### DaemonSet
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod.
[linl](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
### Job

## Control panel
### Master
- Responsible for maintaining the desired state for cluster, which `kubectl` talks to directly.
- It's a collection of processes, usually running on a single node, so it's also called as master node.
### Node
The basic worker component of running applications. Can be VM or bare metal.
#### Node status
- Address: hostname, external-ip, internal-ip
- Condition/Status(booleans): OutOfDisk, Ready, MemoryPressure, PIDPressure, DiskPressure, NetworkUnavailable
#### Capacity
CPU, memory, max number of schedule-able pods.
#### Info
General info of kernel verson, k8s version, docker version and os name.
### Management
[link](https://kubernetes.io/docs/concepts/architecture/nodes/#management)
### API object
[link](https://kubernetes.io/docs/concepts/architecture/nodes/#api-object)

## Master-Node Comm
### Cluster to Master
[link](https://kubernetes.io/docs/concepts/architecture/master-node-communication/#cluster-to-master)
### Master to Cluster
[link](https://kubernetes.io/docs/concepts/architecture/master-node-communication/#master-to-cluster)

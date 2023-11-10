## CKA notes

### ETCD Basics
Whereas the commands are different in version 3

```
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put
```

To set the right version of API set the environment variable ETCDCTL_API command

```
export ETCDCTL_API=3
```


When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.


Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

```
--cacert /etc/kubernetes/pki/etcd/ca.crt
--cert /etc/kubernetes/pki/etcd/server.crt
--key /etc/kubernetes/pki/etcd/server.key
```

So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form for `etcd-docker-desktop`:

```
kubectl exec etcd-docker-desktop -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --endpoints 127.0.0.1:2379 --cert=/run/config/pki/etcd/server.crt --key=/run/config/pki/etcd/server.key --cacert=/run/config/pki/etcd/ca.crt"
```
### Kube-api Server Operations

The Kubernetes API server is the central management entity of the cluster. It processes REST operations, validates them, and updates the corresponding objects in etcd.

1. **Authenticate User**: The API server must confirm the identity of the requester to ensure they are permitted to perform the requested operation.
2. **Validate Request**: After authentication, the request is validated to ensure it is well-formed and authorized according to the cluster's policies.
3. **Retrieve Data**: If the request is a query for information, the API server retrieves the requested data from etcd, the cluster's consistent and highly-available key value store.
4. **Update ETCD**: For operations that modify the cluster state, the API server will persist these changes to etcd.
5. **Scheduler**: When it comes to running pods, the API server interacts with the scheduler to place the pods on appropriate nodes based on scheduling rules.
6. **Kubelet**: Each node has a kubelet service that communicates with the API server to manage the containers running on that node and ensure they are healthy.

### Kube Controller Manager

The Kube Controller Manager is a component of Kubernetes that runs controller processes. It is responsible for ensuring the state of the cluster matches the desired state expressed by the various workloads and user commands. Some of the key controllers it manages include:

1. **Node Controller**: Responsible for noticing and responding when nodes go down. It ensures that nodes within the cluster are functioning correctly.
2. **Replication Controller**: Ensures that the correct number of pod replicas are running for each replication controller object in the system. It is crucial for maintaining the desired state of the system by creating or killing pods as required.
3. **Endpoints Controller**: Populates the Endpoints object (that is, joins Services & Pods).
4. **Service Account & Token Controllers**: Create default accounts and API access tokens for new namespaces.

Additional controllers include:

- **Deployment Controller**: Manages the deployment of replicated pods, allowing for easy updates and rollbacks.
- **DaemonSet Controller**: Ensures that all (or some) nodes run a copy of a pod, which is useful for cluster-wide services.
- **Job Controller**: Watches for Job objects that represent one-off tasks and creates Pods to run those tasks to completion.

Each of these controllers is responsible for specific aspects of cluster management and ensures high availability, resilience, and correct application execution within the Kubernetes ecosystem.


### Kube Scheduler

The Kube Scheduler is one of the core components of Kubernetes responsible for workload distribution across the nodes in a cluster. It watches for newly created Pods with no assigned node and selects a node for them to run on.

#### Functions of the Kube Scheduler

- **Resource Requirement Check**: The scheduler ensures that a node has enough resources (CPU, memory, disk, etc.) to meet the requirements of the Pod.
- **Quality of Service (QoS)**: It considers the quality of service requirements that are set for the Pod, such as whether the Pod needs to be co-located with other Pods on the same node.
- **Affinity and Anti-affinity**: The scheduler can check for node/pod affinities or anti-affinities to determine where Pods can or cannot be placed.
- **Data Locality**: It places Pods as close as possible to where their data is stored, considering the data locality requirements.
- **Inter-Pod Affinity**: Takes into account the placement of Pods relative to other Pods, following rules defined in the Pod specifications.
- **Constraints**: Honors all constraints that a Pod might have, such as node selectors or taints and tolerations.
- **Load Balancing**: Works to balance out the workload on the cluster nodes to ensure optimal performance.

The scheduler's ability to place Pods on the appropriate nodes is crucial for the efficient operation of a Kubernetes cluster. It uses a combination of policies and strategies to make scheduling decisions.

### Kubelet

The Kubelet is the primary "node agent" that runs on each node in a Kubernetes cluster. It ensures that containers are running in a Pod.

#### Key Responsibilities of the Kubelet

- **Pod Lifecycle Management**: It manages the lifecycle of the pods and containers running on a machine.
- **Node Registration**: Each kubelet registers the node it is running on with the API server, using one of: the hostname; a flag to override the hostname; or specific logic for a cloud provider.
- **Pod Spec Compliance**: The kubelet reads the PodSpecs provided by the API server and ensures that the described containers are started and running.
- **Resource Usage Reporting**: It also reports back to the master the resource usage of the pods it manages.
- **Health Checking**: It performs health checks to identify and resolve issues with the node itself, or with the containers it is managing.
- **Node Resource Management**: The kubelet also manages the node's resources appropriately, ensuring that the pods are not using more than their share of system resources.
- **Kubernetes API Communication**: It interacts with the apiserver to report the status of the node and the pods.
- **Container Log Management**: It manages the lifecycle of logs produced by the containers.

#### Operation

- The kubelet works in terms of a PodSpec. A PodSpec is a YAML or JSON object that describes a pod.
- It takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy.
- The kubelet doesn't manage containers that were not created by Kubernetes.

#### Communication

- The kubelet communicates with the container runtime using the Container Runtime Interface (CRI) to perform operations like pulling container images, starting and stopping containers, fetching container logs, etc.

In summary, the kubelet is the pivotal component that communicates with the container runtime and the master components, ensuring that containers are running as expected.

### Kube-proxy

The `kube-proxy` is a key network component in Kubernetes, running on each node in the cluster. It maintains network rules on nodes, allowing network communication to your Pods from network sessions inside or outside of your cluster.

#### Key Responsibilities of Kube-proxy

- **Service Abstraction**: It provides a network proxy and load balancing for a service on a single worker node.
- **Network Rules Management**: `kube-proxy` manages the network rules on nodes. These rules allow network communication to your Pods from network sessions inside or outside of your cluster.
- **Connection Forwarding**: It uses the operating system packet filtering layer if there is one and it's available. Otherwise, `kube-proxy` forwards the traffic itself.

#### Working Modes

`kube-proxy` can run in one of several modes:

- **User space mode**: `kube-proxy` manages a user space proxy for TCP and UDP connections. Although this is slower and outdated, it provides a reference implementation and an example for more sophisticated proxies.
- **Iptables mode**: This uses the native packet filtering capabilities of Linux to redirect traffic to Pods. This is the default as it provides better performance and scalability.
- **IPVS mode**: Built on the Netfilter framework, IPVS provides more sophisticated load balancing features, such as traffic sharding and a richer set of balancing algorithms.

###### Operation

- When Kubernetes creates a service, it creates a set of network rules on the node to handle traffic that's directed to the service's virtual IP address.
- `kube-proxy` ensures that these network rules are maintained and updated according to the Kubernetes API.
- As services and Pods come up or go down, `kube-proxy` updates the routing rules to ensure accurate traffic routing.

In summary, `kube-proxy` is responsible for request forwarding and load balancing of services in Kubernetes, helping to ensure that service discovery and connectivity are maintained efficiently.

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

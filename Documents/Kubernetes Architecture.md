# Kubernetes Architecture Diagram

```
+--------------------------------------------------------------+
|                       Kubernetes Cluster                     |
|          Control Plane                   Data Plane          |
|  +-----------------------------+    +---------------------+  |
|  |                             |    |                     |  |
|  |     Master Node             |    |   Worker Node 1     |  |
|  |                             |    |                     |  |
|  |  +-----------------------+  |    |  +---------------+  |  |
|  |  |  API Server           |  |    |  |   Pod         |  |  |
|  |  +-----------------------+  |    |  |  +-----------+|  |  |
|  |  |  Scheduler            |  |    |  |  | Container ||  |  |
|  |  +-----------------------+  |    |  |  +-----------+|  |  |
|  |  |  Controller Manager    | |    |  +---------------+  |  |
|  |  +-----------------------+  |    |  |   Pod         |  |  |
|  |  |  etcd (Key-Value Store)| |    |  |  +-----------+|  |  |
|  |  +-----------------------+  |    |  |  | Container ||  |  |
|  +-----------------------------+    |  |  +-----------+|  |  |
|                                     |  +---------------+  |  |
|                                     |                     |  |
|                                     +---------------------+  |
|                                                              |
|                                     +---------------------+  |
|                                     |                     |  |
|                                     |   Worker Node 2     |  |
|                                     |                     |  |
|                                     |  +---------------+  |  |
|                                     |  |   Pod         |  |  |
|                                     |  |  +-----------+|  |  |
|                                     |  |  | Container ||  |  |
|                                     |  |  +-----------+|  |  |
|                                     |  +---------------+  |  |
|                                     |                     |  |
|                                     +---------------------+  |
|                                                              |
+--------------------------------------------------------------+
```

## Components

### Master Node Components

1. **API Server**:
   - **Function**: The API server acts as the front-end for the Kubernetes control plane. It exposes the Kubernetes API and is responsible for handling RESTful requests (e.g., to create, update, delete Kubernetes objects).
   - **Benefit**: Facilitates communication between various components of the Kubernetes system and with external clients.

2. **Scheduler**:
   - **Function**: The scheduler assigns newly created pods to nodes in the cluster based on resource availability and other constraints.
   - **Benefit**: Ensures efficient distribution of workloads across the nodes.

3. **Controller Manager**:
   - **Function**: The controller manager runs controllers, which are background threads responsible for regulating the state of the cluster (e.g., node controllers, replication controllers).
   - **Benefit**: Maintains the desired state of the cluster by managing various controllers that handle different aspects of the system.

4. **etcd (Key-Value Store)**:
   - **Function**: etcd is a distributed key-value store that Kubernetes uses to store all cluster data, including configuration and state information.
   - **Benefit**: Provides consistent and highly available data storage for the cluster.

### Worker Node Components

1. **Kubelet**:
   - **Function**: The kubelet is an agent that runs on each worker node and ensures that containers are running in pods as expected.
   - **Benefit**: Communicates with the API server to manage the lifecycle of pods and report node status.

2. **Kube-Proxy**:
   - **Function**: kube-proxy maintains network rules on nodes, enabling network communication to and from pods.
   - **Benefit**: Facilitates networking in Kubernetes, providing load balancing and forwarding services for application traffic.

3. **Container Runtime**:
   - **Function**: The container runtime (e.g., Docker, containerd) is responsible for pulling container images from a registry, unpacking them, and running the containers.
   - **Benefit**: Provides the necessary environment to run containers.

### Pod

- **Definition**: A pod is the smallest deployable unit in Kubernetes, consisting of one or more containers that share storage, network, and a specification for how to run the containers.
- **Benefit**: Provides a higher-level abstraction for managing containerized applications, allowing for easy deployment and scaling.

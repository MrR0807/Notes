# 2: Kubernetes principles of operation

## Masters (control plane)

![Kubernetes-Master.PNG](pictures/Kubernetes-Master.PNG)

* **API server** - All communication, between all components, must go through the API server. It exposes a RESTful API that you POST YAML configuration files to over HTTPS.
* **Cluster store** - The only stateful part of the control plane, and it persistently stores the entire configuration and state of the cluster. The cluster store is currently based on **etcd**, a popular distributed database. **On the topic of availability, etcd prefers consistency over availability.**
* **Controller manager** - Implements all of the background control loops that monitor the cluster and respond to events. Some of the control loops include; the node controller, the endpoints controller, and the replicaset controller. Each control loop is also extremely specialized and only interested in its own little corner of the Kubernetes cluster.
* **Scheduler** - Watches the API server for new work tasks and assigns them to appropriate healthy nodes. Behind the scenes, it implements complex logic that filters out nodes incapable of running the task, and then ranks the nodes that are capable.

## Nodes

![Kubernetes-Node.PNG](pictures/Kubernetes-Node.PNG)

* **Kubelet** - It’s the main Kubernetes agent, and it runs on every node in the cluster. When you join a new node to a cluster, the process installs kubelet onto the node. One of the main jobs of the kubelet is to watch the API server for new work assignments.
* **Container runtime** - The Kubelet needs a **container runtime** to perform container-related tasks – things like pulling images and starting and stopping containers.
* **Kube-proxy** - Runs on every node in the cluster and is responsible for local cluster networking. For example, it makes sure each node gets its own unique IP address, and implements local IPTABLES or IPVS rules to handle routing and load-balancing of traffic on the Pod network.

## Kubernetes DNS

Every Kubernetes cluster has an internal DNS service that is vital to operations. The cluster’s DNS service has a static IP address that is **hard-coded into every Pod on the cluster**, meaning all containers and Pods know how to find it. Every new service is automatically registered with the cluster’s DNS.

## Pods

Term **Pod** comes from a *pod of whales*. As the Docker logo is a whale, it makes sense that we call a group of containers a Pod. **Kubernetes Pod is a construct for running one or more containers.** If you’re running multiple containers in a Pod, **they all share the same Pod environment.** This includes things like the IPC namespace, shared memory, volumes, network stack and more.

**Pod is only considered ready for service when all of its containers are up and running.**

# 4: Working with Pods









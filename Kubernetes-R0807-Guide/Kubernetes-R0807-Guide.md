**Pause container**

the pause container serves as the "parent container" for all of the containers in your pod. The pause container has two core responsibilities. First, it serves as the basis of Linux namespace sharing in the pod. And second, with PID (process ID) namespace sharing enabled, it serves as PID 1 for each pod and reaps zombie processes.


https://kubernetes.io/docs/reference/kubectl/overview/#resource-types

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#container-v1-core


# Kubernetes Objects

* **Namespaces** provide isolation and access control, so that each microservice can control the degree to which other services interact with it.
* **Labels** are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. They can be arbitrary, and are useful for attaching identifying information to Kubernetes objects. Labels provide the foundation for grouping objects.
* **Annotations** provide a storage mechanism that resembles labels: annotations are key/value pairs designed to hold nonidentifying information that can be leveraged by tools and libraries.
* **Pods**, or groups of containers, can group together container images developed by different teams into a single deployable unit.
* **ReplicaSet** acts as a cluster-wide Pod manager, ensuring that the right types and number of Pods are running all times.
* **Deployment** is a wrapper around ReplicaSet, which simplyfies the rollout of new versions of ReplicaSets.
* **DaemonSets** ensures a copy of a Pod is running across a set of nodes in a Kubernetes cluster. DaemonSets are used to deploy system daemons such as log collectors and monitoring agents, which typically must run on every node. By default a DaemonSet will create a copy of a Pod on every node unless a node selector is used, which will limit eligible nodes to those with a matching set of labels.
* **Jobs** are short-lived, one-off tasks. A job creates Pods that run until successful termination (i.e., exit with 0).
* **ConfigMap** is as a Kubernetes object that defines a small filesystem. Or is as a set of variables that can be used when defining the environment or command line for your containers.
* **Secrets** enable container images to be created without bundling sensitive data. This allows containers to remain portable across environments.
* Kubernetes **services** provide load balancing, naming, and discovery to isolate one microservice from another.
* **Ingress** objects provide an easy-to-use frontend that can combine multiple microservices into a single externalized API surface area.

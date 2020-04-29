**Pause container**

the pause container serves as the "parent container" for all of the containers in your pod. The pause container has two core responsibilities. First, it serves as the basis of Linux namespace sharing in the pod. And second, with PID (process ID) namespace sharing enabled, it serves as PID 1 for each pod and reaps zombie processes.


https://kubernetes.io/docs/reference/kubectl/overview/#resource-types

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#container-v1-core


Kubernetes Objects:
* **Pods**, or groups of containers, can group together container images developed by different teams into a single deployable unit.
* Kubernetes **services** provide load balancing, naming, and discovery to isolate one microservice from another.
* **Namespaces** provide isolation and access control, so that each microservice can control the degree to which other services interact with it.
* **Ingress** objects provide an easy-to-use frontend that can combine multiple microservices into a single externalized API surface area.

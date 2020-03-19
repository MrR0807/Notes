# Chapter 1. Introduction

## Scaling Development Teams with Microservices

Kubernetes provides numerous abstractions and APIs that make it easier to build these decoupled microservice architectures:
* **Pods**, or groups of containers, can group together container images developed by different teams into a single deployable unit.
* Kubernetes **services** provide load balancing, naming, and discovery to isolate one microservice from another.
* **Namespaces** provide isolation and access control, so that each microservice can control the degree to which other services interact with it.
* **Ingress** objects provide an easy-to-use frontend that can combine multiple microservices into a single externalized API surface area.


# Chapter 1. Introduction

## Scaling Development Teams with Microservices

Kubernetes provides numerous abstractions and APIs that make it easier to build these decoupled microservice architectures:
* **Pods**, or groups of containers, can group together container images developed by different teams into a single deployable unit.
* Kubernetes **services** provide load balancing, naming, and discovery to isolate one microservice from another.
* **Namespaces** provide isolation and access control, so that each microservice can control the degree to which other services interact with it.
* **Ingress** objects provide an easy-to-use frontend that can combine multiple microservices into a single externalized API surface area.

# Chapter 2. Creating and Running Containers

Container images bundle a program and its dependencies into a single artifact under a root filesystem. The most popular container image format is the Docker image format, which has been standardized by the Open Container Initiative to the OCI image format.

## Container Images

Container images are constructed with a series of filesystem layers, where each layer inherits and modifies the layers that came before it.
```
.
└── container A: a base operating system only, such as Debian
    └── container B: build upon #A, by adding Ruby v2.1.10
    └── container C: build upon #A, by adding Golang v1.6
```

Containers fall into two main categories:
* System containers
* Application containers

System containers seek to mimic virtual machines and often run a full boot process. They often include a set of system services typically found in a VM, such as ssh, cron, and syslog. **Over time, they have come to be seen as poor practice and application containers have gained favor.**

Application containers differ from system containers in that they commonly run a single program.

## Building Application Images with Docker

### Dockerfiles

To package this up as a Docker image we need to create two additional files: **.dockerignore (Example 2-3) and the Dockerfile (Example 2-4).** The Dockerfile is a recipe for how to build the container image, while **.dockerignore defines the set of files that should be ignored when copying files into the image.**

Example 2-3. .dockerignore
```
node_modules
```

Example 2-4. Dockerfile

```
# Start from a Node.js 10 (LTS) image 
FROM node:10
# Specify the directory inside the image in which all commands will run 
WORKDIR /usr/src/app
# Copy package files and install dependencies 
COPY package*.json ./
RUN npm install
# Copy all of the app files into the image 
COPY . .
# The default command to run when starting the container 
CMD [ "npm", "start" ]
```

## Storing Images in a Remote Registry

The standard within the Docker community is to store Docker images in a remote registry. To push an image, you need to authenticate to the registry. You can generally do this with the ``docker login`` though there are some differences for certain registries. 

## The Docker Container Runtime

### Running Containers with Docker

To deploy a container from the gcr.io/kuar-demo/kuard-amd64:blue image, run the following command:
```
$ docker run -d --name kuard \
  --publish 8080:8080 \
    gcr.io/kuar-demo/kuard-amd64:blue
```

### Limiting Resource Usage

#### LIMITING MEMORY RESOURCES

One of the key benefits to running applications within a container is the ability to restrict resource utilization. To limit kuard to 200 MB of memory and 1 GB of ``swap space``, use the ``--memory`` and ``--memory-swap`` flags with the docker run command.

**Swap memory**

Linux divides its physical RAM (random access memory) into chucks of memory called pages. Swapping is the process whereby a page of memory is copied to the preconfigured space on the hard disk, called swap space, to free up that page of memory. The combined sizes of the physical memory and the swap space is the amount of virtual memory available.

```
$ docker run -d --name kuard \
  --publish 8080:8080 \
  --memory 200m \
  --memory-swap 1G \
  gcr.io/kuar-demo/kuard-amd64:blue
```

#### LIMITING CPU RESOURCES

Another critical resource on a machine is the CPU. Restrict CPU utilization using the --cpu-shares flag with the docker run command:
```
$ docker run -d --name kuard \
  --publish 8080:8080 \
  --memory 200m \
  --memory-swap 1G \
  --cpu-shares 1024 \
  gcr.io/kuar-demo/kuard-amd64:blue
```

## Cleanup

Once you are done building an image, you can delete it with the docker rmi command:
```
docker rmi <tag-name>
```
or
```
docker rmi <image-id>
```

Images can either be deleted via their tag name (e.g., gcr.io/kuar-demo/kuard-amd64:blue) or via their image ID.

# Chapter 3. Deploying a Kubernetes Cluster

### Checking Cluster Status

```
$ kubectl get componentstatuses
```
The output should look like this:
```
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

### Listing Kubernetes Worker Nodes

List out all of the nodes in your cluster:
```
$ kubectl get nodes
NAME         STATUS         AGE       VERSION
kubernetes   Ready,master   45d       v1.12.1
node-1       Ready          45d       v1.12.1
node-2       Ready          45d       v1.12.1
node-3       Ready          45d       v1.12.1
```

# Chapter 4. Common kubectl Commands

## Namespaces

Kubernetes uses namespaces to organize objects in the cluster. You can think of each namespace as a folder that holds a set of objects. **By default, the kubectl commandline tool interacts with the default namespace.**
If you want to use a different namespace, you can pass ``kubectl`` the ``--namespace`` flag. For example, ``kubectl --namespace=mystuff`` references objects in the mystuff namespace. 
If you want to interact with all namespaces - for example, to list all Pods in your cluster you can pass the ``--all-namespaces`` flag.

## Contexts

If you want to change the default namespace more permanently, you can use a context. This gets recorded in a kubectl configuration file, usually located at ``$HOME/.kube/config``. For example, you can create a context with a different default namespace for your kubectl commands using:
```
$ kubectl config set-context my-context --namespace=mystuff
```

This creates a new context, but it doesn’t actually start using it yet. To use this newly created context, you can run:
```
$ kubectl config use-context my-context
```

## Viewing Kubernetes API Objects

``kubectl get <resource-name>`` get a list of all resources in the current namespace.
``kubectl get <resource-name> <obj-name>`` get a specific resource.

``-o wide`` flag gives more details on a longer line.
``-o json`` or ``-o yaml`` flag gives view of complete object.

``kubectl get pods my-pod -o jsonpath --template={.status.podIP}`` uses JSONPath to extract specific fields.

If you are interested in more detailed information about a particular object, use the describe command:
```
$ kubectl describe <resource-name> <obj-name>
```

## Creating, Updating, and Destroying Kubernetes Objects

Let’s assume that you have a simple object stored in obj.yaml. You can use kubectl to create this object in Kubernetes by running:
```
$ kubectl apply -f obj.yaml
```
Similarly, after you make changes to the object, **you can use the apply command again to update the object**:
```
$ kubectl apply -f obj.yaml
```

If you want to see what the ``apply`` command will do without actually making the changes, you can use the **``--dry-run`` flag to print the objects to the terminal without actually sending them to the server.**

When you want to delete an object, you can simply run:
```
$ kubectl delete -f obj.yaml
```

## Labeling and Annotating Objects

Labels and annotations are tags for your objects. To add the color=red label to a Pod named bar, you can run:
```
$ kubectl label pods bar color=red
```

If you want to remove a label, you can use the ``<label-name>-`` syntax:
```
$ kubectl label pods bar color-
```




















































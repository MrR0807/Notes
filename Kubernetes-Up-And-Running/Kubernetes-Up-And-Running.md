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

## Debugging Commands

You can use the following to see the **logs** for a running container:
```
$ kubectl logs <pod-name>
```

If you have multiple containers in your Pod, you can choose the container to view using the ``-c`` flag. 

By default, kubectl logs lists the current logs and exits. If you instead want to continuously stream the logs back to the terminal without exiting, you can add the ``-f (follow)`` command-line flag.

You can also use the **exec** command to execute a command in a running container:
```
$ kubectl exec -it <pod-name> -- bash
```
This will provide you with an interactive shell inside the running container so that you can perform more debugging.

If you don’t have bash or some other terminal available within your container, you can always **attach** to the running process:
```
$ kubectl attach -it <pod-name>
```

**Difference**

exec: any one you want to create, for example bash
attach: the one currently running (no choice)

You can also **copy** files to and from a container using the cp command:
```
$ kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>
```

If you want to access your Pod via the network, you can use the ``port-forward`` command to forward network traffic from the local machine to the Pod. For example, the following command:
```
$ kubectl port-forward <pod-name> 8080:80
```

opens up a connection that forwards traffic from the local machine on port 8080 to the remote container on port 80.

you can use the ``top`` command to see the list of resources in use by either nodes or Pods. This command:
```
$ kubectl top nodes
$ kubectl top pods
```

**Help** command:
```
$ kubectl help
$ kubectl help <command-name>
```

# Chapter 5. Pods

**Sidecars** - containers that co-exists with "main" containers in pods. For example, web application in pod could be a "main" container, while Git synchronizer might be a sidecar.

### Creating a Pod

```
$ kubectl run kuard --generator=run-pod/v1 --image=gcr.io/kuar-demo/kuard-amd64:blue
```

You can see the status of this Pod by running:
```
$ kubectl get pods
```

```
$ kubectl delete pods/kuard
```

## Running Pods

Use the kubectl apply command to launch a single instance of kuard:
```
$ kubectl apply -f kuard-pod.yaml
```

### Pod Details

To find out more information about a Pod:
```
$ kubectl describe pods kuard
```

### Deleting a Pod

When it is time to delete a Pod, you can delete it either by name:
```
$ kubectl delete pods/kuard
```
or using the same file that you used to create it:
```
$ kubectl delete -f kuard-pod.yaml
```

## Accessing Your Pod

### Using Port Forwarding

To access Pod, you can use the port-forwarding support built into the Kubernetes API and command-line tools. When you run:
```
$ kubectl port-forward kuard 8080:8080
```
a secure tunnel is created from your local machine, through the Kubernetes master, to the instance of the Pod running on one of the worker nodes. As long as the port-forward command is still running, you can access the Pod (in this case the kuard web interface) at http://localhost:8080.

### Getting More Info with Logs

The kubectl logs command downloads the current logs from the running instance:
```
$ kubectl logs kuard
```

Adding the ``-f`` flag will cause you to continuously stream logs.

The kubectl logs command always tries to get logs from the **currently running container.** Adding the ``--previous`` flag will get logs **from a previous instance of the container.** This is useful, for example, if your containers are continuously restarting due to a problem at container startup.

### Running Commands in Your Container with exec

Get an interactive shell.

```
$ kubectl exec kuard -it bash
```

### Copying Files to and from Containers

Suppose you had a file called /captures/capture3.txt inside a container in your Pod. You could securely copy that file to your local machine by running:
```
$ kubectl cp <pod-name>:/captures/capture3.txt ./capture3.txt
```
Other times you may need to copy files from your local machine into a container. Let’s say you want to copy $HOME/config.txt to a remote container. In this case, you can run:
```
$ kubectl cp $HOME/config.txt <pod-name>:/config.txt
```
Generally speaking, copying files into a container is an anti-pattern.

## Health Checks

### Liveness Probe

Liveness health checks run application-specific logic (e.g., loading a web page) to verify that the application is not just still running, but is functioning properly. Since these liveness health checks are application-specific, you have to define them in your Pod manifest.

**Liveness probes are defined per container, which means each container inside a Pod is health-checked separately.**

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    livenessProbe:
      httpGet:
        path: /healthy
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```


The preceding Pod manifest uses an httpGet probe to perform an HTTP GET request against the /healthy endpoint on port 8080 of the kuard container. The probe sets an initialDelaySeconds of 5, and thus will not be called until 5 seconds after all the containers in the Pod are created. The probe must respond within the 1-second timeout, and the **HTTP status code must be equal to or greater than 200 and less than 400 to be considered successful.** Kubernetes will call the probe every 10 seconds. If more than three consecutive probes fail, the container will fail and restart.

### Restart Policy

A PodSpec has a restartPolicy field with possible values Always, OnFailure, and Never. The default value is Always. restartPolicy applies to all Containers in the Pod.

### Readiness Probe

Liveness determines if an application is running properly. Containers that fail liveness checks are restarted. Readiness describes when a container is ready to serve user requests. **Containers that fail readiness checks are removed from service load balancers.**

### Types of Health Checks

Kubernetes also supports tcpSocket health checks that open a TCP socket; if the connection is successful, the probe succeeds. This style of probe is useful for non-HTTP applications; for example, databases or other non–HTTP-based APIs.

## Resource Management

Kubernetes allows users to specify two different resource metrics:
* **Requests** - specify the minimum amount of a resource required to run the application. 
* **Limits** - specify the maximum amount of a resource that an application can consume.

### Resource Requests: Minimum Required Resources

**The most commonly requested resources are CPU and memory**, but Kubernetes has support for other resource types as well, such as GPUs and more.

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

**Resources are requested per container, not per Pod. The total resources requested by the Pod is the sum of all resources requested by all containers in the Pod.**

### Capping Resource Usage with Limits

In addition to setting the resources required by a Pod, which establishes the minimum resources available to the Pod, you can also set a maximum on a Pod’s resource usage via resource limits.

```
apiVersion: v1
kind: Pod
  metadata:
    name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
      limits:
        cpu: "1000m"
        memory: "256Mi"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP

```

## Persisting Data with Volumes

### Using Volumes with Pods

To add a volume to a Pod manifest, there are two new stanzas to add to our configuration. The first is a new ``spec.volumes`` section. This array defines all of the volumes that may be accessed by containers in the Pod manifest. It’s important to note that not all containers are required to mount all volumes defined in the Pod. The second addition is the ``volumeMounts`` array in the container definition. This array defines the volumes that are mounted into a particular container, and the path where each volume should be mounted. **Note that two different containers in a Pod can mount the same volume at different mount paths.**

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes:
  - name: "kuard-data"
    hostPath:
      path: "/var/lib/kuard"
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    volumeMounts:
    - mountPath: "/data"
      name: "kuard-data"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

### Different Ways of Using Volumes with Pods

* **Communication/Synchronization**. In the first example of a Pod, we saw how two containers used a shared volume to serve a site while keeping it synchronized to a remote Git location.
* **Cache**. An application may use a volume that is valuable for performance, but not required for correct operation of the application. For example, perhaps the application keeps prerendered thumbnails of larger images. Of course, they can be reconstructed from the original images, but that makes serving the thumbnails more expensive.
* **Persistent Data**. Kubernetes supports a wide variety of remote network storage volumes, including widely supported protocols like NFS and iSCSI as well as cloud provider network storage like Amazon’s Elastic Block Store, Azure’s Files and Disk Storage, as well as Google’s Persistent Disk.

## Putting It All Together

```
apiVersion: v1
kind: Pod
metadata:
name: kuard
spec:
  volumes:
  - name: "kuard-data"
    nfs:
      server: my.nfs.server.local
      path: "/exports"
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
      limits:
        cpu: "1000m"
        memory: "256Mi"
    volumeMounts:
    - mountPath: "/data"
      name: "kuard-data"
    livenessProbe:
      httpGet:
        path: /healthy
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
    initialDelaySeconds: 30
    timeoutSeconds: 1
    periodSeconds: 10
    failureThreshold: 3
```

# Chapter 6. Labels and Annotations

**Labels** are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. They can be arbitrary, and are useful for attaching identifying information to Kubernetes objects. Labels provide the foundation for grouping objects.

**Annotations**, on the other hand, provide a storage mechanism that resembles labels: annotations are key/value pairs designed to hold nonidentifying information that can be leveraged by tools and libraries.

## Labels

Label keys can be broken down into two parts: an optional prefix and a name, separated by a slash. The prefix, if specified, must be a DNS subdomain with a 253-character limit. **The key name is required and must be shorter than 63 characters. Names must also start and end with an alphanumeric character and permit the use of dashes (-), underscores (\_), and dots (.) between characters.**

**Label values are strings with a maximum length of 63 characters.** The contents of the label values follow the same rules as for label keys.

### Applying Labels

We’ll take two apps (called alpaca and bandicoot) and have two environments for each. We will also have two different versions.


First, create the alpaca-prod deployment and set the ver, app, and env labels:
```
$ kubectl run alpaca-prod \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--replicas=2 \
--labels="ver=1,app=alpaca,env=prod"
```

Next, create the alpaca-test deployment and set the ver, app, and env labels with the appropriate values:

```
$ kubectl run alpaca-test \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=1 \
--labels="ver=2,app=alpaca,env=test"
```
Finally, create two deployments for bandicoot. Here we name the environments prod and staging:
```
$ kubectl run bandicoot-prod \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=2 \
--labels="ver=2,app=bandicoot,env=prod"

$ kubectl run bandicoot-staging \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=1 \
--labels="ver=2,app=bandicoot,env=staging"
```

At this point you should have four deployments—alpaca-prod, alpaca-test, bandicoot-prod, and bandicoot-staging:

### Modifying Labels

Labels can also be applied (or updated) on objects after they are created:
```
$ kubectl label deployments alpaca-test "canary=true"
```

You can also use the -L option to kubectl get to show a label value as a column:
```
$ kubectl get deployments -L canary
```

You can remove a label by applying a dash suffix:
```
$ kubectl label deployments alpaca-test "canary-"
```

### Label Selectors

Label selectors are used to filter Kubernetes objects based on a set of labels.

```
$ kubectl get pods --show-labels

NAME ... LABELS
alpaca-prod-3408831585-4nzfb ... app=alpaca,env=prod,ver=1,...
alpaca-prod-3408831585-kga0a ... app=alpaca,env=prod,ver=1,...
alpaca-test-1004512375-3r1m5 ... app=alpaca,env=test,ver=2,...
bandicoot-prod-373860099-0t1gp ... app=bandicoot,env=prod,ver=2,...
bandicoot-prod-373860099-k2wcf ... app=bandicoot,env=prod,ver=2,...
bandicoot-staging-1839769971-3ndv ... app=bandicoot,env=staging,ver=2,...
```

You may see a new label that we haven’t seen yet: ``pod-template-hash``. This label is applied by the deployment so it can keep track of which Pods were generated from which template versions.

If we only wanted to list Pods that had the ver label set to 2, we could use the ``--selector`` flag:
```
$ kubectl get pods --selector="ver=2"
```

If we specify two selectors separated by a comma, only the objects that satisfy both will be returned. This is a logical AND operation:
```
$ kubectl get pods --selector="app=bandicoot,ver=2"
```

We can also ask if a label is one of a set of values. Here we ask for all Pods where the app label is set to alpaca or bandicoot (which will be all six Pods):
```
$ kubectl get pods --selector="app in (alpaca,bandicoot)"
```

Finally, we can ask if a label is set at all. Here we are asking for all of the deployments with the canary label set to anything:
```
$ kubectl get deployments --selector="canary"
```

For example, asking if a key, in this case canary, is not set can look like:
```
$ kubectl get deployments --selector='!canary'
```
Similarly, you can combine positive and negative selectors together as follows:
```
$ kubectl get pods -l 'ver=2,!canary'
```

| Operator  | Description |
| ------------- | ------------- |
| key=value | key is set to value |
| key!=value | key is not set to value |
| key in (value1, value2) | key is one of value1 or value2 |
| key notin (value1, value2) | key is not one of value1 or value2 |
| key | key is set |
| !key | key is not set |

### Label Selectors in API Objects

When a Kubernetes object refers to a set of other Kubernetes objects, a label selector is used.

There are two forms.

Older form of specifying selectors (used in ReplicationControllers and services) only supports the = operator. This is a simple set of key/value pairs that must all match a target object to be selected. The selector app=alpaca,ver=1 would be represented like this:
```
selector:
  app: alpaca
  ver: 1
```

Newer form of specifying a selector of ``app=alpaca,ver in (1, 2)`` would be converted to this:
```
selector:
  matchLabels:
    app: alpaca
  matchExpressions:
  - {key: ver, operator: In, values: [1, 2]}
```

### Labels in the Kubernetes Architecture

In addition to enabling users to organize their infrastructure, labels play a critical role in linking various related Kubernetes objects. In many cases objects need to relate to one another, and these relationships are defined by labels and label selectors.

## Annotations

**Annotations provide a place to store additional metadata for Kubernetes objects with the sole purpose of assisting tools and libraries.**

While labels are used to identify and group objects, annotations are used to provide extra information about where an object came from, how to use it, or policy around that object.

There is overlap, and it is a matter of taste as to when to use an annotation or a label. When in doubt, add information to an object as an annotation and promote it to a label if you find yourself wanting to use it in a selector.

Annotations are used to:
* Keep track of a "reason" for the latest update to an object.
* Communicate a specialized scheduling policy to a specialized scheduler.
* Extend data about the last tool to update the resource and how it was updated (used for detecting changes by other tools and doing a smart merge).
* Attach build, release, or image information that isn’t appropriate for labels (may include a Git hash, timestamp, PR number, etc.).
* Enable the Deployment object to keep track of ReplicaSets that it is managing for rollouts.
* Provide extra data to enhance the visual quality or usability of a UI. For example, objects could include a link to an icon (or a base64-encoded version of an icon).
* Prototype alpha functionality in Kubernetes (instead of creating a first-class API field, the parameters for that functionality are encoded in an annotation).

Annotations are used in various places in Kubernetes, with the primary use case being rolling deployments. During rolling deployments, annotations are used to track rollout status and provide the necessary information required to roll back a deployment to a previous state.

### Defining Annotations

Annotation keys use the same format as label keys. However, because they are often used to communicate information between tools, the "namespace" part of the key is more important. Example keys include ``deployment.kubernetes.io/revision`` or ``kubernetes.io/changecause``.

Annotations are defined in the common metadata section in every Kubernetes object:
```
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```

# Chapter 7. Service Discovery

While the dynamic nature of Kubernetes makes it easy to run a lot of things, it creates problems when it comes to **finding** those things.

## What Is Service Discovery?

Service-discovery tools help solve the problem of finding which processes are listening at which addresses for which services.

The Domain Name System (DNS) is the traditional system of service discovery on the internet. It is a great system for the internet but falls short in the dynamic world of Kubernetes. 

Unfortunately, many systems (for example, Java, by default) look up a name in DNS directly and never re-resolve. This can lead to clients caching stale mappings and talking to the wrong IP. Even with short TTLs and well-behaved clients, there is a natural delay between when a name resolution changes and when the client notices.

## The Service Object

Let’s create some deployments and services so we can see how they work:
```
$ kubectl run alpaca-prod \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--replicas=3 \
--port=8080 \
--labels="ver=1,app=alpaca,env=prod"
$ kubectl expose deployment alpaca-prod
$ kubectl run bandicoot-prod \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=2 \
--port=8080 \
--labels="ver=2,app=bandicoot,env=prod"
$ kubectl expose deployment bandicoot-prod
```
```
$ kubectl get services -o wide
NAME CLUSTER-IP ... PORT(S) ... SELECTOR
alpaca-prod 10.115.245.13 ... 8080/TCP ... app=alpaca,env=prod,ver=1
bandicoot-prod 10.115.242.3 ... 8080/TCP ...
app=bandicoot,env=prod,ver=2
kubernetes 10.115.240.1 ... 443/TCP ... <none>
```

The kubernetes service is automatically created for you so that you can find and talk to the Kubernetes API from within the app.

Service is assigned a new type of **virtual IP** called a **cluster IP**.

To interact with services, we are going to port forward to one of the alpaca Pods. Start and leave this command running in a terminal window. You can see the port forward working by accessing the alpaca Pod at http://localhost:48858:
```
$ ALPACA_POD=$(kubectl get pods -l app=alpaca \
-o jsonpath='{.items[0].metadata.name}')
$ kubectl port-forward $ALPACA_POD 48858:8080
```

### Service DNS

Kubernetes DNS service provides DNS names for cluster IPs.

### Readiness Checks

Often, when an application first starts up it isn’t ready to handle requests. There is usually some amount of initialization that can take anywhere from under a second to several minutes. One nice thing the Service object does is track which of your Pods are ready via a readiness check.

## Cloud Integration

If you have support from the cloud that you are running on (and your cluster is configured to take advantage of it), you can use the **LoadBalancer type.** This builds on the NodePort type by additionally configuring the cloud to create a new load balancer and direct it at nodes in your cluster.

If you do a kubectl get services right away you’ll see that the EXTERNALIP column for alpaca-prod now says <pending>. Wait a bit and you should see a public address assigned by your cloud.
```
$ kubectl describe service alpaca-prod
Name: alpaca-prod
Namespace: default
Labels: app=alpaca
env=prod
ver=1
Selector: app=alpaca,env=prod,ver=1
Type: LoadBalancer
IP: 10.115.245.13
LoadBalancer Ingress: 104.196.248.204
Port: <unset> 8080/TCP
NodePort: <unset> 32711/TCP
Endpoints:
10.112.1.66:8080,10.112.2.104:8080,10.112.2.105:8080
Session Affinity: None
Events:
FirstSeen ... Reason Message
--------- ... ------ -------
3m ... Type NodePort -> LoadBalancer
3m ... CreatingLoadBalancer Creating load balancer
2m ... CreatedLoadBalancer Created load balancer
```

Here we see that we have an address of 104.196.248.204 now assigned to the alpaca-prod service. Open up your browser and try!

## Advanced Details

### Endpoints

Some applications (and the system itself) want to be able to use services without using a cluster IP. This is done with another type of object called an Endpoints object. For every Service object, Kubernetes creates a buddy Endpoints object that contains the IP addresses for that service:
```
$ kubectl describe endpoints alpaca-prod
Name: alpaca-prod
Namespace: default
Labels: app=alpaca
env=prod
ver=1
Subsets:
Addresses: 10.112.1.54,10.112.2.84,10.112.2.85
NotReadyAddresses: <none>
Ports:
Name Port Protocol
---- ---- --------
<unset> 8080 TCP
```

In a terminal window, start the following command and leave it running:
```
$ kubectl get endpoints alpaca-prod --watch
```
It will output the current state of the endpoint and then "hang":
```
NAME ENDPOINTS AGE
alpaca-prod 10.112.1.54:8080,10.112.2.84:8080,10.112.2.85:8080 1m
```
Now open up another terminal window and delete and recreate the deployment backing alpaca-prod:
```
$ kubectl delete deployment alpaca-prod
$ kubectl run alpaca-prod \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--replicas=3 \
--port=8080 \
--labels="ver=1,app=alpaca,env=prod"
```

Your output will look something like this:
```
NAME ENDPOINTS AGE
alpaca-prod 10.112.1.54:8080,10.112.2.84:8080,10.112.2.85:8080 1m
alpaca-prod 10.112.1.54:8080,10.112.2.84:8080 1m
alpaca-prod <none> 1m
alpaca-prod 10.112.2.90:8080 1m
alpaca-prod 10.112.1.57:8080,10.112.2.90:8080 1m
alpaca-prod 10.112.0.28:8080,10.112.1.57:8080,10.112.2.90:8080 1m
```

### Manual Service Discovery

































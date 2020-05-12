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
* **StatefulSets** manages the deployment and scaling of a set of Pods , and provides guarantees about the **ordering and uniqueness of these Pods**.
* **Persistent Volume (PV)** is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.
* **Persistent Volume Claim (PVC)** is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted once read/write or many times read-only).
* **Storage Classes (SC)** enables for dynamic volume provisioning.
* **ConfigMap** is as a Kubernetes object that defines a small filesystem. Or is as a set of variables that can be used when defining the environment or command line for your containers.
* **Secrets** enable container images to be created without bundling sensitive data. This allows containers to remain portable across environments.
* Kubernetes **services** provide load balancing, naming, and discovery to isolate one microservice from another.
* **Ingress** objects provide an easy-to-use frontend that can combine multiple microservices into a single externalized API surface area.

## Namespaces

```
kubectl get namespace
```

To set the namespace for a current request, use the --namespace flag.
```
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
kubectl get pods -n=<insert-namespace-name-here>
```

Setting the namespace preference:
```
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
kubectl config view --minify | grep namespace:
```

Creating a new namespace
```
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>
  
kubectl create -f ./my-namespace.yaml
```

Alternatively:
```
kubectl create namespace <insert-namespace-name-here>
```

Deleting a namespace

```
kubectl delete namespaces <insert-some-namespace-name>
```

To list all Pods in your cluster you can pass the ``--all-namespaces`` flag:
```
kubectl get pods --all-namespaces
```

## Labels

To add the color=red label to a Pod named bar, you can run:
```
$ kubectl label pods bar color=red
```

Remove a label:
```
$ kubectl label pods bar color-
```

Create a ReplicaSet with labels:
```
$ kubectl run alpaca-prod \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--replicas=2 \
--labels="ver=1,app=alpaca,env=prod"
```

Labels can also be applied (or updated) on objects after they are created:
```
$ kubectl label deployments alpaca-test "canary=true"
```
You can also use the -L option to ``kubectl get`` to show a label value as a column:
```
$ kubectl get deployments -L canary
```
You can remove a label by applying a dash suffix:
```
$ kubectl label deployments alpaca-test "canary-"
```

Show labels:
```
$ kubectl get pods --show-labels

NAME ... LABELS
alpaca-prod-3408831585-4nzfb ... app=alpaca,env=prod,ver=1,...
```

If we only wanted to list Pods that had the ver label set to 2, we could use the --selector flag:
```
$ kubectl get pods --selector="ver=2"
```

If we specify two selectors separated by a comma, only the objects that satisfy both will be returned. This is a logical AND operation:
```
$ kubectl get pods --selector="app=bandicoot,ver=2"
```

We can also ask if a label is one of a set of values. Here we ask for all Pods where the app label is set to alpaca or bandicoot:
```
$ kubectl get pods --selector="app in (alpaca,bandicoot)"
```

Here we are asking for all of the deployments with the canary label set to anything:
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

## Annotations

Annotations are defined in the common metadata section in every Kubernetes object:

```
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```

## Pods

Create:
```
kubectl run kuard --generator=run-pod/v1 --image=gcr.io/kuar-demo/kuard-amd64:blue
$ kubectl apply -f kuard-pod.yaml
```

Get more information:
```
$ kubectl get pods
$ kubectl describe pods kuard
$ kubectl logs kuard
$ kubectl get pods hello-pod -o yaml
$ kubectl get pods hello-pod -o wide
```

Deleting:
```
$ kubectl delete pods/kuard
$ kubectl delete -f kuard-pod.yaml
```

Running commands:
```
$ kubectl exec kuard -it bash
$ kubectl exec -it shell-demo -- /bin/bash
```

Utility commands:
```
$ kubectl cp <pod-name>:/captures/capture3.txt ./capture3.txt
$ kubectl cp $HOME/config.txt <pod-name>:/config.txt
```

Watch flag:
```
$ kubectl get pods --watch
```

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-name
spec:
  containers:
  - image:
    imagePullPolicy: Always | Never | IfNotPresent
    args: [] #The docker image's CMD is used if this is not provided.
    command: [ "/bin/sh", "-c", "env" ] #Not executed within a shell. The docker image's ENTRYPOINT is used if this is not provided.
    ports:
      - containerPort: 8080
        protocol: UDP | TCP | SCTP #Defaults to TCP
        name: http
        hostPort: #Don’t specify a hostPort for a Pod unless it is absolutely necessary.
    env:
    - name: varvalue
      value: ${VAR_NAME}
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: another
      valueFrom: # Cannot be used if value is not empty.
        configMapKeyRef:
          name: hello
    envFrom:
    - configMapRef:
        name: java-options
    - secretRef:
        name: rabbit-credentials
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
        path: /healthy
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    startupProbe: ## If the startup probe never succeeds, the container is killed after 300s and subject to the pod’s restartPolicy.
      httpGet:
        path: /healthy
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
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
  imagePullSecrets: #Secret used to pull images
  restartPolicy: Always | OnFailure | Never
  volumes: #List of volumes that can be mounted by containers belonging to the pod.
  - name: "kuard-data"
    nfs:
      server: my.nfs.server.local
      path: "/exports"
```

## ReplicaSet

```
apiVersion:
kind:
metadata:
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mylabel
  template:
    metadata:
      labels:
        app: mylabel
    spec: ##podSpec from before
      ...
```

Create ReplicaSet:
```
$ kubectl apply -f kuard-rs.yaml
replicaset "kuard" created
```

Insepect ReplicaSet:
```
$ kubectl describe rs kuard
```

Find a ReplicaSet from a Pod:
```
$ kubectl get pods <pod-name> -o yaml
```

Find a set of Pods for a ReplicaSet. Selector flag ``--selector`` or the shorthand ``-l``:
```
$ kubectl get pods -l app=kuard,version=2
```

Scale ReplicaSet. ``rs`` for short:
```
$ kubectl scale replicasets kuard --replicas=4
```

Deleting ReplicaSet:
```
$ kubectl delete rs kuard
```

Deleting ReplicaSet, but not underlying Pods:
```
$ kubectl delete rs kuard --cascade=false
```

Scale a replicaset named 'foo' to 3
```
kubectl scale --replicas=3 rs/foo
```


## Deployment

```
apiVersion:
kind:
metadata:
spec:
  replicas: 3
  revisionHistoryLimit: #default 10
  minReadySeconds: 60 #indicates that the deployment must wait for 60 seconds after seeing a Pod become healthy before moving on to                            updating the next Pod
  progressDeadlineSeconds: #default 600
  strategy:
    type: #Recreate | RollingUpdate. Defaults to RollingUpdate
    rollingUpdate: #Only if type is RollingUpdate
      maxSurge: 1 #You will never have more than 11 Pods during the update process
      maxUnavailable: 1 #You'll never have less than 9
  selector:
    matchLabels:
        app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec: ##podSpec from before
      ...

```

Create:
```
kubectl create deployment nginx --image=nginx  # Start a single instance of nginx

kubectl get deployment my-dep 
# Or deployment short name deploy
kubectl get deploy my-dep

kubectl scale deployments kuard --replicas=2

kubectl describe deployments kuard

kubectl rollout status deployments kuard # Monitor new deployment rollout

kubectl rollout history deployment kuard # Rollout history

kubectl rollout history deployment kuard --revision=2 # More details about specific revision

kubectl rollout undo deployments kuard # Rollback deployment

kubectl rollout undo deployments kuard --to-revision=3 # Rollback deployment to specific revision

kubectl delete deployments kuard

kubectl delete -f kuard-deployment.yaml
```

## DaemonSets

```
apiVersion:
kind:
metadata:
  labels:
    app: hello
spec:
  minReadySeconds:
  revisionHistoryLimit:
  selector:
    matchLabels:
  updateStrategy:
    type: RollingUpdate | OnDelete # Default is RollingUpdate
    rollingUpdate: # Only if type is RollingUpdate
      maxUnavailable:
  template: #podSpec
    ...
```

DaemonSets require a unique name.
```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  ...
```

```
$ kubectl apply -f fluentd.yaml
daemonset "fluentd" created
```
```
$ kubectl describe daemonset fluentd
```

## Jobs

```
apiVersion:
kind:
metadata:
spec:
  activeDeadlineSeconds: # Once a Job reaches activeDeadlineSeconds, all of its running Pods are terminated
  bacckoffLimit: # Number of retries, before job is failed. Defaults to 6
  completions: # Iterations how many times the pod will be run
  parallelism: # How many pods will run at any given time
  ttlSecondsAfterFinished: # Automatic cleanup of Job objects after it has finished (either Complete or Failed)
  selector:
    matchLabels:
  template: #podSpecs
    ...
```

Create. All parameter after ``--`` are command-line arguments:
```
$ kubectl run -i oneshot \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--restart=OnFailure \ 
-- --keygen-enable \
   --keygen-exit-on-complete \
   --keygen-num-to-gen 10
```

``-a`` 

```
kubectl get pod -a -l job-name=oneshot
```




## StatefulSets
## Persistent Volume
## Persistent Volume Claim
## Storage Classes
## ConfigMap
## Secrets
## Services
## Ingress

## Misc

If you want to see what the apply command will do without actually making the changes, you can use the ``--dry-run`` flag to print the objects to the terminal without actually sending them to the server.




# Content
* [Deep dive into the Prometheus configuration](#deep-dive-into-the-prometheus-configuration)
  * [The storage section](#the-storage-section)
  * [The web section](#the-web-section)
  * [The query section](#the-query-section)
  * [Prometheus configuration file walkthrough](#prometheus-configuration-file-walkthrough)
  * [Global configuration](#global-configuration)

# Deep dive into the Prometheus configuration

## The storage section

Following the same logic from the previous section, the ``--storage.tsdb.path`` flag should be set to configure the base path to the data storage location. This defaults to ``data/`` on the current working directory, and so it is advisable to point this to a more appropriate path — possibly to a different drive/volume, where data can be safely persisted and I/O contention can be mitigated. **To note that NFS (AWS EFS included)**.

**Placing the Prometheus data storage directory in a network share is also ill-advised as transient network failures would impact the monitoring system's ability to keep functioning.**

The Prometheus local storage can only be written to by a single Prometheus instance at a time. To make sure this is the case, it uses a lock file in the data directory.

There can be an edge case to this behavior; when using persistent volumes to store the data directory, there is a chance that, when relaunching Prometheus as another container instance using the same volume, the previous instance might not have unlocked the database. This problem would make a setup of this kind susceptible to race conditions. Luckily, there is the ``--storage.tsdb.no-lockfile`` flag, which can be used in exactly this type of situation. Be warned though that, in general (and namely, in most Prometheus deployments), **it is a bad idea to disable the lock file, as doing so makes unintended data corruption easier.**

## The web section

The next step is to configure what address users are going to utilize to get to the Prometheus server. The ``--web.external-url`` flag sets this base URL so that weblinks generated both in the web user interface and in outgoing alerts link back to the Prometheus server or servers correctly. This might be the DNS name for a load balancer/reverse proxy, a Kubernetes service, or, in the simplest deployments, the publicly accessible, fully qualified domain name of the host running the server.

The ``--web.enable-lifecycle`` flag can be used to enable the ``/-/reload`` and ``/-/quit`` HTTP endpoints, which can be used to control, reload, and shut down, respectively. To prevent accidental triggering of these endpoints, and because a GET wouldn't be semantically correct, a POST request is needed.

Similarly, the ``--web.enable-admin-api`` flag is also turned off by default for the same reason. This flag enables HTTP endpoints that provide some advanced administration actions, such as creating snapshots of data, deleting time series, and cleaning tombstones.

## The query section

Some are fairly straightforward to understand, such as how long a given query can run before being aborted (``--query.timeout``), or how many queries can run simultaneously (``--query.max-concurrency``).

However, two of them set limits that can have non-obvious consequences. The first is ``--query.max-samples``, which was introduced in Prometheus 2.5.0, that sets the maximum number of samples that can be loaded onto memory. This was done as a way of capping the maximum memory the query subsystem can use to try and prevent the dreaded **query-of-death** — a query that loaded so much data to memory that it made Prometheus hit a memory limit and then killing the process. Defaults to 50,000,000 samples.

The second one is --query.lookback-delta. Without going into too much detail regarding how PromQL works internally, this flag sets the limit of how far back Prometheus will look for time series data points before considering them stale. **This implicitly means that if you collect data at a greater interval than what's set here (the default being five minutes), you will get inconsistent results in alerts and graphs, and as such, two minutes is the maximum sane value to allow for failures.**

## Prometheus configuration file walkthrough

Everything related to scrape jobs, rule evaluation, and remote read/write configuration is all defined here. As mentioned previously, these configurations can be reloaded without shutting down the Prometheus server by either sending a ``SIGHUP`` to the process, or by sending an HTTP POST request to the ``/-/reload`` endpoint (when ``--web.enable-lifecycle`` is used at startup).

At a high level, we can split the configuration file into the following sections:
* global
* scrape_configs
* alerting
* rule_files
* remote_read
* remote_write

Example configuration:
```
global:
  scrape_interval: 1m
...
scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 15s
    scrape_timeout: 5s
    sample_limit: 1000
    static_configs:
      - targets: ['localhost:9090']
    metric_relabel_configs:
      - source_labels: [ __name__ ]
        regex: expensive_metric_.+
        action: drop
```

### Global configuration

The **global configuration defines the default parameters for every other configuration section**, as well as outlining what labels should be added to metrics going to external systems, as shown in the following code block:

```
global:
  scrape_interval: 1m
  scrape_timeout: 10s
  evaluation_interval: 1m
  external_labels:
    dc: dc1
    prom: prom1
```

* ``scrape_interval`` sets the default frequency targets that should be scraped. This is **usually between 10 seconds and one minute**, and the **default 1m** is a good conservative value to start. **Longer intervals are not advisable;**
* ``scrape_timeout`` defines how long Prometheus should wait by default for a response from a target before closing the connection and marking the scrape as failed (10 seconds if not declared);
* ``evaluation_interval`` sets the default frequency recording and alerting rules are evaluated. **For sanity, both ``scrape_timeout`` and ``evaluation_interval`` have to be the same;**
* ``external_labels`` sets labels to add to any time series or alerts when communicating with external systems (federation, remote storage, Alertmanager).

## Scrape configuration

Even though Prometheus accepts an empty file as a valid configuration file, the absolute minimum useful configuration needs a ``scrape_configs`` section. This is where we define the targets for metrics collection, and if some post-scrape processing is needed before actual ingestion.

```
scrape_configs:
 - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
...

  - job_name: 'blackbox'
    static_configs:
      - targets:
        - http://example.com
        - https://example.com:443
...
```

In Prometheus terms, a scrape is the action of collecting metrics through an HTTP request from a targeted instance, parsing the response, and ingesting the collected samples to storage. The default HTTP endpoint used in the Prometheus ecosystem for metrics collection is aptly named ``/metrics``.

A collection of such instances is called a **job**.

A scrape job definition needs at least a job_name and a set of targets. In this example, static_configs was used to declare the list of targets for both scrape jobs. While Prometheus supports a lot of ways to dynamically define this list, static_configs is the simplest and most straightforward method:

```
scrape_configs:
 - job_name: 'prometheus'
    scrape_interval: 15s
    scrape_timeout: 5s
    sample_limit: 1000
    static_configs:
      - targets: ['localhost:9090']
    metric_relabel_configs:
      - source_labels: [ __name__ ]
        regex: expensive_metric_.+
        action: drop
```

Analyzing the prometheus scrape job in detail, we can see that both ``scrape_interval`` and ``scrape_timeout`` can be redeclared at the job level, thus overriding the global values.

``sample_limit`` is a scrape config field that will cause a scrape to fail if more than the given number of time series is returned. **``sample_limit`` can save you from exploding in cardinality. ``sample_limit`` is a scrape config field that will cause a scrape to fail if more than the given number of time series is returned.**

The last relevant configuration here is ``metric_relabel_configs``. This is a powerful rewrite engine that allows a collected metrics' identity to be transformed, or even dropped, before being saved to storage. The most common use cases for this feature is to blacklist a set of misbehaving metrics, dropping labels. The preceding example is using ``metric_relabel_configs`` to drop every metric that starts with ``expensive_metric_``.

```
  - job_name: 'blackbox'
    metrics_path: /probe
    scheme: http
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115
```

* ``metrics_path`` is used to change which endpoint Prometheus should scrape
* ``scheme`` defines whether HTTP or HTTPS is going to be used when connecting to targets
* ``params`` allows you to define a set of optional HTTP parameters

**``relabel_configs`` is used to manipulate the scrape job's list of targets.** By default, a target will have a couple of labels that have been generated automatically and that will be available for relabeling:
* ``job`` label will be set to the ``job_name`` configuration;
* ``__address__`` label will be created with the target's host and port;
* ``__scheme__`` and ``__metrics_path__`` labels will be set to their respective configurations (``scheme`` and ``metrics_path``);
* ``__param_<name>`` label will be created for each of the parameters defined in the params configuration.

Example:
* Copies the target's address into a ``__param_target`` label, which will be used to set the ``target`` ``GET`` parameter in the scrape;
* Copies the content of this newly created label into the ``instance`` label so that it is explicitly set, bypassing the automatic generation based on the ``__address__``;
* Replaces the ``__address__`` label with the address of the blackbox exporter so that scrapes are done to the exporter and not directly to the target we specified in ``static_configs``.

# Managing Prometheus in a standalone server

Not interested. Good to know where to look if going to setup it.

# Managing Prometheus in Kubernetes

## Prometheus server deployment

ConfigMap:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring 
data:
  prometheus.yml: |
    scrape_configs:
    - job_name: prometheus
      static_configs:
      - targets:
        - localhost:9090
```

Deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
  labels:
    app: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
      annotations:
        version: "1"
    spec:
      containers:
      - name: prometheus-server
        image: prom/prometheus:v2.9.2
        imagePullPolicy: "IfNotPresent"
        args:
          - --config.file=/etc/config/prometheus.yml
          - --storage.tsdb.path=/data
          - --web.console.libraries=/etc/prometheus/console_libraries
          - --web.console.templates=/etc/prometheus/consoles
          - --web.enable-lifecycle
        volumeMounts:
          - name: config-volume
            mountPath: /etc/config/prometheus.yml
            subPath: prometheus.yml
          - name: prometheus-data
            mountPath: /data
            subPath: ""
        resources:
          limits:
            cpu: 200m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 256Mi
        ports:
        - containerPort: 9090
        readinessProbe:
          httpGet:
            path: /-/ready
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
        livenessProbe:
          httpGet:
            path: /-/healthy
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
      volumes:
        - name: config-volume
          configMap:
            name: prometheus-config
        - name: prometheus-data
          emptyDir: {}
```

```
kind: Service
apiVersion: v1
metadata:
  name: prometheus-service
  namespace: monitoring
spec:
  selector:
    app: prometheus
  type: NodePort
  ports:
  - name: prometheus-port
    protocol: TCP
    port: 9090
    targetPort: 9090
```

Check which nodePort was assign to this service, then you can try to access it.

## Adding targets to Prometheus

For the sake of this example, we'll deploy yet another service and add it to our Prometheus server, going step by step on how to do it. We'll use a small Hello World type of application called ``Hey`` for our setup.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hey-deployment
  namespace: monitoring
  labels:
    app: hey
...
      - name: hey
        image: kintoandar/hey:v1.0.1
...
        - name: http
          containerPort: 8000
...
```

```
kind: Service
apiVersion: v1
metadata:
  name: hey-service
  namespace: monitoring
spec:
  selector:
    app: hey
  type: NodePort
  ports:
  - name: hey-port
    protocol: TCP
    port: 8000
    targetPort: 8000
```

We need to change the Prometheus ConfigMap to reflect the newly added service:
```
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring 
data:
  prometheus.yml: |
    scrape_configs:
    - job_name: prometheus
      static_configs:
      - targets:
        - localhost:9090
    - job_name: hey
      static_configs:
      - targets:
        - hey-service.monitoring.svc:8000
```

After a moment, a new deployment will take place, changing the Prometheus configuration and a new target will present itself, which you can validate in the Prometheus web user interface:

![Prometheus-add-targets-1.png](pictures/Prometheus-add-targets-1.png)


## Dynamic configuration – the Prometheus Operator

The combination of both a Kubernetes custom resource and custom controller into a pattern is what brings the Operator definition to life.

In our case, **besides managing the deployment, including the number of pods and persistent volumes of the Prometheus server, the Prometheus Operator will also update the configuration dynamically using the concept of ServiceMonitor, which targets services with matching rules against the labels of running containers**:

![Prometheus-operator.png](pictures/Prometheus-operator.png)

## Prometheus Operator deployment

Like the previous example, we'll be creating a new namespace called ``monitoring``.

### Prometheus Operator deployment

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus-operator
rules:
- apiGroups: [apiextensions.k8s.io]
  resources: [customresourcedefinitions]
  verbs: ['*']
- apiGroups: [monitoring.coreos.com]
  resources: 
  - alertmanagers
  - prometheuses
  - prometheuses/finalizers
  - alertmanagers/finalizers
  - servicemonitors
  - prometheusrules
  verbs: ['*']
- apiGroups: [apps]
  resources: [statefulsets]
  verbs: ['*']
- apiGroups: [""]
  resources: [configmaps, secrets]
  verbs: ['*']
- apiGroups: [""]
  resources: [pods]
  verbs: [list, delete]
- apiGroups: [""]
  resources: [services, endpoints]
  verbs: [get, create, update]
- apiGroups: [""]
  resources: [nodes]
  verbs: [list, watch]
- apiGroups: [""]
  resources: [namespaces]
  verbs: [get, list, watch]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus-operator
subjects:
- kind: ServiceAccount
  name: prometheus-operator
  namespace: monitoring
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus-operator
  namespace: monitoring
```

Having the new service account configured, we're ready to deploy the Operator itself, like so:

```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    k8s-app: prometheus-operator
  name: prometheus-operator
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: prometheus-operator
  template:
    metadata:
      labels:
        k8s-app: prometheus-operator
    spec:
      containers:
      - args:
        - --kubelet-service=kube-system/kubelet
        - --logtostderr=true
        - --config-reloader-image=quay.io/coreos/configmap-reload:v0.0.1
        - --prometheus-config-reloader=quay.io/coreos/prometheus-config-reloader:v0.29.0
        image: quay.io/coreos/prometheus-operator:v0.29.0
        name: prometheus-operator
        ports:
        - containerPort: 8080
          name: http
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
      nodeSelector:
        beta.kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
      serviceAccountName: prometheus-operator
```

### Prometheus server deployment

Before proceeding with the setup of Prometheus, we'll need to grant its instances with the right access control permissions. The following snippets from the Prometheus RBAC manifest do just that. Next, we create a ``ClusterRoleBinding`` to grant the permissions from the aforementioned ``ClusterRole`` to a user, which in our case will be a ``ServiceAccount``.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus-k8s
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus-k8s
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus-k8s
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
  namespace: monitoring
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus-k8s
  namespace: monitoring
```

Having the service account ready, we can now use the Prometheus Operator to deploy our Prometheus servers using the following manifest:

```
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  labels:
    prometheus: k8s
  name: k8s
  namespace: monitoring
spec:
  baseImage: quay.io/prometheus/prometheus
  version: v2.9.2
  replicas: 2
  resources:
    requests:
      memory: 200Mi
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus-k8s
  serviceMonitorNamespaceSelector: {}
  serviceMonitorSelector: {}
```

### Adding targets to Prometheus

So far, we've deployed the Operator and used it to deploy Prometheus itself. Now, we're ready to add targets and go over the logic of how to generate them.

Before proceeding, we'll also deploy an application to increase the number of available targets. For this, we'll be using the Hey application once again.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hey-deployment
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hey
  template:
    metadata:
      labels:
        app: hey
    spec:
      containers:
      - name: hey
        image: kintoandar/hey:v1.0.1
        imagePullPolicy: "IfNotPresent"
        resources:
          limits:
            cpu: 50m
            memory: 48Mi
          requests:
            cpu: 25m
            memory: 24Mi
        ports:
        - name: hey-port
          containerPort: 8000
        readinessProbe:
          httpGet:
            path: /health
            port: hey-port
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
```

```
apiVersion: v1
kind: Service
metadata:
  labels:
    squad: frontend
  name: hey-service
  namespace: default
spec:
  selector:
    app: hey
  type: NodePort
  ports:
  - name: hey-port
    protocol: TCP
    port: 8000
    targetPort: hey-port
```

Finally, we are going to create service monitors for both the Prometheus instances and the Hey application, which will instruct the Operator to configure Prometheus, adding the required targets. Pay close attention to the selector configuration – it will be used to match the services we created previously.

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    k8s-app: prometheus
  name: prometheus
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: web
  selector:
    matchLabels:
      prometheus: k8s
```

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app: hey
  name: hey-metrics
  namespace: default
spec:
  endpoints:
  - interval: 30s
    port: hey-port
  selector:
    matchLabels:
      squad: frontend
```

ServiceMonitors are the main building block when using the Prometheus Operator. You can configure anything that goes into a scrape job, such as scrape and timeout intervals, metrics endpoint to scrape, HTTP query parameters, and so on.

# Exporters and Integrations

Even though first-party exporters cover the basics pretty well, the Prometheus ecosystem provides a wide variety of third-party exporters that cover everything else. In this chapter, we will be introduced to some of the most useful exporters available — from **operating system (OS)** metrics and **Internet Control Message Protocol (ICMP)** probing to generating metrics from logs, or how to collect information from short-lived processes, such as batch jobs.

# Test environments for this chapter

In this chapter, we'll be using two test environments: 
* one based on virtual machines (VMs) that mimic traditional static infrastructure;
* one based on Kubernetes for modern workflows.

## Static infrastructure test environment

Not interested for now.

## Kubernetes test environment

```
kubectl apply -f ./provision/kubernetes/operator/bootstrap/
kubectl apply -f ./provision/kubernetes/operator/deploy/
kubectl apply -f ./provision/kubernetes/operator/monitor/
```

Content can be found in: ``./operator/``.

# Operating system exporter

Metrics for resources such as CPU, memory, and storage devices, as well as kernel operating counters and statistics provide valuable insight to assess a system's performance characteristics. For a Prometheus server to collect these types of metrics, an OS-level exporter is needed on the target hosts to expose them in an HTTP endpoint. 

## The Node Exporter

The Node Exporter is the most well-known Prometheus exporter, for good reason. It provides over 40 collectors for different areas of the OS, as well as a way of exposing local metrics for cron jobs and static information about the host. **Like the rest of the Prometheus ecosystem, the Node Exporter comes with a sane default configuration and some smarts to identify what can be collected, so it's perfectly reasonable to run it without much tweaking.**

Although this exporter was designed to be run as a non-privileged user, it does need to access kernel and process statistics, which aren't normally available when running inside a container. This is not to say that it doesn't work in containers—every Prometheus component can be run in containers—but that additional configuration is required for it to work. It is, therefore, **recommended that the Node Exporter be run as a system daemon directly on the host whenever possible.**

## Container exporter

**Container Advisor (cAdvisor)** is a project developed by Google that collects, aggregates, analyzes, and exposes data from running containers. The data available covers pretty much anything you might require, from memory limits to GPU metrics, all available and segregated by container and/or host.

Besides exposing metrics in the Prometheus format, cAdvisor also ships with a useful web interface, allowing the instant visualization of the status of hosts and their containers.

### Configuration

There are quite a few runtime flags, so we'll feature some of the most relevant for our test case in the following table:

![cAdvisor-command-line-flags.JPG](pictures/cAdvisor-command-line-flags.JPG)

### Deployment

Although historically the cAdvisor code was embedded in the Kubelet binary, it is currently scheduled to be deprecated there. Therefore, we'll be launching cAdvisor as a **DaemonSet** to future proof this example and to expose its configurations, while also enabling its web interface, as a Kubernetes service, to be explored.

Config files can be found in: ``./cadvisor/``.

It's time to add cAdvisor exporters as new targets for Prometheus. For that, we'll be using the next ``ServiceMonitor`` manifest as shown here:
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    p8s-app: cadvisor
  name: cadvisor-metrics
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: http
  selector:
    matchLabels:
      p8s-app: cadvisor
```

**With this, we now have container-level metrics. Do note that cAdvisor exports a large amount of samples per container, which can easily balloon exported metrics to multiple thousands of samples per scrape, possibly causing cardinality-related issues on the scraping Prometheus.**

From the thousands of metrics exported by cAdvisor, these are generally useful for keeping an eye out for problems:

* ``container_last_seen``, which keeps track of the timestamp the container was last seen as running;
* ``container_cpu_usage_seconds_total``, which gives you a counter of the number of CPU seconds per core each container has used;
* ``container_memory_usage_bytes`` and ``container_memory_working_set_bytes``, which keep track of container memory usage (including cache and buffers) and just container active memory, respectively;
* ``container_network_receive_bytes_total`` and ``container_network_transmit_bytes_total``, which let you know how much traffic in the container receiving and transmitting, respectively.

### kube-state-metrics

``kube-state-metrics`` does not export container-level data, as that's not its function. It operates at a higher level, exposing the Kubernetes state, providing metrics regarding the API internal objects such as pods, services, or deployments.

Some interesting metrics from kube-state-metrics that can be used to keep an eye on your Kubernetes clusters are:

* ``kube_pod_container_status_restarts_total``, which can tell you if a given pod is restarting on a loop;
* ``kube_pod_status_phase``, which can be used to alert on pods that are in a non-ready state for a long time;
* Comparing ``kube_<object>_status_observed_generation`` with ``kube_<object>_metadata_generation`` can give you a sense when a given object has failed but hasn't been rolled back.

# From logs to metrics

## mtail

Developed by Google, mtail is a very light log processor that is capable of running programs with pattern matching logic, allowing the extraction of metrics from said logs.

# Blackbox monitoring

## Blackbox exporter

The blackbox_exporter service exposes two main endpoints:

* /metrics: Where its own metrics are exposed;
* /probe: It is the query endpoint that enables blackbox probes, returning their results in Prometheus exposition format.

The /probe endpoint, when hit with an HTTP GET request with the parameters module and target, it executes the specified prober module against the defined target, and the result is then exposed as Prometheus metrics:

![blackbox-exporter.JPG](pictures/blackbox-exporter.JPG)

For example, a request such as http://192.168.42.11:9115/probe?module=http_2xx&target=example.com will return something like the following snippet (a couple of metrics were discarded for briefness):

```
# HELP probe_duration_seconds Returns how long the probe took to complete in seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 0.454460181
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 0
# HELP probe_http_status_code Response HTTP status code
# TYPE probe_http_status_code gauge
probe_http_status_code 200
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 4
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

# Pushing metrics

Not interested for a moment.

# More exporters

## JMX exporter

The Java Virtual Machine (JVM) is a popular choice for core infrastructure services, such as Kafka, ZooKeeper, and Cassandra, among others. These services, like many others, do not natively offer metrics in the Prometheus exposition format and instrumenting such applications is far from being a trivial task. In these scenarios, we can rely on the Java Management Extensions (JMX) to expose the application's internal state through the Managed Beans (MBeans). The JMX exporter extracts numeric data from the exposed MBeans and converts it into Prometheus metrics, exposing them on an HTTP endpoint for ingestion.

The exporter is available in the following two forms:

* **Java agent**: In this mode, the exporter is loaded inside the local JVM where the target application is running and exposes a new HTTP endpoint.
* **Standalone HTTP server**: In this mode, a separate JVM instance is used to run the exporter that connects via JMX to the target JVM and exposes collected metrics on its own HTTP server.

**The documentation strongly advises deploying the exporter using the Java agent**, for good reason; the agent produces richer sets of metrics as compared with the standalone exporter, as it has access to the full JVM being instrumented. However, both have trade-offs that are important to be aware of so that the right tool for the job is chosen.

# Prometheus Query Language - PromQL

## Getting to know the basics of PromQL

### Selectors

**A selector** refers to a set of label matchers. The metric name is also included in this definition as, technically, its internal representation is also a label, albeit a special one: ``__name__``.  Each label name/value pair in a selector is called a label matcher, and multiple matchers can be used to further filter down the time series matched by the selector. Label matchers are enclosed in curly brackets. If no matcher is needed, the curly brackets can be omitted. Example:
```
prometheus_build_info{version="2.9.2"}
```

This selector is equivalent to the following:
```
{__name__="prometheus_build_info", version="2.9.2"}
```

### Label matchers

Matchers are employed to restrict a query search to a specific set of label values. We'll be using the ``node_cpu_seconds_total`` metric to exemplify the four available label matcher operators: 
* ``=``. Using ``=``, we can perform an exact match on the label value. For instance, if we only match CPU core 0;
* ``!=``. We can also negate a match to obtain all the remaining time series using the ``!=`` matcher;
* ``=~``. This matcher includes results that match the expression. Example: ``node_cpu_seconds_total{cpu=~"(1|0)"}`` matches all where cpu = 1 and cpu = 0;
* ``!~``. This matcher excludes results that match the expression and allows all the remaining time series. Example: ``node_cpu_seconds_total{cpu!~"(1|0)"}`` matches all where cpu != 1 or cpu != 0.

Without any matching specification, this metric alone returns an instant vector with all the available time series containing the metric name, as well as all combinations of the CPU core numbers (``cpu=”0”``, ``cpu=”1”``) and CPU modes (``mode="idle"``, ``mode="iowait"``, ``mode="irq"``, ``mode="nice"``, ``mode="softirq"``, ``mode="steal"``, ``mode="user"``, ``mode="system"``), which makes a grand total of 16 time series, as shown in the following screenshot:

![node-cpu-seconds-total-prometheus.JPG](pictures/node-cpu-seconds-total-prometheus.JPG)

### Range vectors

A range vector selector is similar to an instant vector selector, but it returns a set of samples for a given time range, for each time series that matches it. To define a range vector selector query, you have to set an instant vector selector and append a range using square brackets ``[ ]``. Possible time units:
* s - seconds
* m - minutes
* h - hours
* d - days
* w - weeks
* y - years

```
http_requests_total{code="200"}[2m]
```

### The offset modifier

The offset modifier allows you to query data in the past. This means that we can offset the query time of an instant or range vector selector relative to the current time. It is applied on a per-selector basis, which means that offsetting one selector but not another effectively unlocks the ability to compare current behavior with past behavior for each of the matched time series.

```
http_requests_total{code="200"}[2m] offset 1h
```

### Subqueries

Subquery selector allows evaluation of functions that return instant vectors over time and return the result as a range vector. We'll be using the following query example to explain its syntax:
```
max_over_time(rate(http_requests_total{handler="/health", instance="172.17.0.9:8000"}[5m])[1h:1m])
```

Splitting the query into its components, we can see the following:
* ``rate(http_requests_total{handler="/health", instance="172.17.0.9:8000"}[5m])`` - The inner query to be run, which in this case is aggregating five minutes' worth of data into an instant vector;
* ``[1h`` - Just like a range vector selector, this defines the size of the range relative to the query evaluation time;
* ``:1m]`` - The resolution step to use. If not defined, it defaults to the global evaluation interval;
* ``max_over_time`` - The subquery returns a range vector, which is now able to become the argument of this aggregation operation over time.


**Subqueries are fairly expensive to evaluate, so it is strongly discouraged to use them for dashboarding, as recording rules would produce the same result given enough time.** Similarly, they should not be used in recording rules for the same reason. **Subqueries are best suited for exploratory querying, where it is not known in advance which aggregations are needed to be looked at over time.**

## Operators

### Binary operators

Binary operators: 
* arithmetic;
* comparison.

#### Arithmetic

The arithmetic operators provide basic math between two operands. Available arithmetic operators:
* ``+``
* ``-``
* ``*``
* ``/``
* ``%``
* ``^``

#### Comparison

* ``==``
* ``!=``
* ``>``
* ``<``
* ``>=``
* ``<=``

Say, for example, we have the following instant vector:
```
process_open_fds{instance="172.17.0.10:8000", job="hey-service"} 8
process_open_fds{instance="172.17.0.11:8000", job="hey-service"} 23
```

Comparison operator:
```
process_open_fds{job="hey-service"} > 10
```

The result will be as follows:
```
process_open_fds{instance="172.17.0.11:8000", job="hey-service"} 23
```

Moreover, **we can use the bool modifier to not only return all matched time series but also modify each returned sample to become 1 or 0**, depending on whether the sample would be kept or dropped by the comparison operator.

Example:
```
process_open_fds{job="hey-service"} > bool 10
```

Result:
```
process_open_fds{instance="172.17.0.10:8000", job="hey-service"} 0
process_open_fds{instance="172.17.0.11:8000", job="hey-service"} 1
```

### Vector matching

Vector matching, as the name implies, is an operation only available between vectors. So far, we have learned that when we have a scalar and an instant vector, the scalar gets applied to each sample of the instant vector. However, when we have two instant vectors, how can we match their samples?

#### One-to-one

Example:
```
node_filesystem_avail_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 100397019136
node_filesystem_avail_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/data"} 14120038400
node_filesystem_size_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 250685575168
node_filesystem_size_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/data"} 17293533184
```

Apply the following operation:
```
node_filesystem_avail_bytes{} / node_filesystem_size_bytes{} * 100
```

Returns:
```
{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 40.0489813060515
{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/data"} 81.64923991971679
```

It might be useful to aggregate vectors with mismatching labels. In those situations, you can apply the ``ignoring`` keyword right after the binary operator to ignore the specified labels. Additionally, it is also possible to restrict which labels from both sides should be used in matching by using the ``on`` keyword after the binary operator.

#### Many-to-one and one-to-many

Occasionally, you are required to perform operations where the element of one side is matched with several elements on the other side of the operation. When this happens, you are required to provide Prometheus with the means to interpret such operations. If the higher cardinality is on the left-hand side of the operation, you can use the ``group_left`` modifier after either ``on`` or ``ignoring``; if it's on the right-hand side, then ``group_right`` should be applied. The ``group_left`` operation is commonly used for its ability to copy labels over from the right side of the expression.

### Logical operators

These operators are the only ones in PromQL that work many-to-many:
* and - intersection. The and logical operator works by only **returning the matches from the left-hand side if the expression on the right-hand side has results with matching label key/value pairs.** All other time series from the left-hand side that do not have a match on the right-hand side are dropped;
* or - union. The union logical operator, ``or``, works by returning the elements from the left-hand side, except if there are no matches, it will return the elements from the right-hand side. Again, both sides need to have matching label names/values.
* unless - complement. The ``unless`` logical operator will return the elements from the first expression that do not match the label name/value pairs from the second.

Example:
```
node_filesystem_avail_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 1003970
node_filesystem_avail_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/data"} 141200
node_filesystem_size_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 2506855
node_filesystem_size_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/data"} 172935
```

Expression:
```
node_filesystem_size_bytes > 100000 and node_filesystem_size_bytes < 200000
```

Results:
```
node_filesystem_avail_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/data"} 141200
node_filesystem_size_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/data"} 172935
```

Expresion:
```
node_filesystem_avail_bytes > 200000 or node_filesystem_avail_bytes < 2500000
```

Results:
```
node_filesystem_avail_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 1003970
```

Expresion:
```
node_filesystem_avail_bytes unless node_filesystem_avail_bytes < 200000
```

Results:
```
node_filesystem_avail_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 1003970
node_filesystem_size_bytes{instance="172.17.0.13:9100", job="node-exporter-service", mountpoint="/Users"} 2506855
```

## Aggregation operators

By employing aggregation operators, we can take an instant vector and aggregate its elements, resulting in a new instant vector, usually with fewer elements.

Available aggregation:
* ``sum`` - Sums the elements
* ``min`` - Selects the minimum element
* ``max`` - Selects the maximum element
* ``avg`` - Calculates the average of the elements
* ``stddev`` - Calculates the standard deviation of the elements
* ``stdvar`` - Calculates the standard variance of the elements
* ``count`` - Counts the number of elements
* ``count_value`` - Counts the number of elements with the same value
* ``bottomk`` - The lower k elements by sample
* ``topk`` - The higher k elements by sample value
* ``quantile`` - Calculates the quantile of the elements

**There are two available modifiers to use in conjunction with aggregation operators that take a list of label names: ``without`` allows you to define which labels to aggregate away, effectively dropping those labels from the resulting vector, while ``by`` does exactly the opposite; that is, it allows you to specify which labels to keep from being aggregated.**

Example:
```
rate(http_requests_total[5m])
```

Would generate:
```
{code="200",endpoint="hey-port",handler="/",instance="172.17.0.10:8000",job="hey-service",method="get"} 5.891716069444445
{code="200",endpoint="hey-port",handler="/",instance="172.17.0.11:8000",job="hey-service",method="get"} 5.9669884444444445
{code="200",endpoint="hey-port",handler="/",instance="172.17.0.9:8000",job="hey-service",method="get"} 11.1336484826487
{code="200",endpoint="hey-port",handler="/health",instance="172.17.0.10:8000",job="hey-service",method="get"} 0.1
{code="200",endpoint="hey-port",handler="/health",instance="172.17.0.11:8000",job="hey-service",method="get"} 0.1
{code="200",endpoint="hey-port",handler="/health",instance="172.17.0.9:8000",job="hey-service",method="get"} 0.1000003703717421 
```

Aggregating:
```
sum(rate(http_requests_total[5m]))
```

Result:
```
{} 23.292353366909335
```

Example:
```
sum by (handler) (rate(http_requests_total[5m]))
```

Result:
```
{handler="/"} 22.99235299653759
{handler="/health"} 0.3000003703717421
```

## Functions

### absent()

This function is quite useful for alerting on, as the name suggests, absent time series.


For example, say that the instant vector exists and we execute the following expression:
```
absent(http_requests_total{method="get"})
```
This will return the following:
```
no data
```
Let's say we use an expression with a label matcher using a nonexistent label value, like in the following example:
```
absent(http_requests_total{method="nonexistent_dummy_label"})
```
This will produce an instant vector with the nonexistent label value:
```
{method="nonexistent_dummy_label"} 1
```
Let's apply absent to a nonexistent metric, as shown in this snippet:
```
absent(nonexistent_dummy_name)
```
This will translate into the following output:
```
{} 1
```

Finally, let's say we use absent on a nonexistent metric and a nonexistent label/value pair, as shown in the following snippet:
```
absent(nonexistent_dummy_name{method="nonexistent_dummy_label"})
```
The result can be seen in the following snippet:
```
{method="nonexistent_dummy_label"} 1
```

### label_join() and label_replace()

These functions are used to manipulate labels—they allow you to join labels to other ones, extract parts of label values, and even drop labels.

```
label_join(<vector>, <resulting_label>, <separator>, source_label1, source_labelN)
```

For example, say that we use the following sample data:
```
http_requests_total{code="200",endpoint="hey-port", handler="/",instance="172.17.0.10:8000",job="hey-service",method="get"} 1366
http_requests_total{code="200",endpoint="hey-port", handler="/health",instance="172.17.0.10:8000",job="hey-service",method="get"} 942
```
We then apply the following expression:
```
label_join(http_requests_total{instance="172.17.0.10:8000"}, "url", "", "instance", "handler")
```
We end up with the following instant vector:
```
http_requests_total{code="200",endpoint="hey-port", handler="/",instance="172.17.0.10:8000",job="hey-service", method="get",**url="172.17.0.10:8000/"**} 1366
http_requests_total{code="200",endpoint="hey-port", handler="/health",instance="172.17.0.10:8000",job="hey-service", method="get",**url="172.17.0.10:8000/health"**} 942
```

When you need to arbitrarily manipulate labels, ``label_replace`` is the function to use. The way it works is by applying a regular expression to the value of a chosen source label and storing the matched capture groups on the destination label.

Say that we take the preceding sample data and apply the following expression:
```
label_replace(http_requests_total{instance="172.17.0.10:8000"}, "port", "$1", "instance", ".*:(.*)")
```
The result will then be the matching elements with the new label, called port:
```
http_requests_total{code="200",endpoint="hey-port",handler="/", instance="172.17.0.10:8000", job="hey-service",method="get",port="8000"} 1366
http_requests_total{code="200",endpoint="hey-port",handler="/health", instance="172.17.0.10:8000", job="hey-service",method="get",port="8000"} 942
```
When using ``label_replace``, if the regular expression doesn't match the label value, the originating time series will be returned unchanged.

### predict_linear()

This function receives a range vector and a scalar time value as arguments. It extrapolates the value of each matched time series from the query evaluation time to the specified number of seconds in the future, given the trend in the data from the range vector. 

We'll apply the following expression, which employs ``predict_linear`` using a range of one hour of data, and extrapolate the sample value four hours in the future (60 (seconds) * 60 (minutes) * 4):
```
predict_linear(node_filesystem_free_bytes{mountpoint="/data"}[1h], 60 * 60 * 4)
{device="/dev/sda1", endpoint="node-exporter",fstype="ext4",instance="10.0.2.15:9100", job="node-exporter-service",mountpoint="/data", namespace="monitoring", pod="node-exporter-r88r6", service="node-exporter-service"} 15578514805.533087
```

### rate() and irate()

These two functions allow you to calculate the rate of increase of the given counters. Both automatically adjust for counter resets and take a range vector as an argument.

While the ``rate()`` function provides the **per second average rate of change over the specified interval** by using the first and last values in the range scaled to fit the range window, the ``irate()`` function uses the **last two values in the range for the calculation, which produces the instant rate of change.**

### histogram_quantile()

This function takes a float, which defines the required quantile (0 ≤ φ ≤ 1), and an instant vector of the gauge type as arguments. Each time series must have a ``le`` label (which means less than or equal to) to represent the upper bound of a bucket. This function also expects one of the selected time series to have a bucket named such as ``+Inf``, which works as the catch-all, the last bucket of the cumulative histogram. Since histograms that are generated by Prometheus client libraries use counters for each bucket, ``rate()`` needs to be applied to convert them into gauges for this function to do its work.

```
histogram_quantile(0.75, sum(rate(prometheus_http_request_duration_seconds_bucket[5m])) by (handler, le)) > 0
```

Result:
```
{handler="/"} 0.07500000000000001
{handler="/api/v1/label/:name/values"} 0.07500000000000001
{handler="/static/*filepath"} 0.07500000000000001
{handler="/-/healthy"} 0.07500000000000001
{handler="/api/v1/query"} 0.07500000000000001
{handler="/graph"} 0.07500000000000001
{handler="/-/ready"} 0.07500000000000001
{handler="/targets"} 0.7028607713983935
{handler="/metrics"} 0.07500000000000001
```

### sort() and sort_desc()

As their names suggest, sort receives a vector and sorts it in ascending order by the sample values, while sort_desc does the same function but in descending order.

### vector()

This is normally used as a way of ensuring an expression always has at least one result, by combining a vector expression with it, like in the following code:
```
http_requests_total{handler="/"} or vector(0)
```

## Aggregation operations over time

The aggregation operations we discussed earlier are always applied to instant vectors. When we need to perform those aggregations on range vectors, PromQL provides the ``*_over_time family`` of functions.

| Operation | Description |
| --------- | ----------- | 
| avg_over_time() | Average value of all samples in the range. |
| count_over_time() | Count of all samples in the range. |
| max_over_time() | Maximum value of all samples in the range. |
| min_over_time() | Minimum value of all samples in the range. |
| quantile_over_time() | The quantile of all samples in the range. It requires two arguments, the definition of the desired quantile, φ, as a scalar, where 0 ≤ φ ≤ 1, as a first argument and then a range-vector. |
| stddev_over_time() | The standard deviation of the sample's value in the range. |
| stdvar_over_time() | The standard variance of the sample's value in the range. |
| sum_over_time() | The sum of all sample values in the range.|

# Common patterns and pitfalls

Not interested for a moment.

# Moving on to more complex queries

Not interested for a moment.

# Troubleshooting and Validation

Not interested for a moment.

# Exploring promtool

Not interested for a moment.

# Logs and endpoint validation

Not interested for a moment.

# Analyzing the time series database

Not interested for a moment.

# Section 3: Dashboards and Alerts

















































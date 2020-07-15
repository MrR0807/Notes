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






























































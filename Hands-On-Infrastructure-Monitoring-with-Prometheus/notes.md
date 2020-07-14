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














































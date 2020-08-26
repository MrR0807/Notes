# Java, Spring, Prometheus and Grafana

#### Prerequisite

* Have docker installed

## Too long. Didn't not read

### Setup

Create necessary volumes:

```
docker volume create prom-config
docker volume create prom-data
docker volume create node-ex-data
```

Create a custom network, so other containers can talk to each other:
```
docker network create my-net
```

Create a dummy container to copy information from local computer to volume, if your corporation does not allow direct access 




### For corporate users

If your corporation firewalled docker direct access to filesystem, you can use dummy container, to copy ``prometheus.yml`` like so:

Create volume:
```
docker volume create prom-config
```

Copy to volume via dummy container:
```
docker container create --name dummy -v prom-config:/etc/prometheus/ hello-world
docker cp <full-path>/prometheus.yml dummy:/etc/prometheus/prometheus.yml
docker rm dummy
```

When launching prometheus, launch with volume:
```
docker run -d -p 9090:9090 --name prom --user root -v prom-config:/etc/prometheus -v prom-data:/data/prometheus --network my-net prom/prometheus --config.file="/etc/prometheus/prometheus.yml" --storage.tsdb.path="/data/prometheus"
```





Start infrastructure containers:
```
docker run -d -p 9090:9090 --name prom --user root -v prom-config:/etc/prometheus -v prom-data:/data/prometheus --network my-net prom/prometheus --config.file="/etc/prometheus/prometheus.yml" --storage.tsdb.path="/data/prometheus"

docker run -d -p 9100:9100 --name nex --user 995:995 -v node-ex-data:/hostfs --network my-net prom/node-exporter --path.rootfs=/hostfs

docker run -d --name=grafana -p 3000:3000 --network my-net grafana/grafana
```

Compile and build your own application:
```

mvn clean
mvn install -DskipTests
docker image build  -t citizen .

docker run -d -p 8080:8080 --name citizen --network my-net citizen
```

prometheus.yml:
```
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']

  - job_name: node-exporter
    static_configs:
      - targets: ['nex:9100']

  - job_name: citizen-app
    metrics_path: /actuator/prometheus
    static_configs:
      - targets: ['citizen:8080']
```


# Testing Alertmanager

prometheus.yml:
```
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  - "alerting_rules.yml"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']

  - job_name: node-exporter
    static_configs:
      - targets: ['nex:9100']

  - job_name: citizen-app
    metrics_path: /actuator/prometheus
    static_configs:
      - targets: ['citizen:8080']
```

alerting_rules.yml:
```
groups:
- name: alerting_rules
  rules:
  - alert: NodeExporterDown
    expr: up{job="node-exporter"} != 1
    for: 1m
    labels:
      severity: "critical"
    annotations:
      description: "Node exporter {{ $labels.instance }} is down."
      link: "https://example.com"
```

Copy ``alerting_rules.yml`` to volume:
```
docker container create --name dummy -v prom-config:/etc/prometheus/ hello-world
docker cp <full-path>/alerting_rules.yml dummy:/etc/prometheus/alerting_rules.yml
docker rm dummy
docker container restart prom
```

Go to ``http://localhost:9090/rules``. You should see something like this:
![alerting-rules-prometheus-localhost.JPG](pictures/alerting-rules-prometheus-localhost.JPG)

Stop ``nex`` container. Go to ``http://localhost:9090/alerts``.





----------------------

Prometheus Queries:
```

```


Cpu Usage (system_cpu_usage{application="$application", instance="$instance"}, process_cpu_usage{application="$application", instance="$instance"}, avg_over_time(process_cpu_usage{application="$application", instance="$instance"}[1h]))
Memeory Usage (sum(jvm_memory_used_bytes{application="$application", instance="$instance"}), sum(jvm_memory_committed_bytes{application="$application", instance="$instance"}), sum(jvm_memory_max_bytes{application="$application", instance="$instance"}))
Uptime (process_uptime_seconds{application="$application", instance="$instance"})
Latency (moving average)
increase(http_server_requests_seconds_sum[1m])
/
increase(http_server_requests_seconds_count[1m])

Latency spike

Traffic (requests per second) sum(increase(http_server_requests_seconds_count{outcome != "REDIRECTION", uri !~ "/|/\\*\\*|/actuator.*|/swagger-resources.*|/webjars.*|root"}[5m]))
Errors (failing requests per second) sum(increase(http_server_requests_seconds_count{outcome !~ "REDIRECTION|SUCCESS", uri !~ "/|/\\*\\*|/actuator.*|/swagger-resources.*|/webjars.*|root"}[5m]))






## Docker compose

```
version: "3.3"
services:
  prom:
    image: prom/prometheus
    ports:
      - 9090:9090
    volumes:
      - prom-config:/etc/prometheus
      - prom-data:/data/prometheus
    command: 
      - '--config.file=/etc/prometheus/prometheus.yml' 
      - '--storage.tsdb.path="/data/prometheus"'
volumes:
  prom-data:
  prom-config:
    external: true
```

```
docker-compose -f docker-compose-prom.yml up -d
```

```
docker-compose -f docker-compose-prom.yml down
```

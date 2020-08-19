```
docker volume create prom-config
docker volume create prom-data
docker volume create node-ex-data
```

We need to create a custom network, so other containers can talk to each other:
```
docker network create my-net
```

I needed a dummy container to copy information from my work computer to volume, because it was blocked by firewalls:
```
docker container create --name dummy -v prom-config:/etc/prometheus/ hello-world
docker cp <full-path>/prometheus.yml dummy:/etc/prometheus/prometheus.yml
docker rm dummy
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




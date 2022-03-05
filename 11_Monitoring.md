# Monitoring

## DataDog

DataDog provide metrics/visualization/monitoring.

### DataDog Archtecture

deploy DataDog Agent to each nodes with using DaemonSet.

```mermaid
graph TD

datadog(DataDogHQ)

subgraph node1
  c11(DataDog)
  c12(Container)
end

subgraph node2
  c21(DataDog)
  c22(Container)
end

subgraph node3
  c31(DataDog)
  c32(Container)
end

c12-->c11
c22-->c21
c32-->c31
c11-->datadog
c21-->datadog
c31-->datadog
```

kube-state-metrics plugin recommended.

### install

ex) datadog_values.yaml

```yaml
datadog:
  # DataDog SaaS server certificates
  apiKey: xx
  appKey: xx
  tags: "project:sample,env:dev"
  clusterAgent:
    enabled: true
    metricsProvider:
      enabled: true
  processAgent:
    enabled: true
    processCollection: true
  collectEvents: true
  loadElection: true
```

```shell
helm install sample-datadog \
  --version 2.3.0 \
  -f datadog_values.yaml
```

### DataDog Metrics

| metrics key | description |
| :----- | :----- |
| `docker.*` | for docker container |
| `kubernetes.*` | for k8s |
| `kubernetes_state.*` | for k8s ccluster level |

### watch container and alert

+ Anomary Monitoring
  + compare with past trends
+ Forecast Monitoring
  + forecast trends in metrics
+ Outlier Monitoring
  + detect differ action in specify group

## Prometheus

Promethous is observation OpenSource tool provided by CNCF.

### Prometheus Archtecture

| Actor | desctiption |
| :----- | :----- |
| Prometheus Server | collection metrics and saving |
| Alert Manager | provide alert notification |
| Expoter | provide middleware metrics to prometheus server |
| Push Gateway | provide batch metrics to prometheus server |

```mermaid
graph TD

subgraph Prometheus
  c1(Prometheus Server)
  c2(Alert Manger)
  c3(Exporter)
  c4(Push Gateway)
end

c5(User)
c6(middleware)
c7(Batch Process)

c6-->|metrict|c3
c7-->|push metricw|c4
c3-->|metrics|c1
c4-->|metrics|c1
c1-->|action|c2
c2-->|alert|c5
```

### install Prometheus

```shell
git clone git@github.com:coreos/kube-prometheus.git -b v0.5.0
kubectl apply -f kube-prometheus/manifests/setup/
kubectl apply -f kube-prometheus/manifests/
```

port forward setting

```shell
# for Graphana
kubectl -n monitoring port-forward service/grafana 3000
# for Prometheus Server
kubectl -n monitoring port-forward service/prometheus-k8s 9090
# for AlermManager
kubectl -n monitoring port-forward service/alertmanager-main 9093
```

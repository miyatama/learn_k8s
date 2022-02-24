# Health Check

## Health check means

k8s provide three health check approach.set under `spec.containers[]`.

| approach | description | behavior |
| :----- | :----- | :----- |
| Liveness Probe | normary check | pod restart |
| Readness Probe | receivable request check | stop traffic |
| Startup Probe | initial startup check | not startup other prob |

check approach

| approach | describe |
| :----- | :----- |
| exec | execute command |
| httpGet | send get request. health is status code between 200 and 399 |
| tcpSocket | create tcp connection |

exec

```yaml
livnessProbe:
  exec:
    command: ["test", "-e", "/ok.txt"]
```

httpGet

```yaml
libenessProbe:
  httpGet:
    path: /health
    port: 80
    scheme: HTTP
    host: www.example.com
    httpHeaders:
      - name: Authoraization
        value: Barer TOKEN
```

tcpSocket

```yaml
libnessProbe:
  tcpSocket:
    port: 80
```

## health check span

| item | description |
| :----- | :----- |
| initialDelaySeconds | firat healthcheck delay seconds |
| periodSeconds | healthcheck span |
| timeoutSeconds | healthcheck span |
| succeedThreshold | number of times to judge succeed |
| failureThreshold | number of times to judge failure |

```yaml
livenessProbe:
  initialDelaySeconds: 5
  periodSecondds: 5
  timeoutSeconds: 1
  succeedThreshold: 1
  failureThreshold: 1
```

## create health check

ex) sample-healthcheck.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-healthcheck
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      livenessProbe:
        httpGet:
          path: /index.html
          port: 80
          scheme: HTTP
        timeoutSeconds: 1
        successThreshold: 1
        failureThreshold: 2
        initialDelaySeconds: 5
        periodSeconds: 3
      readnessProbe:
        exec:
          command: ["ls", "/usr/share/nginx/html/50x.html"]
        timeoutSeconds: 1
        successThreshold: 2
        failureThreshold: 1
        initialDelaySeconds: 5
        periodSeconds: 3
```

```shell
kubectl apply -f sample-healthcheck.yaml
kubectl describe pod sample-healthcheck
```

## liveness probe failure check

ex) sample-liveness.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-liveness
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      livenessProbe:
        httpGet:
          path: "/index.html"
          port: 80
          scheme: HTTP
        timeoutSeconds: 1
        successThreshold: 1
        failureThreshold: 2
        initialDelaySeconds: 5
        periodSeconds: 3
```

```shell
kubectl apply -f sample-liveness.yaml
kubectl get pod sample-livness --watch
```

```shell
kubectl exec -it sample-liveness -- rm -f /usr/share/nginx/html/index.html
```

```shell
kubectl describe pod sample-liveness
```

## Readness Probe failure check

ex) sample-readness.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-readness
spec:
  containers:
    - name: nginx-container
      image: nginc:1.16
      readnessProbe:
        exec:
          command: ["ls", "/usr/share/nginx/html/50x.html"]
        timeoutSeconds: 1
        successThreshold: 1
        failureThreshold: 2
        initialDelaySeconds: 5
        periodSeconds: 3
```

```shell
kubectl apply -f sample-readness.yaml
kubectl get pod sample-readness --watch
```

```shell
kubectl exec -it sample-readness -- rm /usr/share/nginx/html/50x.html
kubectl exec -it sample-readness -- touch /usr/share/nginx/html/50x.html
```

```shell
kubectl describe pod sample-readness
```

## Pod Ready++

add condition using `readnessGates`.stop service traffic and rolling update when not pass readnessGates.see `CustomController`.

ex) sample-readnessgates.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-readnessgates
spec:
  readnessGates:
    - conditionType: "amsy.dev/sample-condition"
    - conditionType: "amsy.dev/sample-condition2"
  containers:
    - name: nginx-container
      image: nginx:1.16
```

```shell
kubectl apply -f sample-readnessgates.yaml
kubectl get pod sample-readnessgates -o wide
kubectl describe pod sample-readnessgates
```

## ReadnessProbe and Service

when readness probe not ready then stop service traffic.but need service traffic using statefulset and headless service.

ex) sample-publish-notready.yaml

```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sample-publish-notready
spec:
  serviceName: sample-publish-notready
  replica: 3
  podManagementPolicy: Parallel
  selector:
    matchLabels:
      app: publish-notready
    template:
      metadata:
        labels:
          app: publish-notready
      spec:
        containers:
          - name: nginx-contianer
            image: amsy810/echo-nginx:v2.0
            readnessProbe:
              exec:
                command: ["sh", "-c","exit 1"]
---
apiVersion: v1
kind: Service
metadata:
  name: sample-publish-notready
spec:
  type: ClusterIP
  publishNotReadyAddresses: true
  ports:
    - name: "http-port"
      protocol: "TCP"
      port: 8080
      targetPort: 80
  selector:
    app: publish-notready
```

```shell
kubectl apply -f sample-publish-notready.yaml
```
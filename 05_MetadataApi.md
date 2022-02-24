# Metadata API

+ LimitRange
+ HorizontalPodAutoscaler
+ PodDistuptionBudget
+ CustomResourceDefinition

## LimitRange

LimitRange resource provide max value, min value and default value for pod, container and PersistentVolumeClaim.

| item | description |
| :----- | :----- |
| default | default limit value |
| defaultRequest | default request value |
| max | max resource |
| min | min resource |
| maxLimitRequestRatio | Limits/Request ratio |

resource and control

| resource | allow control |
| :----- | :----- |
| Pod | max/min/maxLimitRequestRatio |
| Container | default/defaultRequest/max/min/maxLimitRequestRatio |
| PersistentVolumeClaim | max/min |

### default define LimitRange

check default LimitRange

```shell
kubectl get limitranges limits -o yaml
```

```text
```

### LimitRange for container

ex) sample-limitrange-container.yaml

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: sample-limitrange-container
spec:
  limits:
    - type: Container
      default:
        memory: 512Mi
        cpu: 500m
      defaultRequest:
        memory: 256Mi
        cpu: 250m
      max: 
        memory: 1024Mi
        cpu: 1000m
      min:
        memory: 128Mi
        cpu: 125m
      maxLimitRequestRatio:
        memory: 2
        cpu: 2
```

check default resource limit

```shell
kubectl apply -f sample-pod.yaml
kubectl get pod sample-pod -o json | jq 'spec.containers[].resources'
kubectl delete pod sample-pod
```

check LimitRange

```shell
kubectl apply -f sample-limitrange-container.yaml
kubectl apply -f sample-pod.yaml
kubectl get pod sample-pod -o json | jq 'spec.containers[].resources'
```

### LimitRange for Pod

ex) sample-limitrange-pod.yaml

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: sample-limitrange-pod
  namespace: default
spec:
  limits:
  - type: Pod
    max:
      memory: 2048Mi
      cpu: 2000m
    min:
      memory: 128Mi
      cpu: 125m
    maxLimitRequestRatio:
      memory: 1.5
      cpu: 1.5
```

### LimitRange for PersistentVolumeClaim

ex) sample-limitrange-persistentvolumeclaim

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: sample-limitrange-persistentvolumeclaim
  namespace: default
spec:
  limits:
  - type: PersistentVolumeClaim
    max:
      storage: 20Gi
    min:
      storage: 3Gi
```

```shell
kubectl apply -f sample-limitrange-persistentvolumeclaim.yaml
# fail
kubectl apply -f sample-pvc-fail.yaml
```

## HorizontalPodAutoscaler

HorizontalPodAutoscaler resource that vary the number of replicas depending on the loadaverage.

+ high loadaverage => scale out
  + scale out up to once every 3 minute
  + scale out raise `avg(cpu usage) / targetAverageUtilization > 1.1`
+ low loadaverage => scale in
  + scale in up to once every 5 minute
  + scale in raise `avg(cpu usage) / targetAverageUtilization < 0.9`

HorizontalPodAutoscaler check status every 30 sec.

needfull numer of replicas = ceil(sum(cpu usage)/ targetAverageUtilization)
cpu usage is one minuite average from metric-server.

usable metrics

| metrics | description |
| :----- | :----- |
| Resource | cpu/memory resource |
| Object | kubernetes object resource(ex: Ingress hiss/s) |
| Pods | Pod metrics(connection count) |
| External | metrics outside k8s.(ex: LB, Cloud Pub/Sub) |

ex) sample-horizontalpodautoscaler.yaml

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: sample-horizontalpodautoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sample-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
```

```shell
kubectl apply -f sample-horizontalpodautoscaler.yaml
```

or command apporach

```shell
kubectl autoscale deployment sample-deployment \
  --cpu-percent=50 \
  --min=1 \
  --max=10
```

check status

```shell
kubectl get horizontalpodautoscalers
```

## VerticalPodAutoscaler

HorizontalPodAutoscaler is scale out/in.VerticalPodAutoscaler is scale up.

ex) sample-vpa.yaml

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: sample-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sample-vpa-deployment
  updatePolicy:
    updateMode: Auto
  resourcePolicy:
    containerPolicies:
    - containerName: no-vpa-container
      mode: "Off"
    - containerName: "*"
      mode: "Auto"
      minAllow:
        memory: 300Mi
      maxAllow:
        memory: 1000Mi
      controlledResources: ["memory", "cpu"]
```

updateMode

| value | description |
| :----- | :------ |
| Off | calc only. no update |
| Initial |  update requests to recommended value when pod creatation |
| Recreate | recreate pod with recommended request value when change recommended value |
| InPlace(not release) | update request to recommended value when change recommended value |
| Auto | update request to recommended value(or recreate pod) when change recommended value |

```shell
kubectl describe vpa sample-vpa
```

| meter | description |
| :----- | :----- |
| Lower Bound | Requests lower recommended value |
| Upper Bound | Requests upper recommended value |
| Target | Request recommended value |
| Uncapped Target | recommmended value when ignore resource limit |

Max Allow <- pod update range -> Upper Bound -- Target -- Lower Bound <- pod update range -> Min Allow
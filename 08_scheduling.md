# Scheduling

k8s scheduling have 2 phase `filtering`(Predicates) and `scoreing`(Priority).

+ filtering
  + calculate assinable node
    + resource size
    + labels
+ scoreing
  + assing priority to assinable node, and choose the most suitable node.

## manifest

+ specify node that user wanna place
  + nodeSelector -> Filtering
  + Node Affinity -> Filtering/Scoreing
  + Node Anti-Affinity -> Filtering/Scoreing
  + Inter-Pod Affinity -> Filtering/Scoreing
  + Inter-Pod Anti-Affinity -> Filtering/Scoreing
+ specify node that administrator not wanna place
  + Taints/Toleration -> Filtering/Scoreing

| kind | description |
| :------ | :----- |
| nodeSelector | easy Node Affinity |
| Node Affinity | Specify Node Only |
| Node Anti-Affinity | Except Specify Node |
| Inter-Pod Affinity | Specity Pod exists node |
| Inter-Pod Anti-Affinity | except specify pod exists node |

## build-in node label and add label

+ build-in node labe
  + hostname
  + os
  + archtecture
  + instance type
  + resion
  + zone
  + k8s platform
  + distribution environment

add label example

```shell
# disk type, cpu spec,cpu gen 
nodeid=`kubectl get nodes | tail -n 1 | awk '{print $1}'`
kubectl label node ${nodeid} \
  disktype=hdd \
  cpuspec=low \
  cpugen=3
nodeid=`kubectl get nodes | tail -n 2 | head -n 1 | awk '{print $1}'`
kubectl label node ${nodeid} \
  disktype=ssd \
  cpuspec=low \
  cpugen=2
nodeid=`kubectl get nodes | tail -n 2 | head -n 2 | awk '{print $1}'`
kubectl label node ${nodeid} \
  disktype=ssd \
  cpuspec=high \
  cpugen=4
kubectl -L=disktype,cpuspec,cpugen get node
```

WARNING!do not use `kubernetes.io` namespace.

## nodeSelector

ex) sample-nodeselector.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-nodeselector
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
  nodeSelector:
    disktype: ssd
```

## Node Affinity

ex) sample-node-affinity.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-node-affinity
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
  affinity:
    nodeAffinity:
      # must condition
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchLabels:
            - key: disktype
              operator: In
              values:
                - hdd
      # prioritiez condition
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preferrence:
            matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - gke-k8s-default-pool-xxxx-0000
```

```shell
kubectl apply -f sample-node-affinity.yaml
kubectl get pod sample-node-affinity -o wide
kubectl delete pod sample-node-affinity
kubectl drain gke-k8s-default-pool-xxxx-0000
kubectl apply -f sample-node-affinity.yaml
kubectl get pod sample-node-affinity -o wide
kubectl uncordon gke-k8s-default-pool-xxxx-0000
```

failure case like a Lack a resource.

ex) sample-node-affinity-failure.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-node-affinity-failure
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoreDuring:
          # or condition
          nodeSelectorTerms:
          # and condition
          - matchExpressions:
            - key: disktype
              operator: In
              values:
                - nvme
        prefferedDuringSchedulingIgnoredDuring:
        - weight: 1
          preference:
            # and condition
            matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - gke-k8s-default-pool-xxxx-0000
```

```shell
kubectl apply -f sample-node-affinity-failuer.yaml
kubectl get pod sample-node-affinity-failuer -o wide
kubectl delete pod sample-node-affinity-failuer
kubectl drain gke-k8s-default-pool-xxxx-0000
kubectl apply -f sample-node-affinity-failuer.yaml
kubectl get pod sample-node-affinity-failuer -o wide
kubectl uncordon gke-k8s-default-pool-xxxx-0000
```

## matchExpressions - operation and set-based

`matchExpressions` use a set-based condition.

```ex
- matchExpressions:
  - key: disktype
    operator: In
    values:
      - ssd
      - hdd
# oneliner
- matchExpressions:
  - {key: disktype, operator: In, values: {ssd, hdd}}
```

operators

| operator | how to use | description |
| :---- | :----- | :----- |
| In | A In [B, ...] | like a sql `in` statement |
| NotIn | A NotIn [B, ...] | like a sql `not in` statement |
| Exists | A Exists [] | label A exists |
| DoesNotExists | A DoesNotExists [] | label A not exists |
| Gt | A Gt [B] | Label A value greater than B |
| Lt | A Lt [B] | Label A value lower than B |

Gt/Lt operators can use for ReplicaSet/Deployment/DaemonSet/StatefulSet/Job

ex) sample-matchexpressions-deployments.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-matchexpressions-deployments
spec:
  replicas: 3
  selector:
    - matchExpressions:
      - key: app
        operator: In
        values: 
          - sample-app
          - sample-application
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.16
```

## Not Anti-Affinity

`Not Anti-Affinity` is not exists.It is realized by the negative form of nodeAffinity.

```yaml
requiredDuringSchedulingIgnoreDurlingExcusion:
  nodeSelectorTerms:
    - matchExpressions:
      - key: disktype
        operator: NotIn
        values:
          - hdd
```

## Inter-Pod Affinity

use `spec.affinity.podAffinity`.

ex) sample-affinity-host.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod-affinity-host
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoreDurlingExcusion:
        - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - sample-app
          topologyKey: kubernetes.io/hostname
          # topologyKey: topology.kubernetes.io/zone
  containers:
    - name: nginx-container
      image: nginx:1.16
```

usable `preferredDuringSchedulingIgnoreDuringExcusion`.

ex) sample-pod-affinity-zone-host.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod-affinity-zone-host
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoreDuringExpression:
        - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - sample-app
          topologyKey: topology.kubernetes.io/zone
      preferredDuringSchedulingIgnoreDuringExpression:
        - weight: 1
          podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: app
                  operator: In
                  values:
                    - sample-app
              topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx-container
      image: nginx:1.16
```

## Inter-Pod Anti-Affinity

use `spec.affinity.podAntiAffinity`.

ex) sample-pod-antiaffinity-host.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod-antiaffinity-host
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoreDuringExpression:
        - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
              - sample-app
          topologyKey: kubernetes.io/hostname
```

## complex condition

example scheduling

+ required
  + disktype is `ssd` or `nvme`
  + same zone of specify pod
  + not exists specify pod
+ prioritiez
  + cpu generation greater than 2

ex) sample-pod-complex-scheduling.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod-complex-scheduling
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
  affinity:
    nodeAffinity:
      requiredDuringScheludingIgnoreDuringExcusion:
        nodeSelectTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
              - ssd
              - nvme
      prefferedDuringSchedulingIgnoreDuringExcusion:
      - weight: 1
        preference:
          matchExpressions:
            - key: cpugen
              operator: Gt
              values: 
              - "2"
    podAffinity:
      requiredDuringScheludingIgnoreDuringExcusion:
        - labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
                - sample-app
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      requiredDuringScheludingIgnoreDuringExcusion:
        - labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
                - sample-app
        topologyKey: kubernetes.io/hostname
```

## Taints and Tolerations

specify node that administrator not wanna place.
add Taints format `Key=Value:Effect`.

Effects

| Effect | severity | description |
| :----- | :----- | :----- |
| PreferNoSchedule | low | do not schedule as much as possible. |
| NoSchedule | middle | no schedule and keep pod |
| NoExecute | high | no schedule and remove pod |

```shell
nodeid=`kubectl get nodes | tail -n 1 | awk '{print $1}'`
kubectl taint node ${nodeid} env=prd:NoSchedule
kubectl taint node  -l kubernetes.io/os=linux env=prd:NoSchedule
# remove taints
kubectl taint node ${nodeid} env-
kubectl taint node ${nodeid} env:NoSchedule
# check taints
kubectl describe node ${nodeid}
```

pod start using tolerations.

ex) sample-tolerations.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-tolerations
spec:
  containers:
    - name: ngnix-container
      image: nginx:1.16
  tolerations:
  - key: "env"
    # Equal or Exists
    operator: "Equal"
    # not set value and effect is wildcard
    value: "prd"
    effect: "NoSchedule"
```

`env=prd:PreferNoSchedule` taints node

| pod specity effect | `key=env, value=prd, operator=Equal` | `key=env, operator=Exists` |
| :----- | :----: | :----: |
| preferNoSchedule | yes | yes |
| NoSchedule | yes | yes |
| NoExecute | yes | yes |
| wildcard | yes | yes |

`env=prd:NoSchedule` taints node

| pod specity effect | `key=env, value=prd, operator=Equal` | `key=env, operator=Exists` |
| :----- | :----: | :----: |
| preferNoSchedule | no | no |
| NoSchedule | yes | yes |
| NoExecute | no | no |
| wildcard | yes | yes |

`env=prd:NoExecute` taints node

| pod specity effect | `key=env, value=prd, operator=Equal` | `key=env, operator=Exists` |
| :----- | :----: | :----: |
| preferNoSchedule | no | no |
| NoSchedule | no | no |
| NoExecute | yes | yes |
| wildcard | yes | yes |

## `NoExecute` pod stop time range

45 wait before stop

ex) sample-torelation-seconds.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-torelation-seconds
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
  torelations:
  - key: "env"
    operator: "Equal"
    effect: "NoExecute"
    torelationSeconds: 45
```

```shell
nodeid=`kubectl get nodes | tail -n 1 | awk '{print $1}'`
kubectl cordon -l kubernetes.io/os=linux
kubectl uncodon ${nodeid}
kubectl taint node ${nodeid} \
  env=prd:NoExecute
```

```shell
kubectl get pods --watch
```

## k8s taints

k8s assing taints to node when failure case.

| effect | key | description |
| :----- | :----- | :---- |
| NoSchedule | node.kubernetes.io/memory-pressure | node memory depletion |
| NoSchedule | node.kubernetes.io/disk-pressure | node disk depletion |
| NoSchedule | node.kubernetes.io/pid-pressure | node process count depletion |
| NoSchedule | node.kubernetes.io/network-unavailable | node network unavairable |
| NoSchedule | node.kubernetes.io/unschedulable | execute `kubectl cordon` |

by cloud

| effect | key | description |
| :----- | :----- | :----- |
| NoSchedule | node.cloudprovider.kubernetes.io/uninitialized | wait node startup by cloud provider |
| NoSchedule | node.cloudprovider.kubernetes.io/shutdown | wait node shutdown by cloud provider |

## PriorityClass

`PriorityClass` provide a pod scheduling priority.

ex) sample-priority-class.yaml

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: sample-priority-class
value: 100
globalDefault: false
description: "use for ServiceA only"
```

Pod using PriorityClass

ex) sample-high-priority.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-high-priority
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
  priorityClassName: sample-priority-class
```

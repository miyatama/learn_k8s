# Cluster API

resource for Security and Quota.

+ Node
+ Namespace
+ PersistentVolume
+ ResourceQuota
+ ServiceAccount
+ Role
+ ClusterRole
+ RoleBinding
+ ClusterRoleBinding
+ NetworkPolicy

## Node

```shell
kubectl get nodes -o wide
kubectl get nodes -o yaml
```

| key | value |
| :----- | :----- |
| `status.address` | InternalIP or ExternalIP |
| `status.allocatable` | allocatable resource size |
| `status.capacity` | assigned resource size |

allocated resource = status.allocatable - status.capacity

```shell
id=`kubectl get nodes | tail -n 1 | awk '{print $1}'`
kubectl describe node ${id}
```

when node status is not ready, check `status.condition`

```shell
kubectl get nodes -o yaml
```

## Namespace

cluster divide resource.

### create namespace

ex) sample-namespace.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: sample-namespace
```

```shell
kubectl apply -f sample-namespace.yaml
```

or `kubectl create` command

```shell
kubectl create namespace sample-namespace
```

### get resource ignored namespace

specific namespace select use `-n` option

```shell
kubectl ge pods -n sample-namespace
```

all namespace

```shell
kubectl get pods --all-namespaces
```

## ResourceQuota

ResourceQuota provide resource limit for namespace.
`worning!! cerate ResourceQuota before creating pod.`

provide function

+ limit the number of resources that can be created
+ resource usage limit

ex) sample-resourcequota.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: sample-resourcequota
  namespace: default
spec:
  hard:
    count/configmaps: 10
```

```shell
kubectl apply -f sample-resourcequota.yaml
kubectl describe resourcequota
for i in `seq 1 11`; do kubectl create configmap conf-${i}   --from-literal=key1=val1; done
```

### limit the number of resources that can be created

these are two notations.

ex) sample-resourcequota-count-new.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: sample-resourcequota-new
  namespace: default
spec:
  hard:
    count/persistentvolumeclaim: 10
    count/services: 10
    count/secrets: 10
    count/configmaps: 10
    count/replicationcontrollers: 10
    count/deployments.apps: 10
    count/replicasets.apps: 10
    count/statefulsets.apps: 10
    count/jobs.batch: 10
    count/cronjobs.batch: 10
    count/deployments.extensions: 10
```

ex) sample-resourcequota-count-old.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: sample-resourcequota-old
  namespace: default
spec:
  hard:
    sample-storageclass.storageclass.storage.k8s.io/persistentvolumeclaims: 10
    services.loadbalancers: 10
    services.nodeports: 10
    pod: 10
    persistentvolumeclaims: 10
    replicationcontrollers: 10
    secrets: 10
    configmaps: 10
    services: 10
    resourcequotas: 10
```

### resource usage limit

ex) sample-resourcequota-usage.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata: 
  name: sample-resourcequota-usage
  namespace: default
spec:
  hard:
    requests.memory: 2Gi
    requests.storage: 5Gi
    sample-storageclass.storageclass.storage.k8s.io/requests.storage: 5Gi
    requests.ephemeral-storage: 5Gi
    requests.nvidia.com/gpu: 2

    limits.memory: 4
    limits.ephemeral-storage: 10Gi
    limits.nvidia.com/gpu: 4
```

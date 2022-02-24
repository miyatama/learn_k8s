# outline

using `gcloud` console.check useful k8s version.

```shell
gcloud container get-server-config --zone asia-northeast1-a
gcloud container clusters create k8s \
  --cluster-version 1.21.6-gke.1500 \
  --zone asia-northeast1-a \
  --num-nodes 3 \
  --machine-type n1-standard-4 \
  --enable-network-policy \
  --enable-vertical-pod-autoscaling
# when enable alpha functions
gcloud container clusters create k8s-alpha \
  --cluster-version 1.21.6-gke.1500 \
  --zone aisa-northeast1-a \
  --num-nodes 3 \
  --machine-type n1-standard-4 \
  --enable-network-policy \
  --enable-vertical-pod-autoscaling \
  --enable-kubernetes-alpha \
  --no-enable-autorepair \
  --no-enable-autoupgrade
```

result

```text
WARNING: Currently VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
WARNING: Starting with version 1.18, clusters will have shielded GKE nodes by default.
WARNING: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
WARNING: Starting with version 1.19, newly created clusters and node-pools will have COS_CONTAINERD as the default node image when no image type is specified.
Creating cluster k8s in asia-northeast1-a...done.     
Created [https://container.googleapis.com/v1/projects/miyata080825/zones/asia-northeast1-a/clusters/k8s].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/asia-northeast1-a/k8s?project=miyata080825
kubeconfig entry generated for k8s.
NAME: k8s
LOCATION: asia-northeast1-a
MASTER_VERSION: 1.21.6-gke.1500
MASTER_IP: 34.85.90.137
MACHINE_TYPE: n1-standard-4
NODE_VERSION: 1.21.6-gke.1500
NUM_NODES: 3
STATUS: RUNNING
```

get certificate

```shell
gcloud container clusters get-condencials k8s --zone aisa-northeast1-a
```

result

```text
Fetching cluster endpoint and auth data.
kubeconfig entry generated for k8s.
```

grant resource access

```shell
kubectl create clusterrolebinding user-cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=n.miyata080825@gmail.com
```

result

```text
clusterrolebinding.rbac.authorization.k8s.io/user-cluster-admin-binding created
```

check k8s version

```shell
kubectl version
```

result

```text
Client Version: version.Info{Major:"1", Minor:"22", GitVersion:"v1.22.4", GitCommit:"b695d79d4f967c403a96986f1750a35eb75e75f1", GitTreeState:"clean", BuildDate:"2021-11-17T15:48:33Z", GoVersion:"go1.16.10", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.6-gke.1500", GitCommit:"7ce0f9f1939dfc1aee910732e84cba03840df91e", GitTreeState:"clean", BuildDate:"2021-11-17T09:30:26Z", GoVersion:"go1.16.9b7", Compiler:"gc", Platform:"linux/amd64"}
```

remove gke cluster

```shell
gcloud container clusters delete k8s --zone asia-northeast1-a
```

result

```text
The following clusters will be deleted.
 - [k8s] in [asia-northeast1-a]

Do you want to continue (Y/n)?  y

Deleting cluster k8s...done.     
Deleted [https://container.googleapis.com/v1/projects/miyata080825/zones/asia-northeast1-a/clusters/k8s].
```
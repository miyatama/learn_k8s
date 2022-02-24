# outline

+ [docker installation](https://docs.docker.com/engine/install/debian/)
+ [kubectl installation](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)
+ [go installation](https://go.dev/doc/install)
+ [kind installation](https://kind.sigs.k8s.io/)

environment

+ GCP
  + VM Instance
    + Disk: 100[GB]

change sudoers

```shell
#Defaults   secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Defaults    env_keep += "PATH"
```

```shell
# docker setting
sudo gpasswd -a $(whoami) docker
sudo chgrp docker /var/run/docker.sock
sudo service docker restart

# yaml editor
sudo apt update
sudo apt install jq
sudo apt install snapd
sudo snap install core
sudo snap install yq
echo 'export PATH=$PATH:/snap/bin' >> ~/.bashrc

# kind command test
kind crate cluster
kind delete cluster
kind build node-image
kind crete cluster --image kindest/node:latest
```

## operation

create kind.yaml

```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
- role: control-plane
  image: kindest/node:v1.23.0
- role: control-plane
  image: kindest/node:v1.23.0
- role: control-plane
  image: kindest/node:v1.23.0
- role: worker
  image: kindest/node:v1.23.0
- role: worker
  image: kindest/node:v1.23.0
- role: worker
  image: kindest/node:v1.23.0
```

start cluster

```shell
sudo kind create cluster --config kind.yaml --name kindcluster
```

change context

```shell
sudo kubectl config use-context kind-kindcluster
```

show nodes

```shell
sudo kubectl get nodes
```

show containers.check ha proxy image exists.

```shell
docker container ls
```

remove cluster

```shell
sudo kind delete cluster --name kindcluster
```

## use alpha

create kind-alpha.yaml.see [configuration](https://kind.sigs.k8s.io/docs/user/configuration/)

```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
kubeadmConfigPatches:
- |-
  apiVersion: kind.x-k8s.io/v1beta2
  kind: ClusterConfiguration
  metadata:
    name: config
    apiServer:
      extraArgs:
        "feature-gates":
```

start cluster

```shell
sudo kind create cluster --config kind-alpha.yaml --name kind-alpha-cluster
```

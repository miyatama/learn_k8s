# outline

minikube from SIG-Cluster-Lifecycle

+ [docker installation](https://docs.docker.com/engine/install/debian/)
+ [kubectl installation](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)
+ [minikube installation](https://minikube.sigs.k8s.io/docs/start/)

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

# behavior

```shell
sudo usermod -aG docker $USER && newgrp docker
docker version --format {{.Server.Os}}-{{.Server.Version}}
minikube version
minikube start
# start with specific k8s version
# minikube start --kubernetes-version v1.18.2
```

```text
😄  minikube v1.24.0 on Debian 10.11 (amd64)
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.22.3 preload ...
    > preloaded-images-k8s-v13-v1...: 501.73 MiB / 501.73 MiB  100.00% 82.69 Mi
    > gcr.io/k8s-minikube/kicbase: 355.78 MiB / 355.78 MiB  100.00% 8.46 MiB p/
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.22.3 on Docker 20.10.8 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

check status

```shell
minikube status
```

```text
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

change context

```shell
kubectl config use-context minikube
```

check nodes

```shell
kubectl get nodes
```

remove minikube cluster

```shell
minikube delete
```
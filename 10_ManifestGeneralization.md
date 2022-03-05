# Manifest Generalization

purpose

+ reuse manifest yaml file
+ batch update

## Helm

Helm is package manager.like a DNF/Yum at RHEL, APT at Debian.
Helm calling package a Chart.

[installation](https://helm.sh/docs/intro/install/)

```shell
OS_TYPE=darwin
OS_TYPE=linux
VERSION=3.2.0

cd /tmp
curl -sL https://get.helm.sh/helm-v${VERSION}-${OS_TYPE}-amd64.tar.gz \
  -o /tmp/helm.tar.gz
tar fx helm.tar.gz
mv ${OS_TYPE}-amd64/helm /usr/local/bin/
chmod +x /usr/local/bin/helm
helm help
```

Chart has `stable` and `incubetor`.It's divided by maturity.
promote from incubetor to stable has condition.

+ updatable
+ eternal data
+ secured
+ has default
+ k8s best practice

### add repository

```shell
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo add bitnami https://charts.bitnami.com/bitnami
# added repositories
helm repo list
# updatae repository
helm repo update
```

### search chart

```shell
# search WordPress
helm search repo wordpress
```

search from Helm Hub

```shell
helm search hub wordpress
```

check settable parameters

```shell
helm show values bitnami/wordpress
```

### install chart

install with parameter

```shell
helm install sample-wordpress bitnami/wordpress \
  --set wordpressUsername=sample-user \
  --set wordpressPassword=sample-pass \
  --set persistence.size=5Gi
```

install with values file

```shell
cat << EOF > values.yaml
wordpressUsername: sample-user
wordpressPassword: sample-pass
wordpressBlogName: "Sample Blog"
persistence:
  size: 5Gi
EOF
helm install sample-wordpress bitnami/wordpress \
  --values values.yaml
```

and test

```shell
helm test sample-wordpress
```

### create manifest from template

helm template file has can be complicated.check manifest with `heml template`.
`helm template` is only output manifest files.not apply cluster.

```shell
helm template sample-wordpress bitnami/wordpress \
  --version 9.2.2 \
  --set wordpressUsername=sample-user \
  --set wordpressPassword=sample-pass \
  --set wordpressBlogName="Sample Blog" \
  --set persistence.size=5Gi
# or use values.yaml
helm template sample-wordpress bitnami/wordpress \
  --version 9.2.2 \
  --values values.yaml
```

### Helm Archtecture

helm client manages a Chart and values pairs.chart and values pair are called a Release.a Release saved to k8s Secret. 

```shell
kubectl get secret  -l owner=helm
# check a Release
helm list
```

uninstall release

```shell
helm uninstall sample-wordpress
```

### create original chart

chart created by manifest file and values.

```shell
helm create sample-chart
tree ./sample-chart
```

```text
```

| file | description |
| :----- | :----- |
| `templates/*.yaml` | template manifest for install |
| `templates/tests/*.yaml` | test template |
| `values.yaml` | setting values for user |
| `templates/NOTES.txt` | output messsage when install |
| `requirements.yaml` | need chart package |
| `templates/_helpers.tpl` | helper variables.|
| `Chart.yaml` | metadata |

ex) sample-chart/templates/service.yaml

```yaml
```

ex) sample-chart/values.yaml

```yaml
```

NOTES.txt output when install helm

```shell
helm install sample-helm sample-chart
```

dependency defined by requirements.yaml.

ex) wordpress/requirements.yaml

```yaml
dependencies:
  - name: mariaDB
    version: 7.x.x
    repository: https://cahrts.bitnami.com/bitnami
    condition: mariadb.enabled
    tags:
      - wordpress-database
```

need tarball when packaging and publish repository.

```shell
helm package --dependency-update ./sample-charts
helm repo index .
cat index.yaml
```

## Kustomize

Kustomize is manifest templating tool that provided by sig-cli fo kubernetes.

### join multi manifest

ex) resource-sample/kustomization.yaml

```yaml
resources:
- sample-deployment.yaml
- sample-lb.yaml
```

```shell
kubectl kustomize resource-sample/
```

### override namespace

ex) namespace-sample/kustomization.yaml

```yaml
namespace: sample-namespace
resource:
- sample-deployment.yaml
- sample-lb.yaml
```

```shell
kubectl kustomize namespace-sampl/
```

### add Prefix and Suffix

ex) name-sample/kustomization.yaml

```yaml
namePrefix: prefix-
nameSuffix: -suffix
resources:
- sample-deployment.yaml
- sample-lb.yaml
```

```shell
kubectl kustomize name-sample/
```

### add metadata(label and annotation)

ex) common-meta-sample/kustomization.yaml

```yaml
commonLabels:
  label1: label1-val
commonAnnotation:
  annotation1: annotation1-val
resources:
- sample-deployment.yaml
- sample-lb.yaml
```

```shell
kubectl kustomize common-meta-sample/
```

### override image

ex) image-sample/kustomization.yaml

```yaml
images:
  - name: nginx
    newName: amsy810/echo-nginx
    newTag: v2.0
resources:
- sample-deployment.yaml
- sample-lb.yaml
```

### override using Overray

override memory/cpu.

ex) production/kustomization.yaml

```yaml
bases:
- ../resources-sample/
patchStagingMerge:
- ./patch-replicas.yaml
images:
- name: nginx
  newTag: production
```

ex) production/patch-replicas.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
spec:
  replicas: 2
```

```shell
kubectl kustomize production
```

### create ConfigMap and Secret

ex) generate-sample/kustomization.yaml

```yaml
resources:
- sample-deployment.yaml
configMapGenerator:
- name: generated-configmap
  literals:
  - KEY=VAL1
  files:
  - ./sample.txt
```

```shell
kubectl kustomize generate-sample
```

### kubectl subcommand for kustomize

kubectl provide easy to kustomize using with `-k` option.

```shell
kubectl apply -k resoruce-sample/
kubectl get -k resource-sample/
kubectl describe -k resource-sample/
kubectl diff -k resource-sample/
```
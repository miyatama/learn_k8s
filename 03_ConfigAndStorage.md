# Config And Storage

| resource | description |
| :----- | :----- |
| Secret | |
| ConfigMap | |
| PersistentVolumeClaim | |

## Environment Variable

how to pass informationto pod

| approach | description |
| :----- | :----- |
| static setting | use `spec.containers[].env`. |
| pod information | use `spec.containers[].fieldRef` |
| container information | use `spec.containers[].resourceFieldRef` |

### static setting

use `spec.containers[].env`.

ex) sample-env.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-env
  labels:
    app: sample-app
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      env:
        - name: MAX_CONNECTION
          value: "100"
```

```shell
kubectl apply -f sample-env.yaml --dry-run=server
kubectl apply -f sample-env.yaml
kubectl exec \
  -it \
  sample-env \
  -- env | grep MAX_CONNECTION
kubectl delete pod sample-env
```

### pod information

use fieldRef.

```shell
kubectl get pod sample-pod -o yaml
```

```text
```

ex) sample-env-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-env-pod
  labels:
    app: sample-app
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      env: 
        - name: K8S_NODE
          valueFrom:
            fieldRef: 
              fieldPath: spec.nodeName
```

```shell
kubectl apply -f sample-env-pod.yaml
kubectl get pod sample-env-pod
kubectl exec \
  -it \
  sample-env-pod \
  -- env | grep K8S_NODE
kubectl delete pod sample-env-pod
```

### container information

use `resourceFieldRef`

```shell
kubectl get pod sample-pod -o yaml
```

ex) sample-env-container.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-env-container
  labels:
    app: sample-app
spec:
  containers:
    - name: nginx-contianer
      image: nginx:1.16
      env:
        - name: CPU_REQUESTS
          valueFrom:
            resourceFieldRef:
              containerName: nginx-container
              resource: requests.cpu
        - name: CPU_LIMITS
          valueFrom:
            resourceFieldRef:
              containerName: nginx-container
              resourdce: limit.cpu
```

```shell
kubectl apply -f sample-env-container.yaml
kubectl get pod sample-env-container
kubectl exec \
  -it \
  sample-env-container \
  -- env | grep K8S_NODE
```

### impotant point

environment variable are not read in the manifest bellow.

ex) sample-env-fail.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-env-fail
  labels:
    app: sample-app
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      command: ["echo"]
      args: ["${TESTENV}", "${HOSTNAME}"]
      env:
        - name: TESTENV
          value: "100"
```

```shell
kubectl apply -f sample-env-fail.yaml
kubectl logs sample-env-fail
```

```text
```

can not read environment variable in command and args fieled except declare env filed use `$()`.

ex) sample-env-fail2.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-env-fail
  labels:
    app: sample-app
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      command: ["echo"]
      args: ["$(TESTENV)", "$(HOSTNAME)"]
      env:
        - name: TESTENV
          value: "100"
```

```shell
kubectl apply -f sample-env-fail2.yaml
kubectl logs sample-env-fail
```

```text
```

use entrypoint.sh when need reading environment variables. 

## Secret

wanna embed information(ex: db user and password).

+ embed docker image(Not Recommended)
+ pod or deplooyment manifest(Not Recommended)
+ use secret

secret have some type.

| type | description |
| :----- | :----- |
| Opaque | general |
| kubernetes.io/tls | TLS Certificate |
| kubernetes.io/basic-auth | Basic Certificate |
| kubernetes.io/dockerconfigjson | Docker registory certificate |
| kubernetes.io/ssh-auth | SSH Certifidate |
| kubernetes.io/service-account-token | Service Account Certificate |
| bootstrap.kubernetes.io/token | Bootstrap token Certificate |

Secret have some Key-Value.storable size is 1MB.

### Opaque

create resource 4 pattern.

| approach | description |
| :---- | :----- |
| --from-file | file reference |
| --from-env-file | envfile reference |
| --from-literal | direct value |
| -f | from manifest |

`--from-file`

```shell
mkdir -p secret/files
echo -n "root" > ./secret/files/username
echo -n "rootpassword" > ./secret/files/password
# username=`cat ./secret/files/username`
# password=`cat ./secret/files/password`
kubectl create secret generic \
  --save-config sample-db-auth \
  --from-file=./secret/files/username \
  --from-file=./secret/files/password
kubectl get secret sample-db-auth -o json | jq .data
kubectl get secret sample-db-auth -o json | jq .data.username
kubectl get secret sample-db-auth -o json | \
  jq .data.username |
  base64 --decode
```

`--from-env-file`

```shell
cat << EOF > ./secret/files/env-secret.txt
username=root
password=rootpassword
EOF
kubectl create secret generic \
  --save-config sampe-db-auth \
  --from-env-file ./secret/files/env-secret.txt
```

`--from-literal`

```shell
kubectl create generic \
  --save-config sample-db-auth \
  --from-literal=username=root \
  --from-literal=password=rootpassword
```

`-f`

ex) sample-db-auth.yaml

```shell
echo "root" | base64
echo "rootpassword" | base64
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sample-db-auth
type: Opaque
data:
  username: cm9vA==
  password: cedfjl...
```

```shell
kubectl apply -f sample-db-auth.yaml
```

ex) sample-db-auth-nonbase64.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sample-db-auth
type: Opaque
stringData:
  username: root
  password: rootpassword
```

```shell
kubectl apply -f sample-db-auth.yaml
```

### TLS type Secret

```shell
openssl req \
  -x509 \
  -nodes \
  -days 3650 \
  -newkey rsa:2048 \
  -keyout ~/tls.key \
  -out ~/tls.crt \
  -subj "/CN=sample1.example.com"
kubectl create secret tls \
  --save-config tls-sample \
  --key ~/tls.key \
  --cert ~/tls.crt
```

### docker registry secret

```shell
kubectl create secret docker-registry \
  --save-config sample-registry-auth \
  --docker-server=REGISTRY_SERVER \
  --docker-username=REGISTRY_USER \
  --docker-password=REGISTRY_PASSWORD \
  --docker-email=REGISTRY_USER_EMAIL
kubectl get secrets -o json sample-registry-auth |
  jq '.data'
```

```text
```

```shell
kubectl get secret sample-registry-auth -o yaml | \
  grep "\.dockerconfigjson" | \
  awk -F' ' '{print $2}' | \
  base64 --decode
```

ex) sample-pull-secret.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pull-secret
spec:
  containers:
    - name: secret-image-container
      image: REGISTRY_NAME/scret-image:latest
  imagePullScrets:
    - name: sample-registry-auth
```

### basic auth

create from command

```shell
kubectl create secret generic \
  --save-config sample-basic-auth \
  --type kuberunetes.io/basic-auth \
  --from-literal=username=root \
  --from-literal=password=rootpassword
```

create from manifest

ex) sample-basic-auth.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sample-basic-auth
type: kubernetes.io/basic-auth
data:
  username: cm9vA==
  password: cedfjl...
```

```shell
kubectl apply -f sample-basic-auth.yaml
```

### ssh auth

`--from-file`

```shell
ssh-keygen -t rsa -b 2048 -f sample-key -C "sample"
kubectl create secret generic \
  --save-config sample-ssh-auth \
  --type kuberunetes.io/ssh-auth \
  --from-file=ssh-privatekey=./sampple-key
```

`-f`

ex) sample-ssh-auth.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sample-ssh-auth
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS...
```

### How to use Secret

2 pattern

+ environment variable
+ valume mount

#### environment variable

ex) sample-secret-single-env.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-secret-single-env
spec:
  containers:
    - name: secret-container
      image: nginx:1.16
      env:
      - name: DB_USERNAME
        valueFrom:
          secretKeyRef:
            name: sample-db-auth
            key: username
```

```shell
kubectl apply -f sample-secret-single-env.yaml
kubectl exec -it sample-secret-single-env -- env | grep DB_USERNAME
```

all value

ex) sample-secret-multi-env.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-secret-multi-env
spec:
  containers:
    - name: secret-container
      image: nginx:1.16
      envFrom:
      - secretRef:
          name: sample-db-auth
```

```shell
kubectl apply -f sample-secret-multi-env.yaml
kubectl exec -it sample-secret-multi-env -- env
```

add prefix

ex) sample-secret-multi-env-prefix.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-secrete-multi-env-prefix
spec:
  container:
    - name: secret-container
      image: nginx:1.16
      envFrom:
      - secretRef:
          name: sample-db-auth
        prefix: DB1_
      - secretRef:
          name: sample-db-auth
        prefix: DB2_
```

```shell
kubectl apply -f sample-secret-multi-env-prefix.yaml
kubectl exec -it sample-secret-multi-env-prefix -- env
```

#### volume mount

use `spec.volumes[].secret.items[]`

ex) sample-secret-single-volume.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-secret-single-volume
spec:
  containers:
    - name: secret-container
      image: nginx:1.16
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      secret: 
        secretName: sample-db-auth
        items:
        - key: username
          path: username.txt
```

```shell
kubectl apply -f sample-secret-single-volume.yaml
kubectl exec \
  -it \
  sample-secret-single-volume \
  -- cat /config/username.txt
```

all key mount

ex) sample-secret-multi-volume.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-secret-multi-volume
spec:
  containers:
    - name: secret-container
      image: nginx:1.16
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      secret: 
        secretName: sample-db-auth
```

```shell
kubectl apply -f sample-secret-multi-volume.yaml
kubectl exec \
  -it \
  sample-secret-multi-volume \
  -- ls /config/
```

refresh secret span is determined by kubelet `--sync-frequency` option.

### mount with permission

ex) sample-secret-scripts.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-secret-scripts
spec:
  containers:
    - name: secret-container
      image: nginx:1.16
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      secret:
        secretName: sample-db-auth
        defaultMode: 256 # 0400
```

```shell
kubectl apply -f sample-secret-scripts.yaml
kubectl exec -it sample-secret-scripts -- ls -la /config/
kubectl exec -it sample-secret-scripts -- ls -la /config/..data/
```

## ConfigMap

### create

3 approach

+ `--from-file`: create from file
+ `--from-literal`: create from direct value
+ `-f`: create from manifest

`--from-file`

```shell
kubectl create configmap \
  --save-config sample-configmap \
  --from-file=./nginx.conf
kubectl get configmap sample-configmap -o json | jq '.data'
```

```text
```

```shell
kubectl describe configmap sample-configmap
```

```text
```

wanna save binary data using `binaryData` filed.

```shell
kubectl create configmap smaple-configmap-binary \
   --from-file image.jpg \
   --from-literal=inde.html="Hello,Kubernetes" \
   --dry-run=client -o yaml > sample-configmap-binary.yaml
cat sample-configmap-binary.yaml
```

```yaml
```

ex) sample-configmap-binary-webserver.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-configmap-binary-webserver
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
      - name: config-volume
        mountPath: /usr/share/nginx/html
  volumes:
    - name: config-volume
      configMap:
        name: sample-configmap-binary
```

```shell
kubectl apply -f sample-configmap-binary-webserver.yaml
kubectl port-forward sample-configmap-binary-webserver 8080:80
curl -OL http://localhost:8080/image.jpg
```

use `--from-literal`

```shell
kubectl create configmap \
  --save-config web-config \
  --from-literal=connection.max=100 \
  --from-literal=connection.min=10
```

use `-f`

ex) sample-configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sample-configmap
data:
  thread: "16"
  connection.max: "100"
  connecion.min: "10"
  sample.properties: |
    property.1=value-1
    property.2=value-2
    property.3=value-3
  nginx.config: |
    user nginx;
    worker_processes auto;
    error_log /var/log/nginx/error.log;
    pid /run/nginx.pid;
  test.sh: |-
    #!/bin/bash
    echo "Hello, kubernetes"
    sleep infinity
```

```shell
kubectl apply -f sample-configmap.yaml
```

### use ConfigMap

2 approach

+ Environment Variables
+ Volume mount

#### Environment Variables

use `spec.containers[].env.valueFrom.configMapMapKeyRef`

ex) sample-configmap-single-env.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-configmap-single-env
spec:
  containers:
    - name: configmap-container
      image: nginx:1.16
      env:
        - name: CONNECTION_MAX
          valueFrom:
            configMapKeyRef:
              name: sample-configmap
              key: connection.max
```

```shell
kubectl apply -f sample-configmap-single-env.yaml
kubectl exec -it sample-config-single-env -- env | grep -i connection_max
kubectl delete pod sample-configmap-single-env
```

ex) sample-configmap-multi-env.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-configmap-multi-env
spec:
  containers:
    - name: configmap-container
      image: nginx:1.16
      envFrom:
        - configMapRef:
            name: sample-configmap
```

```shell
kubectl apply -f sample-configmap-multi-env.yaml
kubectl exec -it sample-configmap-multi-env -- env
kubectl delete pod sample-configmap-multi-env
```

#### Volume mount

use `spec.volumes[].configMap.items[]`

ex) sample-configmap-single-volume.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-configmap-sinble-volume
spec:
  containers:
    - name: configmap-container
      image: nginx:1.16
      volumeMounts:
      - name: config-volume
        mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: sample-configmap
      items:
      - key: nginx.config
        path: nginx-sample.conf
```

```shell
kubectl apply -f sample-configmap-single-volume.yaml
kubectl exec -it sample-configmap-single-volume -- cat /config/nginx-sample.conf
kubectl delete pod sample-configmap-single-volume
```

ex) sample-configmap-multi-volume.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-configmap-multi-volume
spec:
  containers:
    - name: configmap-container
      image: nginx:1.16
      volumeMounts:
      - name: config-volume
        mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: sample-configmap
```

```shell
kubectl apply -f sample-configmap-multi-volume.yaml
kubectl exec -it sample-configmap-multi-volume -- ls /config/
kubectl delete pod sample-configmap-multi-volume
```

### mount with permission

ex) sample-configmap-scripts.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-configmap-scripts
spec:
  containers:
    - name: configmap-container
      image: nginx:1.16
      command: ["/config/test.sh"]
      volumeMounts:
      - name: config-volume
        mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: sample-configmap
      items:
      - key: test.sh
        path: test.sh
        mode: 493 # 0755
```

```shell
kubectl apply -f sample-configmap-scripts.yaml
kubectl logs sample-configmap-scripts
```

## Volume

volume provide plugin

+ emptyDir
+ hostPath
+ downwardAPI
+ projected
+ nfs
+ iscsi
+ cephfs

### emptyDir

pod temporary directory.remove when pod terminated.

ex) sample-emptydir.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-emptydir
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - name: cache-volume
          mountPath: /cache
  volumes:
    - name: cache-volume
      emptyDir: {}
          
```

```shell
kubectl apply -f sample-emptydir.yaml
kubectl exec -it sample-emptydir -- df -h | grep /cache
```

can limitage

ex) sample-emptydir-limit.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-emptydir-limit
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - name: cache-volume
          mountPath: /cache
  volumes:
    - name: cache-volume
      emptyDir: 
        sizeLimit: 128Mi
```

```shell
kubectl apply -f sample-emptydir-limit.yaml
kubectl get pods --watch
```

```shell
kubectl exec -it sample-emptydir-limit -- dd if=/dev/zero of=/cache/dummy bs=1M count=150
```

usable memory.check `resource.limits.memory`.

ex) sample-emptydir-memory.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-emptydir-memory
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - name: cache-volume
          mountPath: /cache
  volumes:
    - name: cache-volume
      emptyDir:
        medium: Memory
        sizeLimit: 128Mi
```

```shell
kubectl apply -f sample-emptydir-memory.yaml
kubectl exec -it sample-emptydir-memory -- df -h | grep /cache
kubectl get pods --watch
```

```shell
# default 70MB limit
# same issue occuerd when count=70
kubectl exec -it sample-emptydir-limit -- dd if=/dev/zero of=/cache/dummy bs=1M count=150
```

### hostPath

hostPath provide any resource on host.have any type.

+ Directory
+ DirectoryOrCreate
+ File
+ Socket
+ BlockDevice

ex) sample-hostpath.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-hostpath
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - name: hostpath-sample
          path: /srv
  volumes:
    - name: hostpath-sample
      hostPath: 
        path: /etc
        type: DirectoryOnCreate
```

```shell
kubectl apply -f sample-hostpath.yaml
kubectl exec -it sample-hostpath -- cat /srv/os-release
```

### downwardAPI

take pod information as file.

ex) sample-downward-api.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-downward-api
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - name: downward-api-volume
          mountPath: /srv
  volumes:
    - name: downward-api-volume
      downwardAPI:
        items:
          - path: "podname"
            fieldRef:
              fieldPath: metadata.name
          - path: "cpu-request"
            resourceFieldRef:
              containerName: nginx-container
              resource: requests.cpu
```

```shell
kubectl apply -f sample-downward-api.yaml
kubectl exec -it sample-downward-api -- ls /srv
kubectl exec -it sample-downward-api -- cat /src/requests-cpu
```

### projected

projected together a Secret/ConfigMap/downwardAPI/serviceAccountToken in one place.

ex) sample-projected.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-projected
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - name: projected-volume
          mountPath: /srv
  volumes:
    - name: projected-volume
      projected:
        sources:
          - secret:
            name: sample-db-auth
            items:
              - key: username
                path: secret/username.txt
          - configMap:
            name: sample-configmap
            items:
              - key: nginx.config
                path: configmap/nginx.config
          - downwardAPI:
            items:
              - path: "podname"
                fieldRef:
                  fieldPath: metadata.name
```

```shell
kubectl apply -f sample-projected.yaml
kubgecl exec -it sample-projected -- ls /srv
kubectl exec -it sample-projected -- cat /srv/secret/username.txt
kubectl exec -it sample-projected -- cat /srv/configmap/nginx.config
```

## PersistentVolume

strictly, Cluster resource.

### create

field

+ label
+ size
+ access mode
+ reclaim policy
+ mount option
+ storage class
+ plugin settings

ex) sample-gce-persistent-disk.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: sample-pv
  labels:
    type: gce-pv
    environment: stg
spec:
  capacity: 
    storage: 10Gi
  accessMode:
  - ReadWriteOnce
  persistentVolemeReclaimPolicy: Retain
  storageClassName: menual
  gcpPersistentDisk:
    pdName: sample-gcp-pv
    fsType: ext4
```

```shell
# error
kubectl apply -f sample-gce-persistent-disk.yaml
```

need Parsistent Disk on GCE.

```shell
gcloud compute disks create \
  --size=10GB \
  sample-gce-pv \
  --zone asia-northeast1-a
kubectl apply -f sample-gce-persistent-disk.yaml
kubectl get persistentvolumes
```

recommended

+ volume
  + type: nfs, scsi
  + environment: gce
  + speed: slow, high
+ access mode
  + ReadWriteOnce(RWO): allow read/write by one node
  + ReadOnlyMany(ROX): allow read only by nodes
  + ReadWriteMany(RWM): allow read/write by nodes
+ ReclaimPolicy: behavior after use
  + Delete
  + Retain
  + Recycle(Not recommended)
+ StorageClass
  + deny auto serve PersistentVolume

StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: manual
provisioner: kubernetes.io/no-provisioner
```

## PersistentVolumeClaim

PersistentVolumeClaim is resource fro eternal file.

| resource | description |
| :----- | :----- |
| Volume | use pre-advance resource(host space, NFS, iSCSI, Ceph).can not create new, delete on kubernetes |
| PersistentVolume | create new volume |

PersistentVolumeClaim is assign resource for PersistentVolume.

### settings

need setting

+ Label Selector
+ Storage Size
+ Access Mode
+ StorageClass

ex) sample-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sample-pvc
spec:
  selector:
    matchLabels:
      type: gcp-pv
    matchExpressions:
      - key: environment
        operator: In
        values: 
          - stg
  resources:
    requests:
      storage: 3Gi
  accessMode:
    - ReadWriteOnce
  storageClassName: manual
```

```shell
kubectl apply -f sample-pvc.yaml
kubectl get persistentvolumeclaims
kubectl get persistentvolumeclaim sample-pvc
kubectl get persistentvolume
```

### use by pod

ex) sample-pvc-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pvc-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-pvc
  volumes:
  - name: nginx-pvc
    persistentVolumeClaim:
      claimName: sample-pvc
```

```shell
kubectl apply -f sample-pvc-pod.yaml
```

### Dynamic Provisioning

dynamic persistent volume assign!very cool!
to use dynamic provisioning,need to use StorageClass.

ex) sample-storageclass.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sample-storageclass
parameters:
  type: pd-ssd
provisioner: kubernetes.io/gce-pd
reclaimPolicy: Delete
```

```shell
kubectl apply -f sample-storageclass.yaml
```

create PersistentVolumeClaim

ex) sample-pvc-dynamic.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sample-pvc-dynamic
spec:
  storageClassName: sample-storageclass
  accessMode: 
  - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```shell
kubectl apply -f sample-pvc-dynamic.yaml
kubectl get persistentvolumeclaims
kubectl get persistentvolumeclaim sample-pvc-dynamic
kubectl get persistentvolumes
```

GCP-PD Provisioner

ex) sample-pvc-dynamic-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pvc-dynamic-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-pvc
  volumes:
    - name: nginx-pvc
      persistentVolumeClaim:
        claimName: sample-pvc-dynamic
```

```shell
kubectl apply -f sample-pvc-dynamic-pod.yaml
```

### timing control

create PersistentVolume timing same as created PersistentVolumeClaim Resource using Dynamic Provisioning.
want to prohibit it, use volumeBindingMode.

| volumeBindingMode | description |
| :----- | :----- |
| Immediate | default.created PersistentVolumeClaim Resource using Dynamic Provisioning |
| WaitForFirstConsumer | created pod |

the plugin available for waitForFirstConsumer are limited.

+ GCE Persistent Disk
+ AWS Elastic Block Store
+ Azure Disk
+ Local
+ Consumer Storage Interface

ex) sample-strageclass-wait.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sample-strageclass-wait
parameters:
  type: pd-ssd
provisioner: kubernetes.io/gce-pd
reclaimPolicy: Delete
volumeBindingMode: waitForFirstConsumer
```

```shell
kubectl apply -f sample-strageclass-wait.yaml
kubectl get persistentvolumeclaims
kubectl get persistentvolumeclaim sample-pvc-dynamic
kubectl apply -f sample-pvc-wait-pod.yaml
```

### GCE StorageClass

```shell
kubectl get storageclasses standard -o yaml
```

+ type
  + pd-standard
  + pd-ssd
+ replication-type
  + none
  + regional-pd
+ zone
  + any zone
+ zones
  + some zones

### block device

set `Block` to `spec.volumeMode`.default value is `FileSystem`.

ex) sample-pvc-block.yaml

```yaml
apiVersion: v1
kiind: PersistentVolumeClaim
metadata:
  name: sample-pvc-block
spec:
  storageClassName: sample-storageclass
  volumeMode: Block
  accessMode: 
  - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```shell
kubectl apply -f sample-pvc-block.yaml
```

ex) sample-block-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-block-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeDevices:
         - name: nginx-pvc
           devicePath: /dev/sample-block
  volumes:
  - name: nginx-pvc
    persistentVolumeClaim:
      className: sample-pvc-block
```

```shell
kubectl apply -f sample-block-pod.yaml
```

### volume resize

usable resize when use Dynamic Provisioning and provided resize.
use `allowVolumeExpantion`.

providable resize plugin

+ gcpPersistentDisk
+ awsElasticBlockStore
+ OpenStack Cinder
+ glusterfs
+ RDB

ex) sample-storageclass-resize.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sample-storageclass-resize
parameters:
  type: pd-ssd
provisioner: kubernetes.io/gce-pd
reclaimPolicy: Delete
allowVolumeExpantion: true
```

ex) sample-pvc-resize.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sample-pvc-resize
spec:
  accessMode:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: sample-storageclass-resize
```

ex) sample-pvc-resize-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pvc-resize-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16 
      volumeMounts:
        - mountPath: "/usr/share/nginx/home"
          name: nginx-pvc
  volumes:
     - name: nginx-pvc
       persistentVolumeClaim:
         claimName: sample-pvc-resize
```

```shell
kubectl apply -f sample-storageclass-resize.yaml
kubectl apply -f sample-pvc-resize.yaml
kubectl apply -f sample-pvc-resize-pod.yaml

kubectl exec -it sample-pvcc-resize-pod -- df -h | grep /usr/share/nginx/html

kubectl patch pvc sample-pvc-resize \
  --patch '{"spec":{"resources": {"requests": {"storage": "16Gi"}}}}'

# can not reduce size
kubectl patch pvc sample-pvc-resize \
  --patch '{"spec":{"resources": {"requests": {"storage": "1Gi"}}}}'

kubectl exec -it sample-pvcc-resize-pod -- df -h | grep /usr/share/nginx/html
```

### with StatfulSet

ex) sample-statefulset-with-pvc.yaml

```yaml
apiVersion: v1
kind: StatefulSet
metadata:
  name: sample-statefulset-with-pvc
spec:
  serviceName: stateful-with-pvc
  replicas: 2
  selector: 
    matchLabels:
      app: sample-pvc
  template:
    metadata:
      labels:
        app: sample-pvc
    spec:
      containers:
        - name: nginx-contanier
          image: nginx:1.16
          volumeMounts:
            - name: pvc-template-volume
              mountPath: /tmp
  volumeClaimTemplate:
    - metadata:
        name: pvc-template-volume
      spec:
        accessMode:
        - ReadWriteOnce
        resources:
          requests:
            storage: 10Gi
        storageClassName: sample-storageclass
```

```shell
kubectl apply -f sample-statefulset-with-pvc.yaml
```

## volumeMounts Option

volumeMounts have any options

+ ReadOnly
+ subPath

ex) sample-readonly-volumemount.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-readonly-volumemouint
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16
      volumeMounts:
        - name: hostpath-sample
          mountPath: /srv
          readOnly: true
  volumes:
    - name: hostpath-sample
      hostPath: 
        path: /etc
        type: DirectoryOrCreate
```

```shell
kubectl apply -f sample-readonly-volumemount.yaml
kubectl exec -it sample-readonly-volumemount -- touch /srv/motd
kubectl exec -it sample-readonly-volumemount -- cat /srv/motd
```

ex) sample-subpath-volumemount.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-subpath-volumemount
spec:
  containers:
    - name: container-a
      image: alpine:3.7
      command: ["sh", "-c", "touch", "/data/a.txt", "sleep 86400"]
      volumeMounts:
        - name: main-volume
          mountPath: /data
    - name: container-b
      image: alpine:3.7
      command: ["sh", "-c", "touch", "/data/b.txt", "sleep 86400"]
      volumeMounts:
        - name: main-volume
          mountPath: /data
          subPath: path1
    - name: container-c
      image: alpine:3.7
      command: ["sh", "-c", "touch", "/data/c.txt", "sleep 86400"]
      volumeMounts:
        - name: main-volume
          mountPath: /data
          subPath: path2
volumes:
  - name: main-volume
    emptyDir: {}
```

```shell
kubectl apply -f sample-subpath-volumemount.yaml
kubectl exec -it sample-subpath-volumemount -c container-b -- find /data
kubectl exec -it sample-subpath-volumemount -c container-c -- find /data
kubectl exec -it sample-subpath-volumemount -c container-a -- find /data
```

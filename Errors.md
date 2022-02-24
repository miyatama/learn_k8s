# Manifest

## error validating "...yaml": error validating data: [ValidationError(..): unknown field

approach: check yaml item depth and item spels

```shell
sudo kubectl apply -f sample.yaml --dry-run=server
```

<details><summary>Error Message</summary><div>

```text
error: error validating "sample-job-multiworkqueue.yaml": error validating data: [ValidationError(Job.spec.template): unknown field "containers" in io.k8s.api.core.v1.PodTemplateSpec, ValidationError(Job.spec.template): unknown field "restartPolicy" in io.k8s.api.core.v1.PodTemplateSpec]; if you choose to ignore these errors, turn validation off with --validate=false
```

</div></details>

# Pod

## pod pending always

approach: add memory and cpu

see [Debug Pod and ReplicaSets](https://kubernetes.io/ja/docs/tasks/debug-application-cluster/debug-pod-replication-controller/).

```shell
podname=`kubectl get pods | tail -n 1 | awk '{print $1}'`
kubectl describe pod ${podname}
```

```text
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  24s (x3 over 2m33s)  default-scheduler  0/4 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 3 node(s) had taint {node.kubernetes.io/not-ready: }, that the pod didn't tolerate.
```

# Job

## error: timed out waiting for the condition

approach: check image exists.`docker pull amsy/tools:v2.0` is failure.

<details><summary>Error Message</summary><div>

```shell
kubectl run testpod \
  --image=amsy/tools:v2.0 \
  --restart=Never \
  --rm \
  -i \
  --wait \
  --command -- curl -m 2 "http://${endpoint}:8080"
# pod "testpod" deleted
# error: timed out waiting for the condition
```

</div></details>


## spec.template: Invalid value ... field is immutable

approach: delete job

```shell
sudo kubectl delete job ...
```

<details><summary>Error message</summary><div>

```text
The Job "sample-job-never" is invalid: 
spec.template: Invalid value: 
core.PodTemplateSpec{
    ObjectMeta:v1.ObjectMeta{
        Name:"", 
        GenerateName:"", 
        Namespace:"", 
        SelfLink:"", 
        UID:"", 
        ResourceVersion:"", 
        Generation:0, 
        CreationTimestamp:v1.Time{
            Time:time.Time{
                wall:0x0, 
                ext:0, 
                loc:(*time.Location)(nil)
            }
        }, 
        DeletionTimestamp:(*v1.Time)(nil), 
        DeletionGracePeriodSeconds:(*int64)(nil), 
        Labels:map[string]string{
            "controller-uid":"15226f22-b492-473f-b893-05af3a5b894d", 
            "job-name":"sample-job-never"
        }, 
        Annotations:map[string]string(nil), 
        OwnerReferences:[]v1.OwnerReference(nil), 
        Finalizers:[]string(nil), 
        ClusterName:"", 
        ManagedFields:[]v1.ManagedFieldsEntry(nil)
    }, 
    Spec:core.PodSpec{
        Volumes:[]core.Volume(nil), 
        InitContainers:[]core.Container(nil), 
        Containers:[]core.Container{
            core.Container{
                Name:"tools-container", 
                Image:"amsy810/tools:v2.0", 
                Command:[]string{"sh", "-c"}, 
                Args:[]string{"$(sleep 3600)"}, 
                WorkingDir:"", 
                Ports:[]core.ContainerPort(nil), 
                EnvFrom:[]core.EnvFromSource(nil), 
                Env:[]core.EnvVar(nil), 
                Resources:core.ResourceRequirements{
                    Limits:core.ResourceList(nil), 
                    Requests:core.ResourceList(nil)
                }, 
                VolumeMounts:[]core.VolumeMount(nil), 
                VolumeDevices:[]core.VolumeDevice(nil), 
                LivenessProbe:(*core.Probe)(nil), 
                ReadinessProbe:(*core.Probe)(nil), 
                StartupProbe:(*core.Probe)(nil), 
                Lifecycle:(*core.Lifecycle)(nil), 
                TerminationMessagePath:"/dev/termination-log", 
                TerminationMessagePolicy:"File", 
                ImagePullPolicy:"IfNotPresent", 
                SecurityContext:(*core.SecurityContext)(nil), 
                Stdin:false, 
                StdinOnce:false, 
                TTY:false
            }
        }, 
        EphemeralContainers:[]core.EphemeralContainer(nil), 
        RestartPolicy:"Never", 
        TerminationGracePeriodSeconds:(*int64)(0xc00cc02458), 
        ActiveDeadlineSeconds:(*int64)(nil), 
        DNSPolicy:"ClusterFirst", 
        NodeSelector:map[string]string(nil), 
        ServiceAccountName:"", 
        AutomountServiceAccountToken:(*bool)(nil), 
        NodeName:"", 
        SecurityContext:(*core.PodSecurityContext)(0xc004e30a00), 
        ImagePullSecrets:[]core.LocalObjectReference(nil), 
        Hostname:"", 
        Subdomain:"", 
        Affinity:(*core.Affinity)(nil), 
        SchedulerName:"default-scheduler", 
        Tolerations:[]core.Toleration(nil), 
        HostAliases:[]core.HostAlias(nil), 
        PriorityClassName:"", 
        Priority:(*int32)(nil), 
        PreemptionPolicy:(*core.PreemptionPolicy)(nil), 
        DNSConfig:(*core.PodDNSConfig)(nil), 
        ReadinessGates:[]core.PodReadinessGate(nil), 
        RuntimeClassName:(*string)(nil), 
        Overhead:core.ResourceList(nil), 
        EnableServiceLinks:(*bool)(nil), 
        TopologySpreadConstraints:[]core.TopologySpreadConstraint(nil)
    }
}: field is immutable
```

</div></details>

# CronJob

## error: unknown object type *v1beta1.CronJob



<details><summary>Error Message</summary><div>

```text
error: unknown object type *v1beta1.CronJob
```

</div></details>

# kubeadm

## It seems like the kubelet isn't running or healthy.

approach: check swap disabled

```shell
sudo swapon -s
sudo swapoff -a
```

approach: change docker daemon cgroup driver.

```shell
sudo cat << EOF > /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

<details><summary>error message</summary><div>

```text
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

        Unfortunately, an error has occurred:
                timed out waiting for the condition

        This error is likely caused by:
                - The kubelet is not running
                - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

        If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                - 'systemctl status kubelet'
                - 'journalctl -xeu kubelet'

        Additionally, a control plane component may have crashed or exited when started by the container runtime.
        To troubleshoot, list all containers using your preferred container runtimes CLI.

        Here is one example how you may list all Kubernetes containers running in docker:
                - 'docker ps -a | grep kube | grep -v pause'
                Once you have found the failing container, you can inspect its logs with:
                - 'docker logs CONTAINERID'

error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher
```

</div></details>

## error execution phase preflight

approach: `kubeadm reset`

<details><summary>Error Message</summary><div>

```text
[init] Using Kubernetes version: v1.23.1
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: missing optional cgroups: hugetlb
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists
        [ERROR FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already exists
        [ERROR FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml]: /etc/kubernetes/manifests/kube-scheduler.yaml already exists
        [ERROR FileAvailable--etc-kubernetes-manifests-etcd.yaml]: /etc/kubernetes/manifests/etcd.yaml already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

</div></details>

## The connection to the server localhost:8080 was refused - did you specify the right host or port?

approach: check environment variable `KUBECONFIG` and exists file.

approach: check `kubectl config view` status.when cluster null then can not load config.remove `sudo`.

<details><summary>Error Message</summary><div>

```text
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

</div></details>

## could not find a JWS signature in the cluster-info ConfigMap for token ID

```text
error execution phase preflight: couldn't validate the identity of the API Server: could not find a JWS signature in the cluster-info ConfigMap for token ID "9utqxl"
To see the stack trace of this error execute with --v=5 or higher
```

# Service

## provided IP is not in the valid range.

approach: change ip address range.

<details><summary>Error Message</summary><div>

```text
The Service "sample-clusterip-vip" is invalid: spec.clusterIPs: Invalid value: []string{"10.11.253.81"}: failed to allocate IP 10.11.253.81: provided IP is not in the valid range. The range of valid IPs is 10.96.0.0/12
```

</div></details>

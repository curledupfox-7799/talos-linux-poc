# Provisioning with the Docker

## Check the provisioning environment
```
❯ talosctl version
Client:
        Tag:         v1.9.5
        SHA:         d07f6daa
        Built:
        Go version:  go1.23.7
        OS/Arch:     linux/amd64
Server:
error constructing client: failed to determine endpoints

❯ kubectl version
Client Version: v1.32.0
Kustomize Version: v5.5.0
The connection to the server localhost:8080 was refused - did you specify the right host or port?

❯ docker version
Client: Docker Engine - Community
 Version:           28.1.1
 API version:       1.49
 Go version:        go1.23.8
 Git commit:        4eba377
 Built:             Fri Apr 18 09:53:36 2025
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          28.1.1
  API version:      1.49 (minimum version 1.24)
  Go version:       go1.23.8
  Git commit:       01f442b
  Built:            Fri Apr 18 09:51:51 2025
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.27
  GitCommit:        05044ec0a9a75232cad458027ca83437aae3f4da
 runc:
  Version:          1.2.5
  GitCommit:        v1.2.5-0-g59923ef
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## Enable the kernel module
```
❯ sudo modprobe br_netfilter
```

## Create the cluster
```
❯ talosctl cluster create
validating CIDR and reserving IPs
generating PKI and tokens
creating state directory in "/home/vagrant/.talos/clusters/talos-default"
downloading ghcr.io/siderolabs/talos:v1.9.5
creating network talos-default
creating controlplane nodes
creating worker nodes
renamed talosconfig context "talos-default" -> "talos-default-1"
waiting for API
bootstrapping cluster
waiting for etcd to be healthy: OK
waiting for etcd members to be consistent across nodes: OK
waiting for etcd members to be control plane nodes: OK
waiting for apid to be ready: OK
waiting for all nodes memory sizes: OK
waiting for all nodes disk sizes: OK
waiting for no diagnostics: OK
waiting for kubelet to be healthy: OK
waiting for all nodes to finish boot sequence: OK
waiting for all k8s nodes to report: OK
waiting for all control plane static pods to be running: OK
waiting for all control plane components to be ready: OK
waiting for all k8s nodes to report ready: OK
waiting for kube-proxy to report ready: OK
waiting for coredns to report ready: OK
waiting for all k8s nodes to report schedulable: OK

merging kubeconfig into "/home/vagrant/.kube/config"
PROVISIONER           docker
NAME                  talos-default
NETWORK NAME          talos-default
NETWORK CIDR          10.5.0.0/24
NETWORK GATEWAY       10.5.0.1
NETWORK MTU           1500
KUBERNETES ENDPOINT   https://127.0.0.1:33077

NODES:

NAME                            TYPE           IP         CPU    RAM      DISK
/talos-default-controlplane-1   controlplane   10.5.0.2   2.00   2.1 GB   -
/talos-default-worker-1         worker         10.5.0.3   2.00   2.1 GB   -
```

## Check whether the cluster is created
```
❯ talosctl kubeconfig .kube/config --nodes 10.5.0.2

❯ export KUBECONFIG=.kube/config

❯ kubectl get nodes
NAME                           STATUS   ROLES           AGE     VERSION
talos-default-controlplane-1   Ready    control-plane   2m18s   v1.32.3
talos-default-worker-1         Ready    <none>          2m18s   v1.32.3

❯ kubectl get pods -A
NAMESPACE     NAME                                                   READY   STATUS    RESTARTS        AGE
kube-system   coredns-578d4f8ffc-sbgxb                               1/1     Running   0               2m34s
kube-system   coredns-578d4f8ffc-vpgmn                               1/1     Running   0               2m34s
kube-system   kube-apiserver-talos-default-controlplane-1            1/1     Running   0               2m24s
kube-system   kube-controller-manager-talos-default-controlplane-1   1/1     Running   2 (2m59s ago)   2m24s
kube-system   kube-flannel-8qjpc                                     1/1     Running   0               2m31s
kube-system   kube-flannel-khzsw                                     1/1     Running   0               2m31s
kube-system   kube-proxy-6jb56                                       1/1     Running   0               2m31s
kube-system   kube-proxy-9ltm9                                       1/1     Running   0               2m31s
kube-system   kube-scheduler-talos-default-controlplane-1            1/1     Running   2 (2m53s ago)   2m24s

❯ talosctl service --nodes 10.5.0.2
NODE       SERVICE      STATE     HEALTH   LAST CHANGE   LAST EVENT
10.5.0.2   apid         Running   OK       18m56s ago    Health check successful
10.5.0.2   containerd   Running   OK       18m58s ago    Health check successful
10.5.0.2   cri          Running   OK       18m56s ago    Health check successful
10.5.0.2   etcd         Running   OK       18m34s ago    Health check successful
10.5.0.2   kubelet      Running   OK       18m13s ago    Health check successful
10.5.0.2   machined     Running   OK       18m58s ago    Health check successful
10.5.0.2   trustd       Running   OK       18m56s ago    Health check successful

❯ talosctl service --nodes 10.5.0.3
NODE       SERVICE      STATE     HEALTH   LAST CHANGE   LAST EVENT
10.5.0.3   apid         Running   OK       19m1s ago     Health check successful
10.5.0.3   containerd   Running   OK       19m3s ago     Health check successful
10.5.0.3   cri          Running   OK       19m2s ago     Health check successful
10.5.0.3   kubelet      Running   OK       18m20s ago    Health check successful
10.5.0.3   machined     Running   OK       19m3s ago     Health check successful
```
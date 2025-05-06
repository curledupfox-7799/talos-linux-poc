# Provisioning with the Virtualbox

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
```

## Create the VMs
- Control plane and worker
  - CPU: 2
  - Memory: 4096MB
  - Disk: 20GB
  - Mount: metal-amd64.iso
  - Network: Bridged Adapter

## Setup the VMs
- Control plane
  - Hostname: talos-cp-01
  - DNS: 8.8.8.8
  - NTP: 210.173.160.27
  - IF: enp0s3
  - Mode: Static
  - IP: 192.168.1.101/24
  - GW: 192.168.1.1
- Worker
  - Hostname: talos-wk-01
  - DNS: Same controle plane
  - NTP: Same controle plane
  - IF: Same controle plane
  - Mode: Same controle plane
  - IP: 192.168.1.111/24
  - GW: Same controle plane

## Create the cluster
```
❯ talosctl gen config talos-vbox-cluster https://192.168.1.101:6443
generating PKI and tokens
Created /home/vagrant/ghq/github.com/curledupfox-7799/talos-linux-poc/virtualbox/controlplane.yaml
Created /home/vagrant/ghq/github.com/curledupfox-7799/talos-linux-poc/virtualbox/worker.yaml
Created /home/vagrant/ghq/github.com/curledupfox-7799/talos-linux-poc/virtualbox/talosconfig
```
```
❯ talosctl apply-config --insecure --nodes 192.168.1.101 --file controlplane.yaml
```
```
❯ talosctl apply-config --insecure --nodes 192.168.1.111 --file worker.yaml
```

## Check whether the cluster is created
```
❯ export TALOSCONFIG=./talosconfig
❯ talosctl --talosconfig $TALOSCONFIG bootstrap
❯ talosctl kubeconfig .kube/config

❯ export KUBECONFIG=.kube/config

❯ kubectl get nodes -o wide
NAME          STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION   CONTAINER-RUNTIME
talos-cp-01   Ready    control-plane   91s   v1.32.3   192.168.1.101   <none>        Talos (v1.9.5)   6.12.18-talos    containerd://2.0.3
talos-wk-01   Ready    <none>          76s   v1.32.3   192.168.1.111   <none>        Talos (v1.9.5)   6.12.18-talos    containerd://2.0.3

❯ kubectl get pods -A -o wide
NAMESPACE     NAME                                  READY   STATUS    RESTARTS        AGE     IP              NODE          NOMINATED NODE   READINESS GATES
kube-system   coredns-578d4f8ffc-7h5h8              1/1     Running   0               3m41s   10.244.1.2      talos-wk-01   <none>           <none>
kube-system   coredns-578d4f8ffc-lf4tj              1/1     Running   0               3m41s   10.244.1.3      talos-wk-01   <none>           <none>
kube-system   kube-apiserver-talos-cp-01            1/1     Running   0               2m39s   192.168.1.101   talos-cp-01   <none>           <none>
kube-system   kube-controller-manager-talos-cp-01   1/1     Running   1 (3m57s ago)   2m39s   192.168.1.101   talos-cp-01   <none>           <none>
kube-system   kube-flannel-75hj7                    1/1     Running   0               3m31s   192.168.1.101   talos-cp-01   <none>           <none>
kube-system   kube-flannel-rsl28                    1/1     Running   0               3m16s   192.168.1.111   talos-wk-01   <none>           <none>
kube-system   kube-proxy-ctc9r                      1/1     Running   0               3m16s   192.168.1.111   talos-wk-01   <none>           <none>
kube-system   kube-proxy-mmsgp                      1/1     Running   0               3m31s   192.168.1.101   talos-cp-01   <none>           <none>
kube-system   kube-scheduler-talos-cp-01            1/1     Running   1 (3m58s ago)   2m39s   192.168.1.101   talos-cp-01   <none>           <none>

❯ talosctl service --nodes 192.168.1.101
NODE            SERVICE      STATE     HEALTH   LAST CHANGE   LAST EVENT
192.168.1.101   apid         Running   OK       33m8s ago     Health check successful
192.168.1.101   auditd       Running   OK       6m49s ago     Health check successful
192.168.1.101   containerd   Running   OK       6m49s ago     Health check successful
192.168.1.101   cri          Running   OK       33m8s ago     Health check successful
192.168.1.101   dashboard    Running   ?        6m47s ago     Process Process(["/sbin/dashboard"]) started with PID 1916
192.168.1.101   etcd         Running   OK       5m41s ago     Health check successful
192.168.1.101   kubelet      Running   OK       32m47s ago    Health check successful
192.168.1.101   machined     Running   OK       6m49s ago     Health check successful
192.168.1.101   syslogd      Running   OK       6m48s ago     Health check successful
192.168.1.101   trustd       Running   OK       33m8s ago     Health check successful
192.168.1.101   udevd        Running   OK       6m49s ago     Health check successful

❯ talosctl service --nodes 192.168.1.111
NODE            SERVICE      STATE     HEALTH   LAST CHANGE   LAST EVENT
192.168.1.111   apid         Running   OK       16m15s ago    Health check successful
192.168.1.111   auditd       Running   OK       2m7s ago      Health check successful
192.168.1.111   containerd   Running   OK       2m7s ago      Health check successful
192.168.1.111   cri          Running   OK       16m16s ago    Health check successful
192.168.1.111   dashboard    Running   ?        2m5s ago      Process Process(["/sbin/dashboard"]) started with PID 1914
192.168.1.111   kubelet      Running   OK       15m51s ago    Health check successful
192.168.1.111   machined     Running   OK       2m7s ago      Health check successful
192.168.1.111   syslogd      Running   OK       2m6s ago      Health check successful
192.168.1.111   udevd        Running   OK       2m7s ago      Health check successfu
```

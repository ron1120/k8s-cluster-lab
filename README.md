# k8s-cluster-lab

A local Kubernetes cluster using Vagrant + VMware Fusion with 1 control plane and 1 worker node (Ubuntu 20.04 ARM64).

## Requirements

- [Vagrant](https://www.vagrantup.com/)
- [VMware Fusion](https://www.vmware.com/products/fusion.html)
- [Vagrant VMware Desktop plugin](https://developer.hashicorp.com/vagrant/docs/providers/vmware/installation)

```bash
vagrant plugin install vagrant-vmware-desktop
```

## Cluster Overview

| Node | Hostname | IP | vCPUs | RAM |
|---|---|---|---|---|
| Control Plane | `controlplane` | 192.168.56.10 | 2 | 2 GB |
| Worker | `worker` | 192.168.56.11 | 2 | 2 GB |

**Kubernetes version:** 1.29  
**Container runtime:** containerd 1.7  
**CNI:** Flannel

---

## Setup

### 1. Start the VMs

```bash
vagrant up
```

This provisions both VMs and installs `containerd`, `kubeadm`, `kubelet`, and `kubectl` on each node automatically.

---

### 2. Initialize the control plane

```bash
vagrant ssh controlplane
```

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.56.10 \
  --pod-network-cidr=10.244.0.0/16
```

Set up kubeconfig for the vagrant user:

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install the Flannel CNI:

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

At the end of `kubeadm init`, copy the `kubeadm join` command printed in the output — you'll need it in the next step.

---

### 3. Join the worker node

```bash
vagrant ssh worker
```

Paste the join command from step 2:

```bash
sudo kubeadm join 192.168.56.10:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

---

### 4. Verify the cluster

Back on the control plane:

```bash
vagrant ssh controlplane
kubectl get nodes -o wide
```

Expected output:

```
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   ...   v1.29.x
worker         Ready    <none>          ...   v1.29.x
```

---

## Daily Usage

```bash
# Start all VMs
vagrant up

# SSH into a node
vagrant ssh controlplane
vagrant ssh worker

# Stop all VMs (saves state)
vagrant halt

# Destroy all VMs
vagrant destroy -f
```

## Re-initializing the Cluster

If you need to reset and re-init from scratch:

```bash
# On controlplane
sudo kubeadm reset -f
sudo kubeadm init --apiserver-advertise-address=192.168.56.10 --pod-network-cidr=10.244.0.0/16

# On worker
sudo kubeadm reset -f
sudo kubeadm join 192.168.56.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
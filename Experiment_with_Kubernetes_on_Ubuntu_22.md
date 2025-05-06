# To experiment with Kubernetes on Ubuntu 22.04 machine, you can use **kubeadm** to set up a lightweight Kubernetes cluster. Given your system's limited resources, this will be a minimal setup with 1 master node and (optionally) 1 worker node — though both can run on the same machine for learning purposes.

Below are the **step-by-step instructions** to install and set up Kubernetes using `kubeadm`:

---

## ✅ Prerequisites

* Ubuntu 22.04
* Minimum: 2 CPU cores and 2 GB RAM
* Swap disabled
* Root/sudo access
* Internet connection

---

## 🛠️ Step 1: Disable Swap

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

---

## 🛠️ Step 2: Load Kernel Modules

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

Set up sysctl params:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

---

## 🛠️ Step 3: Install Container Runtime (Containerd)

```bash
sudo apt update && sudo apt install -y containerd

# Generate default config
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

## 🛠️ Step 4: Install Kubernetes Components (kubeadm, kubelet, kubectl)

```bash
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl

sudo curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 🛠️ Step 5: Initialize the Kubernetes Control Plane (Master Node)

> Run this on your master node

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

After it finishes, you'll get a message like:

```bash
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Copy this command — it will be used for worker node(s).

Set up `kubectl` access for your user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 🛠️ Step 6: Install Pod Network (We’ll use Flannel)

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

---

## ✅ Optional: Set up a Worker Node

> On a separate Ubuntu 22 machine with same setup, run **steps 1–4** to install Docker and kubeadm.

Then, **run the `kubeadm join` command** you copied from the master node.

Example:

```bash
sudo kubeadm join 192.168.0.10:6443 --token abcdef.0123456789abcdef \
--discovery-token-ca-cert-hash sha256:1234567890abcdef...
```

---

## 🧪 Verify Your Cluster

Back on the master node:

```bash
kubectl get nodes
kubectl get pods -A
```

You should see your master and any joined worker node.

---

## 🌱 Useful kubectl Commands

```bash
# View all nodes
kubectl get nodes

# View system pods
kubectl get pods -A

# Deploy a sample app
kubectl create deployment nginx --image=nginx

# Expose the deployment
kubectl expose deployment nginx --port=80 --type=NodePort

# Check services
kubectl get svc
```

---

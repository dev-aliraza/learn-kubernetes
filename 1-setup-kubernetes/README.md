# Setup Kubernetes with Kubeadm on Ubuntu 20.04 (2 Nodes, 1 Control Plane)

## Step 1: Install Required Debian Packages
Run  update and install five debian packages
```
- sudo apt-get update
- sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates lsb-release
```

## Step 2: Install kubelet, kubeadm and kubectl
1) Download and store public key for `apt` repository
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
2) Add kubernetes `apt` repository
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
3) Install kubernetes packages
```
- sudo apt-get update
- sudo apt-get install -y kubelet kubeadm kubectl
```
4) Enable `kubelet` on system start
```
sudo systemctl enable kubelet
```
## Step 3: Disable Swap Disk Permanently
1) Open `fstab` file
```
sudo vim /etc/fstab
```
2) Comment line `file system` line having type `swap`
```
# UUID=XXXXX    none   swap   sw    0   0
```
Above settings will have an effect when you restart system but if you want to disable swap for running ubuntu session, run following commands:
```
- sudo swapoff -a
- sudo mount -a
```
## Step 4: Enable Kernel Modules and Add `sysctl` Settings
1) Automatically load and enable kernel modules on system start
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```
2) Enable kernel modules on runtime for current ubuntu or debian session
```
- sudo modprobe overlay
- sudo modprobe br_netfilter
```
3) Add `sysctl` settings 
```
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system
```
## Step 5: Install Container Runtime
Kubernetes uses `container-runtime` to run container inside a pod. Supported runtimes are:
* Docker _(with additional docker-shim because kubernetes drop native docker support from its runtime)_
* CRI-O
* Containerd
### Install `containerd` as Runtime
1) Add docker public key for `apt` repository
```
- sudo mkdir -p /etc/apt/keyrings
- curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
2) Add docker `apt` repository
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
3) Install and enable containerd
```
- sudo apt update
- sudo apt install -y containerd.io
- sudo systemctl enable containerd
```
## Step 6(A): Initialize Kubernetes Cluster
Run following command to initialize kubernetes cluster _(Run this command only on `Master or Manager or Control Plane` Node(s))_
```
sudo kubeadm init \
  --pod-network-cidr=172.16.0.0/16 \
  --cri-socket unix:///run/containerd/containerd.sock
```
## Step 6(B): Initialize Kubernetes Cluster
Run following command to join kubernetes cluster _(Run this command only on `Worker` Node(s))_
```
kubeadm join {MASTER_NODE_IP}:6443 --token [KUBE_TOKEN] \
    --discovery-token-ca-cert-hash sha256:c692fb049u15883b575bd6710779db5c5af8073f7cab460lod181yr3ddb29a18
```

# install-k8s-with-kubeadm


# on All Nodes
### Enable IPTables Bridged Traffic:
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl –system

# Disable Swap
sudo swapoff -a
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
```

### CRI, SysCTL and OverlayFS setup:
```
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

# Set up required sysctl params, these persist across reboots.

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Enable overlayFS & VxLan pod communications
sudo modprobe overlay
sudo modprobe br_netfilter

# Sysctl params over reboots
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

### CRI-O Repo setup for Ubuntu:
You can setup it following this link: https://cri-o.io
```
# Setup CRI repos for Jammy Jellyfish (Ubuntu 22.04)
OS="jammy"
VERSION="1.31"
cat <<EOF | sudo tee
/etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb
https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS//
EOF
cat <<EOF | sudo tee
/etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb
http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS//
EOF
# GPG Keys
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

### CRI-O Installation:
```
# Install CRI-O and Tools
sudo apt-get update
sudo apt-get install cri-o cri-o-runc cri-tools -y

# Reload Daemon and enable CRIO
sudo systemctl daemon-reload
sudo systemctl enable crio --now

# If using ContainerD
sudo apt -y install curl apt-transport-https containerd; 
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```
### Install kubeadm, kubelet and kubectl:
Follow this documentation: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm


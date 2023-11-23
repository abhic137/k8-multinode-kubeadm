# k8-multinode-kubeadm
# Multi-Node Kubernetes Cluster Setup Using Kubeadm Using Containerd Runtime
This readme provides step-by-step instructions for setting up a multi-node Kubernetes cluster using Kubeadm. 
```VIDEO REF:``` https://www.youtube.com/watch?v=6_i1hXXviHw&t=313s
``` REF:``` https://computingforgeeks.com/install-kubernetes-cluster-ubuntu-jammy/

## Overview
This guide provides detailed instructions for setting up a multi-node Kubernetes cluster using Kubeadm. The guide includes instructions for installing and configuring containerd and Kubernetes, disabling swap, initializing the cluster, installing Flannel, and joining nodes to the cluster.

## Prerequisites
Before starting the installation process, ensure that the following prerequisites are met:

- You have at least two Ubuntu 18.04 or higher servers available for creating the cluster.
- Each server has at least 2GB of RAM and 2 CPU cores.
- The servers have network connectivity to each other.
- You have root access to each server.

## Installation Steps
The following are the step-by-step instructions for setting up a multi-node Kubernetes cluster using Kubeadm:

Update the system's package list and install necessary dependencies using the following commands:

```
sudo apt-get update
sudo apt install apt-transport-https curl -y
```

## Install containerd
To install Containerd, use the following commands:

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y
```

## Create containerd configuration
Next, create the containerd configuration file using the following commands:

```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

## Edit /etc/containerd/config.toml
Edit the containerd configuration file to set SystemdCgroup to true. Use the following command to open the file:

```
sudo nano /etc/containerd/config.toml
```

Set SystemdCgroup to true:
```
SystemdCgroup = true
```

Restart containerd:
```
sudo systemctl restart containerd
```

## Install Kubernetes
To install Kubernetes, use the following commands:

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install kubeadm kubelet kubectl kubernetes-cni
```

## Disable swap
Disable swap using the following command:

```
sudo swapoff -a
```

If there are any swap entries in the /etc/fstab file, remove them using a text editor such as nano:
```
sudo nano /etc/fstab
```

Enable kernel modules
```
sudo modprobe br_netfilter
```

Add some settings to sysctl
```
sudo sysctl -w net.ipv4.ip_forward=1
```
## Initialize the Cluster (Run only on master)
Use the following command to initialize the cluster:
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Create a .kube directory in your home directory:
```
mkdir -p $HOME/.kube
```

Copy the Kubernetes configuration file to your home directory:
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

Change ownership of the file:
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Install Flannel (Run only on master)
Use the following command to install Flannel:
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
```

## Verify Installation
Verify that all the pods are up and running:

```
kubectl get pods --all-namespaces
```

## Join Nodes
To add nodes to the cluster, run the kubeadm join command with the appropriate arguments on each node. The command will output a token that can be used to join the node to the cluster.
## Important Links
https://www.youtube.com/watch?v=pcADx8JFUIA
https://www.youtube.com/watch?v=Zxozz8P_l5M

# Multi-Node Kubernetes Cluster Setup Using Kubeadm Using Docker CE Runtime
## 1. Upgrade your Ubuntu servers

Provision the servers to be used in the deployment of Kubernetes on Ubuntu 22.04. The setup process will vary depending on the virtualization or cloud environment you’re using.

Once the servers are ready, update them.
```
sudo apt update
sudo apt -y full-upgrade
[ -f /var/run/reboot-required ] && sudo reboot -f
```
## 2. Install kubelet, kubeadm and kubectl

Once the servers are rebooted, add Kubernetes repository for Ubuntu 22.04 to all the servers.
```
sudo apt install curl apt-transport-https -y
curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Then install required packages.
```
sudo apt update
sudo apt install wget curl vim git kubelet kubeadm kubectl -y
sudo apt-mark hold kubelet kubeadm kubectl
```
Confirm installation by checking the version of kubectl.
```
$ kubectl version --client
Client Version: v1.28.0
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3

$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"28", GitVersion:"v1.28.0", GitCommit:"855e7c48de7388eb330da0f8d9d2394ee818fb8d", GitTreeState:"clean", BuildDate:"2023-08-15T10:20:15Z", GoVersion:"go1.20.7", Compiler:"gc", Platform:"linux/amd64"}
```
## 3. Disable Swap Space

Disable all swaps from /proc/swaps.
```
sudo swapoff -a 
```
Check if swap has been disabled by running the free command.
```
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       283Mi       5.9Gi       1.0Mi       1.5Gi       7.2Gi
Swap:             0B          0B          0B
```
Now disable Linux swap space permanently in /etc/fstab. Search for a swap line and add # (hashtag) sign in front of the line.
```
sudo sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
```
Or manually edit.
```
$ sudo vim /etc/fstab
#/swap.img	none	swap	sw	0	0
```
Confirm setting is correct
```
sudo mount -a
free -h
```
Enable kernel modules and configure sysctl.

## Enable kernel modules
```
sudo modprobe overlay
sudo modprobe br_netfilter
```
## Add some settings to sysctl
```
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
## Reload sysctl
```
sudo sysctl --system
```

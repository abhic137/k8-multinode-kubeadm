# k8-multinode-kubeadm
# Multi-Node Kubernetes Cluster Setup Using Kubeadm Using Containerd Runtime
This readme provides step-by-step instructions for setting up a multi-node Kubernetes cluster using Kubeadm. 
```VIDEO REF:``` https://www.youtube.com/watch?v=6_i1hXXviHw&t=313s
``` REF:``` https://computingforgeeks.com/install-kubernetes-cluster-ubuntu-jammy/
``` REF:``` https://medium.com/@mrdevsecops/set-up-a-kubernetes-cluster-with-kubeadm-508db74028ce
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
```OR```
Using single command
```
sudo sed -i 's/\(SystemdCgroup\s*=\s*\)false/\1true/' /etc/containerd/config.toml && sudo cat /etc/containerd/config.toml

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
## 4. Install Container runtime Docker CE
Docker runtime

If your container runtime of choice is Docker CE, follow steps provided below to configure it.
```
# Add repo and Install packages
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io docker-ce docker-ce-cli

# Create required directories
sudo mkdir -p /etc/systemd/system/docker.service.d

# Create daemon json config file
sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# Start and enable Services
sudo systemctl daemon-reload 
sudo systemctl restart docker
sudo systemctl enable docker

# Configure persistent loading of modules
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF

# Ensure you load modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
For Docker Engine you need a shim interface. 
 Initialize control plane (run on first master node)

Login to the server to be used as master and make sure that the br_netfilter module is loaded:
```
$ lsmod | grep br_netfilter
br_netfilter           22256  0 
bridge                151336  2 br_netfilter,ebtable_broute
```
Enable kubelet service.
```
sudo systemctl enable kubelet
```
We now want to initialize the machine that will run the control plane components which includes etcd (the cluster database) and the API Server.

Pull container images:
```
$ sudo kubeadm config images pull --v=5
 

[config/images] Pulled registry.k8s.io/kube-apiserver:v1.28.0
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.28.0
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.28.0
[config/images] Pulled registry.k8s.io/kube-proxy:v1.28.0
[config/images] Pulled registry.k8s.io/pause:3.9
[config/images] Pulled registry.k8s.io/etcd:3.5.9-0
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.10.1
```
Docker
```
sudo kubeadm config images pull --cri-socket /run/cri-dockerd.sock
```
Bootstrap without shared endpoint

To bootstrap a cluster without using DNS endpoint, run:
```
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16
```
Bootstrap with shared endpoint (DNS name for control plane API)

Set cluster endpoint DNS name or add record to /etc/hosts file.
```
$ sudo vim /etc/hosts
172.29.20.5 k8s-cluster.computingforgeeks.com
```
Create cluster:
```
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --upload-certs \
  --control-plane-endpoint=k8sapi.computingforgeeks.com
```
Note: If 10.244.0.0/16 is already in use within your network you must select a different pod network CIDR, replacing 10.244.0.0/16 in the above command.

Container runtime sockets:
Runtime	Path to Unix domain socket
Docker	/run/cri-dockerd.sock
containerd	/run/containerd/containerd.sock
CRI-O	/var/run/crio/crio.sock
Configure kubectl using commands in the output:

mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Check cluster status:

$ kubectl cluster-info
Kubernetes master is running at https://k8s-cluster.computingforgeeks.com:6443
KubeDNS is running at https://k8s-cluster.computingforgeeks.com:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluste
You can optionally pass Socket file for runtime and advertise address depending on your setup.
```
# Docker
# Must do https://computingforgeeks.com/install-mirantis-cri-dockerd-as-docker-engine-shim-for-kubernetes/
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --cri-socket /run/cri-dockerd.sock  \
  --upload-certs \
  --control-plane-endpoint=k8s-cluster.computingforgeeks.com
```
Configure kubectl using commands in the output:
```
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Check cluster status:
```
$ kubectl cluster-info
Kubernetes master is running at https://k8s-cluster.computingforgeeks.com:6443
KubeDNS is running at https://k8s-cluster.computingforgeeks.com:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluste
```
## 6. Install Kubernetes network plugin

In this guide we’ll use Flannel network plugin. You can choose any other supported network plugins. Flannel is a simple and easy way to configure a layer 3 network fabric designed for Kubernetes.

Download installation manifest.
```
wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```
If your Kubernetes installation is using custom podCIDR (not 10.244.0.0/16) you need to modify the network to match your one in downloaded manifest.
```
$ vim kube-flannel.yml
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```
Then install Flannel by creating the necessary resources.
```
$ kubectl apply -f kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```
Confirm that all of the pods are running, it may take seconds to minutes before it’s ready.
```
$ kubectl get pods -n kube-flannel
NAME                    READY   STATUS    RESTARTS   AGE
kube-flannel-ds-pppw4   1/1     Running   0          2m16s
```
Confirm master node is ready:
```
kubectl get nodes -o wide
```
## 7. Add worker nodes

With the control plane ready you can add worker nodes to the cluster for running scheduled workloads.

If endpoint address is not in DNS, add record to /etc/hosts.
```
$ sudo vim /etc/hosts
172.29.20.5 k8s-cluster.computingforgeeks.com
```
The join command that was given is used to add a worker node to the cluster.
```
kubeadm join k8s-cluster.computingforgeeks.com:6443 \
  --token sr4l2l.2kvot0pfalh5o4ik \
  --discovery-token-ca-cert-hash sha256:c692fb047e15883b575bd6710779dc2c5af8073f7cab460abd181fd3ddb29a18
```
```
Output:

[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.21" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
```
Run below command on the control-plane to see if the node joined the cluster.
```
$ kubectl get nodes
NAME                                 STATUS   ROLES    AGE   VERSION
k8s-master01.computingforgeeks.com   Ready    master   10m   v1.26.1
k8s-worker01.computingforgeeks.com   Ready    <none>   50s   v1.26.1
k8s-worker02.computingforgeeks.com   Ready    <none>   12s   v1.26.1
```
```
$ kubectl get nodes -o wide
```
# For removing the nodes

```
kubectl drain <your-node-name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <your-node-name>
sudo kubeadm reset
```

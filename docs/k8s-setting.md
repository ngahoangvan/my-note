# Pre-requisites

## Hardware requirement
- PC or VM for Node Master
- Raspberry PI for Node Worker

## OS requirement

- Bionic 18.04 LTS (Highly recommended) in PC or VM
- Raspbian GNU/Linux 9 in Raspberry PI

# Base setup in Node Master and Node Worker

### Step 1: Setting Host and Install Docker

```bash
# Setting Host
raspi_name=your_machine_name &&\
sudo echo ${raspi_name} > /etc/hostname &&\
sudo echo -e "127.0.0.1\t${raspi_name}" >> /etc/hosts

# Install Docker
curl -sfL https://get.docker.com | sh - && \
sudo usermod -aG docker $USER

# Only for Raspberry PI
cmdline=$(cat /boot/cmdline.txt) && \
echo $cmdline cgroup_memory=1 cgroup_enable=memory cgroup_enable=cpuset | sudo tee /

# Must Reboot machine
sudo reboot
```

### Step 2: Stop and remove watchdog

```bash
# Stop and disable watchdog service
sudo systemctl stop watchdog.service && \
sudo systemctl disable watchdog.service

# Remove watchdog
sudo apt remove watchdog

# Remove bmc2835_wdt in system modules
sudo sed -i 's|bcm2835_wdt||' /etc/modules
```

### Step 3: Install k8s include kubeadm, kubectl and kubelet

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
sudo apt-get update -q && \
sudo apt-get install -qy kubeadm
# Should reboot machine
sudo reboot
```

### Step 4: Pull image config by kubeadm

```bash
# Check list image config
kubeadm config images list
# Pull list image config
kubeadm config images pull
```

## Setting in node master

### Step 1: Disable swap in PC (Kubernetes requires swap to be disabled)
```bash
sudo swapoff -a
```

### Step 2: Creating a Single Master Cluster with kubeadm

```bash
# Using pod network Flannel
kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml

```

### Step 3: Check Node Master

```bash
kubectl get nodes
```

## Setting in node woker

### Step 1: Disable swap in Raspberry PI (Kubernetes requires swap to be disabled)

```
sudo dphys-swapfile swapoff && \
sudo dphys-swapfile uninstall && \
sudo update-rc.d dphys-swapfile remove
```

### Step 2: Join master cluster

After you init cluster, you have a mini tutorial to show you about the ways Node Worker join to Master Cluster. The command look line:

```bash
kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### Step 3: Check Node Worker in Master Machine

```bash
kubeadm get nodes
```

### Step 4: Change label of Node Worker (Not really unnecessary)

```bash
kubectl label node <name_node_worker> node-role.kubernetes.io/worker=worker
```

## Uninstall Kubernetes completely if you get error (usually due kubelet activating (auto-restart))

```bash
# If error in Node Master
kubectl drain <node name> — delete-local-data — force — ignore-daemonsets
kubectl delete node <node name>

kubeadm reset
sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*

# Reinstall k8s
```
# Reference

## [Homepage k8s.io](https://k8s.io/)

## [Document Setup k8s: Create cluster kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

## [Setting up a Kubernetes 1.11 Raspberry Pi Cluster using kubeadm](https://kubecloud.io/setting-up-a-kubernetes-1-11-raspberry-pi-cluster-using-kubeadm-952bbda329c8)

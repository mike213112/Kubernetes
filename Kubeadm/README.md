# How To Setup Kubernetes Cluster Using Kubeadm

Kubeadm is an excellent tool to set up a working kubernetes cluster in 
minutes. It does all the heavy lifting in terms of setting up all kubernetes 
components. It follows all the configuration best practices for a kubernetes cluster.

This post walks you through the process of setting up a kubernetes 
cluster with one master and two worker nodes using Kubeadm. I use 
kubeadm for all my kubernetes test clusters. You can set up the kubernetes 
cluster using kubeadm under 7 minutes.



## Prerequisires:
- Minimum two Ubuntu nodes [One master and one worker node]. 
You can have more worker nodes as per your requirement.

- The master node should have a minimum for 2 vCPU and 10GB memory.

- 10.X.X.X/X network range for master and nodes. We will be using 
the 192 series as the pod network range. The Calico network 
plugin will use this range by default.



### Before you begin using Kubeadm

- A compatible Linux host. The Kubernetes project provides generic instructions for Linux distributions based on Debian and Red Hat, and those distributions without a package manager.
- 2 GB or more of RAM per machine (any less will leave little room for your apps).
- 2 CPUs or more.
- Full network connectivity between all machines in the cluster (public or private network is fine).
- Unique hostname, MAC address, and product_uuid for every node. See [here](#verify-the-mac-address-and-product_uuid are-unique-for-every-node) for more details.
- Certain ports are open on your machines. See [here](#port-requirements) for more details.
- Swap disabled. You MUST disable swap in order for the kubelet to work properly.



#### Verify the MAC address and product_uuid are unique for every node
- You can get the MAC address of the network interfaces using the command 
ip link or ifconfig -a
- The product_uuid can be checked by using the command sudo cat 
/sys/class/dmi/id/product_uuid
It is very likely that hardware devices will have unique addresses, although some virtual 
machines may have identical values. Kubernetes uses these values to uniquely identify 
the nodes in the cluster. If these values are not unique to each node, the installation 
process may [fail](https://github.com/kubernetes/kubeadm/issues/31).



#### Port Requirements

| Protocol | Direction | Port Range | Porpose | Used By |
| -------- | --------- | ---------- | -------- | ------- |
| TCP | Inbound | 6443 | Kubernetes Api Server | All |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver,etcd|
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10251 | kube-scheduler | Self |
| TCP | Inbound | 10252 | kube-controller-manager | Self |



# We will use VirtualBox to create the virtual machines 

**We must have virtualbox installed, if you do not have it, you can download it.**

- [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)
**We can work with Ubuntu Server or Debian.**

- [Download Debian](https://mega.nz/file/0D5RgYSb#DXWL8B4FII3iOq3Adprz3WEORM33rut8sqffaGAlwps)
- [Download Ubuntu Server](https://mega.nz/file/JawS2BYB#vUPwCf4i48tbmZfMUasP7l6q6re_NgB12m1xXQF0O2g)


**Apply on all nodes(virtual machines).**

**Install Docker in Ubuntu.**

As a first step, we need to install Docker on all the nodes. Execute the following commands on all the nodes.

**Install the required packages for Docker**

```bash
sudo apt update; sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-releas
```

**Add the Docker GPG key and stable repository.**

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Install the Docker Engine.**

```bash
sudo apt update; sudo apt-get install docker-ce docker-ce-cli containerd.io
```

**Note: 
We must be as super users to execute the following command**

**Add the docker daemon configurations.**

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```


**Create a service director.y**

```bash
mkdir -p /etc/systemd/system/docker.service.d
```


**Restart Docker service.**

```bash
systemctl daemon-reload
systemctl restart docker
```



# Install Kubeadm


**Add the GPG key**

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```


**Add the Kubernetes apr repository**

```bash
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```


**Install kubeadm**

```bash
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```



# Initialize Kubeadm On Master Node

**Note: 
We must be as super users to execute the following command**


**Disable swap**

```bash
swapoff -a
```

**Initial kubeadm on master node with the following command. It will set up all the Kubernetes master components.**

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```

**On a successful kubeadm initialization you should get the following output.**

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.9.150:6443 --token 922dvt.3opyjya2e5nxon2y \
    --discovery-token-ca-cert-hash sha256:1481d21c260d5e64ddc4ea135a0339820f17cdcff4e1b1ba160ffdd0ddf15c9a
```

In the above output, there are two important blocks.

**kubeconfig:**

Use the following commands from the output to create the **kubeconfig** in master so that you can use **kubectl** to interact with cluster API.

**Note:** You can copy the **admin.conf** file from the master to your workstation if you don’t want to execute **kubectl** commands from the master.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Kubeadm Join Token**

The following command from the output is important to join the worker nodes to the master.

```bash
kubeadm join 192.168.9.150:6443 --token 922dvt.3opyjya2e5nxon2y \
    --discovery-token-ca-cert-hash sha256:1481d21c260d5e64ddc4ea135a0339820f17cdcff4e1b1ba160ffdd0ddf15c9a
```

**Install Calico Network Plugin:**

Execute the following command to install the calico network plugin on the cluster. Make sure you execute the kubectl command from where you have configured the **kubeconfig** file.

```bash
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

Check master node status using the following command.

```bash
kubectl get nodes
```

**On Nodes**

On all the nodes, execute the kubeadm join command you got from the output.

```bash
kubeadm join 192.168.9.150:6443 --token 922dvt.3opyjya2e5nxon2y \
    --discovery-token-ca-cert-hash sha256:1481d21c260d5e64ddc4ea135a0339820f17cdcff4e1b1ba160ffdd0ddf15c9a
```

From the master node, execute the following command to check if the node is added to the master.

```bash
kubectl get nodes
```

**Output**

```bash

```


# Initialize Kubeadm with **Load Balancer** and two **Masters**

## **Load Balancer Configuration**

- 1 Login as super users in Ubuntu
  - ```bash
sudo su
```

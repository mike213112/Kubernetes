# How To Setup Kubernetes Cluster Using Kubeadm

Kubeadm is an excellent tool to set up a working kubernetes cluster in
minutes. It does all the heavy lifting in terms of setting up all kubernetes
components. It follows all the configuration best practices for a kubernetes cluster.

This post walks you through the process of setting up a kubernetes
cluster with one master and two worker nodes using Kubeadm. I use
kubeadm for all my kubernetes test clusters. You can set up the kubernetes
cluster using kubeadm under 7 minutes.



---
---
## **Prerequisires:**
- Minimum two Ubuntu nodes [One master and one worker node].
You can have more worker nodes as per your requirement.

- The master node should have a minimum for 2 vCPU and 10GB memory.

- 10.X.X.X/X network range for master and nodes. We will be using
the 192 series as the pod network range. The Calico network
plugin will use this range by default.



---
---
## **Before you begin using Kubeadm**

- A compatible Linux host. The Kubernetes project provides generic instructions for Linux distributions based on Debian and Red Hat, and those distributions without a package manager.
- 2 GB or more of RAM per machine (any less will leave little room for your apps).
- 2 CPUs or more.
- Full network connectivity between all machines in the cluster (public or private network is fine).
- Unique hostname, MAC address, and product_uuid for every node. See [here](#verify-the-mac-address-and-product_uuid-are-unique-for-every-node) for more details.
- Certain ports are open on your machines. See [here](#port-requirements) for more details.
- Swap disabled. You MUST disable swap in order for the kubelet to work properly.



---
---
## **Verify the MAC address and product_uuid are unique for every node**
- You can get the MAC address of the network interfaces using the command
ip link or ifconfig -a
- The product_uuid can be checked by using the command sudo cat
/sys/class/dmi/id/product_uuid
It is very likely that hardware devices will have unique addresses, although some virtual
machines may have identical values. Kubernetes uses these values to uniquely identify
the nodes in the cluster. If these values are not unique to each node, the installation
process may [fail](https://github.com/kubernetes/kubeadm/issues/31).



---
---
## **Port Requirements**

| Protocol | Direction | Port Range | Porpose | Used By |
| -------- | --------- | ---------- | -------- | ------- |
| TCP | Inbound | 6443 | Kubernetes Api Server | All |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver,etcd|
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10251 | kube-scheduler | Self |
| TCP | Inbound | 10252 | kube-controller-manager | Self |



---
---
# **Configuration of our virtual machines**
- ## **Requirements for our Load_Balancer**
   * ### We assign 20GB of hard disk (Optional)
   * ### We assign 1GB of RAM (Recommended)
   * ### We assign 1 Processor (Recommended)

---
- ## **Requirements for our masters**
   * ### We assign 20GB of hard disk (Optional)
   * ### We assign 2GB of RAM (Recommended)
   * ### We assign 2 Processor (Recommended)

---
- ## **Requirements for our workers**
   * ### We assign 20GB of hard disk (Optional)
   * ### We assign 1GB of RAM (Recommended)
   * ### We assign 1 Processor (Recommended)



---
---
# **We will use VirtualBox to create the virtual machines**

### **We must have virtualbox installed, if you do not have it, you can download it.**

- [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)

---

### **We can work with Ubuntu Server or Debian.**

- [Download Debian](https://mega.nz/file/0D5RgYSb#DXWL8B4FII3iOq3Adprz3WEORM33rut8sqffaGAlwps)
- [Download Ubuntu Server](https://mega.nz/file/JawS2BYB#vUPwCf4i48tbmZfMUasP7l6q6re_NgB12m1xXQF0O2g)



---
---
# **Apply on all nodes(Virtual Machines).**

## **Install Docker in Ubuntu.**

---

  ### As a first step, we need to install Docker on all the nodes. Execute the following commands on all the nodes.


- [ ] **Install the required packages for Docker**

  ```bash
  sudo apt update; sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-releas
  ```

---

- [ ] **Add the Docker GPG key and stable repository.**

  ```bash
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

  echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```

---

- [ ] **Install the Docker Engine.**

  ```bash
  sudo apt update; sudo apt-get install docker-ce docker-ce-cli containerd.io
  ```

  > **Note:**
  We must be as super users to execute the following command

---

- [ ] **Add the docker daemon configurations.**

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

---

- [ ] **Create a service directory.**

  ```bash
  mkdir -p /etc/systemd/system/docker.service.d
  ```

---

- [ ] **Restart Docker service.**

  ```bash
  systemctl daemon-reload
  systemctl restart docker
  ```



---
---
# **Install Kubeadm**


- [ ] **Add the GPG key**

  ```bash
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  ```

---

- [ ] **Add the Kubernetes apr repository**

  ```bash
  apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
  ```

---

- [ ] **Install kubeadm**

  ```bash
  apt install kubelet=1.20.4-00 kubeadm=1.20.4-00 kubectl=1.20.4-00
  ```



---
---
# **Initialize Kubeadm On Master Node***

   > ### **Note:** We must be as super users to execute the following command

---

- [ ] **Disable swap**

  ```bash
  swapoff -a
  ```

---

- [ ] **Initial kubeadm on master node with the following command. It will set up all the Kubernetes master components.**

  ```bash
  kubeadm init --pod-network-cidr=192.168.0.0/16
  ```

---


- [ ] **On a successful kubeadm initialization you should get the following output.**

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

  kubeadm join 192.168.1.69:6443 --token 922dvt.3opyjya2e5nxon2y \
      --discovery-token-ca-cert-hash sha256:1481d21c260d5e64ddc4ea135a0339820f17cdcff4e1b1ba160ffdd0ddf15c9a
  ```

---

  #### In the above output, there are two important blocks.

---

- [ ] **kubeconfig:**

  Use the following commands from the output to create the **kubeconfig** in master so that you can use **kubectl** to interact with cluster API.

  > ### **Note:** You can copy the **admin.conf** file from the master to your workstation if you don’t want to execute **kubectl** commands from the master.



  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

---

- [x] **Kubeadm Join Token**

  #### The following command from the output is important to join the worker nodes to the master.

  ```bash
  kubeadm join 192.168.9.150:6443 --token 922dvt.3opyjya2e5nxon2y \
    --discovery-token-ca-cert-hash sha256:1481d21c260d5e64ddc4ea135a0339820f17cdcff4e1b1ba160ffdd0ddf15c9a
  ```

---

- [ ] **Install Calico Network Plugin:**

  #### Execute the following command to install the calico network plugin on the cluster. Make sure you execute the kubectl command from where you have configured the **kubeconfig** file.

  ```bash
  kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
  ```

---

- [ ] Check master node status using the following command.

  ```bash
  kubectl get nodes
  ```

---

- [ ] **On Nodes**

  #### On all the nodes, execute the kubeadm join command you got from the output.

  ```bash
  kubeadm join 192.168.1.69:6443 --token 922dvt.3opyjya2e5nxon2y \
    --discovery-token-ca-cert-hash sha256:1481d21c260d5e64ddc4ea135a0339820f17cdcff4e1b1ba160ffdd0ddf15c9a
  ```

---

- [ ] From the master node, execute the following command to check if the node is added to the master.

  ```bash
  kubectl get nodes
  ```

---

- [ ] **Output**

  ```bash
  NAME   STATUS   ROLES                  AGE   VERSION
  master   Ready    control-plane,master   36s   v1.20.4
  ```


---
---
# Initialize Kubeadm with **Load Balancer** and **two Masters**

  ## **Load Balancer Configuration**

 - [ ] 1. **Login as super users in Ubuntu**
    ```bash
    sudo su
    ```

---

- [ ] 2. **Install Nginx**
    ```bash
    apt install nginx -y
    ```

---

- [ ] 3. **Enable Nginx**
 *  ```bash
    systemctl enable nginx
    ```

---

- [ ] 4. **View status the Nginx**
*   ```bash
    systemctl status nginx
    ```

---
  > ### **NOTE:** If everything is correct with Our Nginx, we continue with the other configuration

---

- [ ] 5. **Create a superfolder**
 *  ```bash
    mkdir -p /etc/nginx/tcpconf.d
    ```

---

- [ ] 6. **Open the Nginx file**
 *  ```bash
    nano /etc/nginx/nginx.conf
    ```

---

- [ ] 7. **Add our subfolder in the NGINX file (we must put this at the bottom of all the nginx configuration)**
  ```bash
  include /etc/nginx/tcpconf.d/*;
  ```
---
  > ### **NOTE** Where (/ *) is for us to add everything that is inside our subfolder.
---

- [ ] 8. **Add Kubernetes configuration**
 *  ```bash
    cat <<EOF |sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
    stream {
        upstream kubernetes {
            server <ip del primer master>:6443;
            server <ip del segundo master>:6443;
        }

        server {
            listen 6443;
            listen 443;
            proxy_pass kubernetes;
        }
    }
    EOF
    ```

---
> ### **NOTE:** We must know the ips of our masters, and remove <> (it only goes to the ips and the port where Kubernetes Api Server listens)
---

- [ ] 9. **Reload the changes**
    ```bash
    nginx -s reload
    ```

---

- [ ] 10. **View load Balancer ip**
    ```bash
    hostname -I
    ```

---
---
# **Initialize Kubeadm on the first master node**

- [ ] 1. **Login as super users in Ubuntu**
 *  ```bash
    sudo su
    ```

---

- [ ] 2. **Disable Swap**
    ```bash
    swapoff -a
    ```

---
   > ### **NOTE:** We deactivate the swap so that it does not give us an error, when we raise our master, the same kubernetes will tell us that we disable it
---

- [ ] 3. **We have two ways to initialize our first master**
    - [ ] **First option:** Initialize our first master
      ```bash
      kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" --upload-certs
      ```

    - [ ] **Second option:** We create a file to configure the Kubernetes api and load our Load_Balancer.
      ```bash
      nano kubeadm-config.yaml
      ```

    - [ ] **We add the following in the file we create.**
      ```bash
      apiVersion: kubeadm.k8s.io/v1beta2
      kind: ClusterConfiguration
      kubernetesVersion: stable
      controlPlaneEndpoint: "ip del load_balancer:6443"
      ```

    - [ ] **Initialize our first master**
      ```bash
      kubeadm init --config=kubeadm-config.yaml --upload-certs
      ```

---
   > #### **NOTE:** When our master gets up he will generate 2 tokens, one that is for masters and another for workers. examples

   - [x] This would be for the masters
     ```bash
     kubeadm join 192.168.1.69:6443 --token phtt6h.ofnlzjagcynoaonn --discovery-token-ca-cert-hash \ sha256:ce243060a6c049f1adb4b73a9cdc5c09fbe0faec5f3fc5c27b606edf0d695079 \
     --control-plane --certificate-key 9997af25e442a472d1d4ee0eefe46a305f98d78a5fd55d065631ba3c6be23d10
     ```

   - [x] This would be for the workers
      ```bash
      kubeadm join 192.168.1.69:6443 --token phtt6h.ofnlzjagcynoaonn --discovery-token-ca-cert-hash \ sha256:ce243060a6c049f1adb4b73a9cdc5c09fbe0faec5f3fc5c27b606edf0d695079
      ```

   - [ ] To finish with the configuration of our cluster, and that we can work with our master, kubernetes tells us that we have to do the following 3 steps.
     ```bash
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
     ```

---

- [ ] 4. **Apply the Kubernetes CNI**
     ```bash
     kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')
     ```

- [ ] 5. **see the status of the pods in hot**
     ```bash
     kubectl get pod -n kube-system -w
     ```

> **NOTE:**
Once the status of our pods is RUNNING, it will take us out and we will verify the status again, to corroborate, with the command from step 5.

- [ ] 6. **Check if our master is READY**
     ```bash
     kubectl get nodes
     ```



---
---
# **Initialize Our Second Master**

- [ ] 1. **Login as super users in Ubuntu**
     ```bash
     sudo su
     ```

- [ ] 2. **We copy the token that the first master generated us**
     ```bash
     kubeadm join 192.168.1.69:6443 --token phtt6h.ofnlzjagcynoaonn --discovery-token-ca-cert-hash \ sha256:ce243060a6c049f1adb4b73a9cdc5c09fbe0faec5f3fc5c27b606edf0d695079 \
     --control-plane --certificate-key 9997af25e442a472d1d4ee0eefe46a305f98d78a5fd55d065631ba3c6be23d10
     ```

- [ ] 3. **Once finished, Kubernetes will ask us to finish the configuration, let's add the following steps.**
     ```bash
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
     ```

- [ ] 4. **We verify the status of the masters**
     ```bash
     kubectl get nodes
     ```



---
---
# **Join the workers to our cluster**

- [ ] 1. **Login as super users in Ubuntu**
     ```bash
     sudo su
     ```

- [ ] 2. **We copy the token that the first master generated us**
     ```bash
     kubeadm join 192.168.1.69:6443 --token phtt6h.ofnlzjagcynoaonn --discovery-token-ca-cert-hash \ sha256:ce243060a6c049f1adb4b73a9cdc5c09fbe0faec5f3fc5c27b606edf0d695079
     ```

- [ ] 3. **Check if our workers joined our cluster(on any master)**
     ```bash
     kubectl get nodes
     ```


---
---
## **Dashboard (we can do it for any of the 2 masters that have a graphical interface)**

- [ ] 1. **Dashboard Deploying for the interface**
 *  ```bash
     kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
     ```

- [ ] 2. **We create the dashboar-admin.yml file(for the Role)**
     ```bash
     nano dashboard-admin.yml
     ```

- [ ] 3. **We add the configuration to the dashboar-admin.yml file**
     ```bash
     apiVersion: v1
     kind: ServiceAccount
     metadata:
       name: admin-user
       namespace: kube-system
     ```

- [ ] 4. **We apply the configuration file to kubernetes, for the admin role**
     ```bash
     kubectl apply -f dashboar-admin.yml
     ```

- [ ] 5. **We create a file for the ClusterRol**
     ```bash
     nano admin-role-binding.yml
     ```

- [ ] 6. **We add the configuration to the admin-role-binding.yml file**
     ```bash
     apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRoleBinding
     metadata:
       name: admin-user
     roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
     subjects:
     - kind: ServiceAccount
       name: admin-user
       namespace: kube-system
     ```

- [ ] 7. **We apply the configuration file to kubernetes, for the ClusterRol**
     ```bash
     kubectl apply -f admin-role-binding.yml
     ```

- [ ] 8. **Generate token, to be able to log in to the Dashboard page through token**
     ```bash
     kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
     ```

- [ ] 9. **To access the panel, we use the following**
     ```bash
     kubectl proxy
     ```

- [ ] 10. **Check if everything is correct, we open our browser and we can do the following**
     ```bash
     localhost:8001
     ```

> ### **Note:** If you show us a json file, we're fine


- [ ] 11. **Check again if we are on the right track**
     ```bash
     localhost:8001/iu
     ```

- [ ] 12. **Now we enter the Dashboard web page**
     ```bash
     http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
     ```

> ### **NOTE:** Once we are on the web page we will give you the token option, and we copy the token that step 8 generated us

---
---

## **Application deployment**
  [] 1. **In this case we are going to show a container using our own image mikedoc1/project:latest, which we downloaded from the docker hub**
     ```bash
     kubectl create deployment test01 --image=mikedoc1/project:latest
     ```

  [] 2. **We can see the status of our deployment. In Kubernetes, a deployment is a pod, which can be a single container or a set of related containers. A pod will always be deployed on a single machine**
     ```bash
     kubectl get pods
     ```

  [] 3. **We can also ask for the details of our deployment, with very interesting information about the node where it has been deployed, the events, etc.**
     ```bash
     kubectl describe pod test01
    ```

  [] 4. **We create the app.yaml file with the following information**
     ```bash
     apiVersion: v1
     kind: Pod
     metadata:
       name: mi-app
       labels:
         app: web
     spec:
       containers:
         - name: front-end
           image: nginx
           ports:
             - containerPort: 80
         - name: back-end
           image: redis
     ```

  > Expose services

  [] 5. **Our implementations will only be visible from within the cluster, if we want to give visibility to our implementations we must do it like this**
     ```bash
     kubectl expose deployment test01 --type=LoadBalancer --port=80
     ```

  [] 6. **Where we tell kubernetes to expose port 80 of the 'test01' service using load balancing. We can see the services**
     ```bash
     kubectl get services
     ```

  ### **Scale services**   
  [] 7. **Very attractive is the ease with which containerized applications can be scaled. For example, to scale the number of pods for the 'test01' service to 3**
     ```bash
     kubectl scale deployment --replicas=3 test01
     ```

---
---
# **If the cluster does not work or does not directly give us an error when raising the first master, Reset it and see the configuration again**

- [ ] 1. **Reset, either master or workers**
    ```bash
    kubeadm reset
    ```

- [ ] 2. **Delete the subfolder that has the kubernete configuration**
    ```bash
    rm -rf $HOME/.kube/config
    ```

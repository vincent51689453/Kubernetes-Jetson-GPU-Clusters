# Kubernetes-Jetson-GPU-Clusters
This repository is a guideline for setting up a GPU Cluster with Nvidia Jetson Series by Kubernetes. A **"Single Master Multi Slaves"** topology is implemented and tested using **Nvidia Jetson Xaiver (Master)** and **Nvidia Jetson TX2 (Slave)** with **JetPack 4.2.2**. 

**Nvidia Jetson Xavier**
----------------------------
The Nvidia Jetson AGX Xavier Developer Kit is the latest addition to the Jetson platform. It is an AI computer for autonomous machines and delivering the performance of a GPU workstation in an embedded module under 30W. It is optimial for robots, dones and other autonomous machines. 

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/Xavier_2.jpg)

**Nvidia Jetson TX2**
----------------------------
The Nvidia Jetson TX2 is a deeplearning platform which is able to provide you exceptional speed and power efficieny in an embedded AI computer device. This supercomputer-on-a-moduel brings true AI computing at the edge. Meanwhile, a wide range of standard hardware interfaces are suppored the fit a variety of products and form factors.

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/Jetson-TX2_2.jpg)

**GPU Cluster Setup**
---------------------------
### A. Master Node
The actual configuration steps I performed are written in ***Installation_Master***, some steps are not essential. You may refer to it if you have any difficulties. I will briefly explain the essential steps you have to perform while setting up master nodes.

1) Install ***JetPack 4.2.2*** by SDK Manager but not necessary to install tensorflow

2) Perform system update and system upgrade
```
$ sudo apt-get update && sudo apt-get upgrade
```
3) Install resources monitoring
```
$ sudo -H pip install -U jetson-stats
$ sudo jtop
```
![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/jtop.png)

4) Set to the highest power mode: MODE 30W ALL
```
$ sudo nvpmodel -m 3
```
5) Disable swap since it may cause issue with Kubernetes. Please notes that swap will be activated everytime the system starts. Remember to disable it.
```
$ sudo swapoff -a
```
6) Edit /etc/docker/daemon.json
```
$ sudo gedit /etc/docker/daemon.json
```
You can simply replace all the content of daemon.json with **Kubernetes-Jetson-GPU-Clusters/Maintainance/daemon.json**.

7) Refresh system
```
$ sudo apt-get update
$ sudo apt-get dist-upgrade
```
8) Add current user to docker group
```
$ sudo groupadd docker
$ sudo usermod -aG docker ianvidia
$ newgrp docker
$ sudo reboot
```
Please change ianvidia to your own account.

9) Test Docker GPU support
```
$ sudo docker run -it jitteam/devicequery ./deviceQuery
```
For the first time to execute this command, it will say there is no image locally. Therefore, please be patient for the system to pull the image from docker hub. If all the setup are done perfectly, you should get a **PASS** in this test.

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/device_pass.png)

10) Install curl
```
$ sudo apt install curl
```
11) Install kuberenets k8s
```
$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```
12) Confiure Master Node
```
$ sudo kubeadm init --pod-network-cidr=10.244.10.0/16 --kubernetes-version 1.18.2
```
***This command should only be executed on master node!. Never do it in the slave***
***Please keep the tokens in the bottom carefully. They cannot be regenearted!***
![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/join_master_info.png)

***If you have any problem after initialize the network, you can reset it by the following.***
```
$ sudo kubeadm reset
```
13) Read the response from **(12)** carefully and perform the following
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
14) Apply flannel to the cluster
```
$ sysctl net.bridge.bridge-nf-call-iptables=1
$ curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ sudo kubectl apply -f kube-flannel.yml
```
15) List out all the nodes
```
$ sudo kubectl get nodes
```
Master should be ***READY***.
Please ignore those "workers" in the picture. Since you did not configure any slave and join the cluster yet, they cannot be seen in this step.

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/Kubectl_get_node.png)

If you encounter an error message like: *"The connection to the sever localhost:8080 was refused - did you specify the right host or port?"*, execute the following to solve this iusse.
```
$ sudo cp /etc/kubernetes/admin.conf $HOME/
$ sudo chown $(id -u):$(id -g) $HOME/admin.conf
$ export KUBECONFIG=$HOME/admin.conf
```
16) ***Assume some slaves joined the network and they are READY"***, change slave label from <none> to worker
```
$ sudo kubectl label node jetson-tx2-004 node-role.kubernetes.io/worker=worker  
```
**jetson-tx2-004** is the device name of the slave.
  
17) Check wethere the cluster can support GPU
```
$ sudo kubectl apply -f gpu-clusters-test.yml
$ kubectl logs devicequery
```
The gpu-cluster-test.yml is inside ***YMAL-Config/pod***.
You should get another ***PASS*** in this test.

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/device_pass_2.png)

18) Apply customized image
```
$ sudo kubectl apply -f deeplearning-gpu-cluster.ymal
$ sudo kubectl get pod
```
The correct outpu should be the following.

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/Kubectl_get_pod.png)

If you want to get more detail about this pod, you may
```
$ sudo kubectl describe pod deeplearning
```

19) Attach to the customized container **only for the deeplearning-gpu-cluster**
```
$ ./access_jetson_tensorflow.sh
```
### B. Slave Node
The actual configuration steps I performed are written in ***Installation_Slave***, some steps are not essential. You may refer to it if you have any difficulties. I will briefly explain the essential steps you have to perform while setting up master nodes.
1) Install ***JetPack 4.2.2*** by SDK Manager but not necessary to install tensorflow

2) Perform system update and system upgrade
```
$ sudo apt-get update && sudo apt-get upgrade
```
3) Install resources monitoring
```
$ sudo -H pip install -U jetson-stats
$ sudo jtop
```
4) Set to the highest power mode: MODE MAXN
```
$ sudo nvpmodel -m 0
```
5) Disable swap since it may cause issue with Kubernetes. Please notes that swap will be activated everytime the system starts. Remember to disable it.
```
$ sudo swapoff -a
```
6) Edit /etc/docker/daemon.json
```
$ sudo gedit /etc/docker/daemon.json
```
You can simply replace all the content of daemon.json with **Kubernetes-Jetson-GPU-Clusters/Maintainance/daemon.json**.
7) Refresh system
```
$ sudo apt-get update
$ sudo apt-get dist-upgrade
```
8) Add current user to docker group
```
$ sudo groupadd docker
$ sudo usermod -aG docker nvidiatx2-004
$ newgrp docker
$ sudo reboot
```
Please change nvidiatx2-004 to your own account.

9) Test Docker GPU support
```
$ sudo docker run -it jitteam/devicequery ./deviceQuery
```
For the first time to execute this command, it will say there is no image locally. Therefore, please be patient for the system to pull the image from docker hub. If all the setup are done perfectly, you should get a **PASS** in this test.

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/device_pass.png)

10) Install curl
```
$ sudo apt install curl
```
11) Install kuberenets k8s
```
$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```
12) Join cluster created by master
```
$ ./slave_join_master.sh
```
***The tokens inside are various based on the creation of clusters. You have to replace with your own tokens.***
Master should be able to see the nodes.

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/Kubectl_get_node.png)

13) Activate kubernetes service (evertime power on)
```
$ ./start_nodes.sh
```
This script can be found in Maintainance.

### C. Common Issue
1) ***ERROR "no such files -> /run/flannel/subnet.env"***
Solu: copy this subnet.env file from master to all slaves which do not have this file

2) ***Error "cni0" already has an IP address different from 10.244.1.1/24***
Solu:
**Inside Slave**
```
$ sudo ifconfig  cni0 down
$ sudo brctl delbr cni0
$ sudo ip link delete flannel.1
$ ./Maintainance/reset_node.sh
```
**Then, go to MASTER and remove the node**
```
$ ./Maintainance/slave_join_master.sh
```

### D. Kubernetes DashBoard
Please perform the followin operation in **MASTER** and refer to ***Installation_Dashboard***.
Here is the reference url: https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/

1) Apply official ymal
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

2) Start kubectl proxy. **Do not close this terminal after execution**
```
$ sudo kubectl proxy
```
![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/proxy.png)

3) Open browser in **MASTER**
```
$ http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```
![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/token_request.png)

You will be able see a login page.

4) In order to get the token, it is neccesary to add a admin to manage the entire cluster.
```
$ sudo kubectl create -f admin-role.ymal
```
The admin-role.ymal can be found in **YMAL-Config/services/**

5) Get admin-token secret name
```
$ sudo kubectl -n kube-system get secret|grep admin-token
```
6) Get token value
```
$ sudo kubectl -n kube-system describe secret admin-token-sm4pn
```
***sm4pn*** should be your corresponding name

![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/token.png)

7) Copy the token and login.
![image](https://github.com/vincent51689453/Kubernetes-Jetson-GPU-Clusters/blob/master/GitHub_Image/Dashboard.png)

### E. Customized Image vincent51689453/jetson-deeplearning
https://hub.docker.com/repository/docker/vincent51689453/jetson-deeplearning
The steps are all described in ***Installation_Custom_Image***.

















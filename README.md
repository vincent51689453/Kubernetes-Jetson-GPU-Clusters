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


# LAB01: 安装配置Kubernetes环境

## 更新时间
2020.01.16

## 1. 实验拓扑
![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-topology.png)

## 2. 准备工作
Kubernetes V1.19版本兼容性：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.17.md

Citrix Ingress Controller 版本兼容性：https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/support-matrix/

Citrix ADC(CPX) 版本兼容性：https://docs.citrix.com/en-us/citrix-adc-cpx/13/about.html

注册Docker免费帐号：https://hub.docker.com/

Ubuntu Server下载：https://ubuntu.com/download/server

## 3. Kubernetes集群信息
|Host |Linux |Docker |Kubernetes |
|------------------------------------|------------------------------------|------------------------------------|------------------------------------|
|Master |Ubuntu Server 18.04.5 |19.03 |1.19 |
|Worker1 |Ubuntu Server 18.04.5 |19.03 |1.19 |
|Worker2 |Ubuntu Server 18.04.5 |19.03 |1.19 |

## 4. Kubernetes Cluster Networking信息
|Host |IP Address(MGMT) |Gateway |CNI |IP Address(DATA) |
|------------------------------------|------------------------------------|------------------------------------|------------------------------------|------------------------------------|
|Master |192.168.204.11/24 |192.168.201.254 |Flannel |192.168.204.11/24 |
|Worker1 |192.168.204.12/24 |192.168.201.254 |Flannel |192.168.204.12/24 |
|Worker2 |192.168.204.13/24 |192.168.201.254 |Flannel |192.168.204.13/24 |


## 5. Citrix ADC信息
|Citrix Platform |Software Version |
|------------------------------------|------------------------------------|
|Citrix ADM |13.0-71.40 |
|Citrix ADC VPX |13.0-71.40 |
|Citrix ADC CPX |13.0-36.29 |

## 6. Linux安装配置
### 6.1 硬件资源

    CPU: 2 vCPUs
    Memory: 2 GB
    Disk: 20 GB
    
### 6.2 Master, Worker上Linux系统安装

选择系统默认语言为English

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-01.png)

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-02.png)

配置主机静态IP地址

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-03.png)

创建管理员帐号

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-04.png)

安装SSH服务

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-05.png)

其他配置保持默认，完成安装。

### 6.3 Master, Worker上Linux系统配置

修改系统时区为UTC+8并确认NTP同步

运行命令: sudo timedatectl set-timezone Asia/Shanghai

运行命令: timedatectl

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-06.png)

编辑本地Host信息，静态绑定集群内所有Host信息

运行命令: sudo vi /etc/hosts

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-07.png)

关闭Swap：Kubernetes是一个用于大规模运行的分布式系统。在大量机器上运行大量容器时，您需要可预测性和一致性。禁用Swap是正确的方法。

运行命令: sudo swapoff -a

运行命令: sudo vi /etc/fstab

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-08.png)

更新系统已安装的软件包

运行命令: sudo apt-get update -y

运行命令: sudo apt-get upgrade -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-09.png)

Linux系统配置完成。

## 7. Master，Worker上Kubernetes安装

安装必要组件

运行命令: sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-10.png)

添加Docker所需要的GPG Key

运行命令: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-11.png)

添加Docker仓库

运行命令: sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-12.png)

更新软件包列表

运行命令: sudo apt-get update -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-13.png)

安装Docker 18.09

运行命令: sudo apt-get install docker-ce=5:18.09.9\~3-0\~ubuntu-bionic docker-ce-cli=5:18.09.9\~3-0\~ubuntu-bionic -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-14.png)

检查Docker版本

运行命令: sudo docker version

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-15.png)

使用国内阿里云作为Kubernetes源，添加Kubernetes的GPG Key

运行命令: curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-16.png)

使用国内阿里云作为Kubernetes源，添加Kubernetes仓库

运行命令: sudo add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-17.png)

更新软件包列表

运行命令: sudo apt-get update -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-18.png)

安装kubectl, kubeadm, kubectl 1.16版本

运行命令: sudo apt-get install kubelet=1.16.6-00 kubeadm=1.16.6-00 kubectl=1.16.6-00 -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-19.png)

修改Docker cgroup driver，从默认的cgroupfs改为systemd

运行命令: sudo vi /etc/docker/daemon.json

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-20.png)

运行命令: sudo systemctl daemon-reload

运行命令: sudo systemctl restart docker

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-21.png)

完成Kubernetes安装。

## 8. Master上Kubernetes配置

初始化Kubernetes集群

运行命令: sudo kubeadm init --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --pod-network-cidr=192.168.226.0/24 --apiserver-advertise-address=192.168.226.10

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-22.png)

注意：集群初始化成功后，保存输出信息，会用于Worker加入集群

运行命令: sudo mkdir -p $HOME/.kube

运行命令: sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

运行命令: sudo chown $(id -u):$(id -g) $HOME/.kube/config

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-23.png)

由于没有安装网络插件，Node状态为NotReady

运行命令: kubectl get nodes

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-24.png)

安装网络插件Weave

运行命令: kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')" 

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-25.png)

等待约30秒后，检查集群状态

运行命令: kubectl get nodes

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-26.png)

Master节点配置完成。

## 9. Worker上Kubernetes配置

输入刚才从Master节点上保存的命令

例如，运行命令: sudo kubeadm join 192.168.226.10:6443 --token is8sc0.ntn5cwtg6hmc0npf \
    --discovery-token-ca-cert-hash sha256:1b756d5fca53a0916652285c5c50195bba14ef5050de472bb881020110371083

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-27.png)

回到Master节点命令行，检查集群状态

运行命令: kubectl get nodes -o wide

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-28.png)

至此为止，Kubernetes集群已经搭建完成，我们部署了一个Master节点和两个Worker节点。

## 10. Master节点下载Citrix CPX镜像

Citrix CPX 13.0镜像托管在Docker Store上。我们需要先登录Docker Store，然后Checkout。Store地址：https://hub.docker.com/_/netscaler-cpx

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-29.png)

完成Checkout后，复制Pull链接: docker pull store/citrix/citrixadccpx:13.0-36.29

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-30.png)

SSH登录到Master节点

运行命令: sudo docker login

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-31.png)

运行命令: sudo docker pull store/citrix/citrixadccpx:13.0-36.29

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-32.png)

查看Image信息

运行命令: sudo docker image ls

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-33.png)

## 11. 部署和测试微应用
完成Kubernetes集群后，我们来做一个简单实验，看看Kubernetes如何部署应用、外部访问。

### 11. 自动补全kubectl, kubeadm命令参数

在Master节点上编辑配置文件并添加如下两行命令: vi ~/.bashrc

    source <(kubectl completion bash)
    source <(kubeadm completion bash)

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-34.png)

### 11.2 Worker节点创建测试页面

容器本身不含有存储，需要通过VolumeMount方式挂载外部存储。在本实验内，我们使用Worker节点的本地存储来作为容器的hostPath。

在Worker1、Worker2节点上分别创建目录和文件: /home/citrix/k8s/lab01/index.html

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-35.png)

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-36.png)

### 11.3 Master节点部署Deployment

从docker上我们知道有image, container概念，kubernetes作为集群编排系统，提出有pod, deployment, service等概念。

|Kubernetes概念                              |说明                                |
|------------------------------------|------------------------------------|
|pod                              |kubernetes编排系统的最小单位，包含一个或多个容器                              |
|deployment                              |kubernetes在多个worker上部署pod副本，支持弹性扩容、负载均衡、版本管理、金丝雀等                              |
|service                              |对指定的pod或deployment进行服务发现                              |

配置文件下载: https://github.com/yazshen/citrix-adc-kubernetes/blob/master/citrix-lab01-nginx-deployment.yaml

运行命令: wget https://github.com/yazshen/citrix-adc-kubernetes/blob/master/citrix-lab01-nginx-deployment.yaml

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-37.png)

在这个deployment配置里面，我们实现了如下目的：创建一个nginx pod、2个副本、指定80端口来访问nginx服务、挂载Worker节点上的目录。

运行命令: kubectl apply -f citrix-lab01-nginx-deployment.yaml

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-38.png)

查看deployment部署状态

运行命令: kubectl get deployments

运行命令: kubectl get pods -o wide

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-39.png)

### 11.4 集群内访问nginx服务

pod生成后，会从CNI上DHCP获取一个cluster IP地址。

运行命令: kubectl get pods -o wide

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-40.png)

但是这个地址只能在集群内可以访问，无法通过外部网络访问。

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-41.png)

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-42.png)

访问pod的nginx服务

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-43.png)

### 11.5 集群内通过Service实现负载均衡并访问nginx服务

Service可以通过4种方式来为pod、deployment提供服务发现：

|Service方式                              |说明                                |
|------------------------------------|------------------------------------|
|ClusterIP                              |默认方式，通过Cluster VIP来访问                                |
|NodePort                              |通过节点地址上端口映射来访问                                |
|LoadBalancer-OnPremise                             |OnPremise等同于NodePort（除非使用额外的方案例如IPAM Controller或metal LB来分配地址）                                |
|LoadBalancer-Cloud Provider Kubernetes Clusters                              |Azure Load Balancer, AWS Elastic Load Balancer, GCP Load Balancer等云端LB方式                                |
|External Name                              |映射到DNS解析                                |

运行命令: kubectl expose deployment citrix-lab01-nginx-deployment --type=ClusterIP --name=citrix-lab01-service-clusterip 

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-44.png)

### 11.6 集群外通过Service实现负载均衡并访问nginx服务

运行命令: kubectl expose deployment citrix-lab01-nginx-deployment --type=NodePort --name=citrix-lab01-service-nodeport --port=81 --target-port=80

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-45.png)

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-46.png)


至此，我们完成了Kubernetes集群、下载Citrix CPX镜像和部署测试了微应用。在下一个实验里面，我们会利用这个环境来验证Citrix容器解决方案。





















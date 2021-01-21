# LAB01: 安装配置Kubernetes环境

## 更新时间
2021.01.21

## 1. 实验拓扑
![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/101-lab01-topology.png)

## 2. 准备工作
Kubernetes V1.19版本兼容性(1.17版本开始支持Docker 19.03)：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.17.md

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
|Host |IP Address(MGMT) |Gateway |CNI |IP Address(Control-Plane) |
|------------------------------------|------------------------------------|------------------------------------|------------------------------------|------------------------------------|
|Master |192.168.204.11/24 |192.168.201.254 |Weave |192.168.204.11/24 |
|Worker1 |192.168.204.12/24 |192.168.201.254 |Weave |192.168.204.12/24 |
|Worker2 |192.168.204.13/24 |192.168.201.254 |Weave |192.168.204.13/24 |


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

运行命令

    sudo timedatectl set-timezone Asia/Shanghai
    sudo timedatectl set-ntp true
    timedatectl

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-06.png)

编辑本地Host信息，静态绑定集群内所有Host信息

运行命令

    sudo vi /etc/hosts

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-07.png)

关闭Swap：Kubernetes是一个用于大规模运行的分布式系统。在大量机器上运行大量容器时，您需要可预测性和一致性。禁用Swap是正确的方法。

运行命令

    sudo swapoff -a
    sudo vi /etc/fstab

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-08.png)

更新系统已安装的软件包

运行命令

    sudo apt-get update -y
    sudo apt-get upgrade -y
    lsb_release -a

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-09.png)

Linux系统配置完成。

## 7. Master，Worker上安装Docker和Kubernetes

移除当前已安装的Docker相关组件

运行命令

    sudo apt-get remove docker docker-engine docker.io containerd runc -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-10.png)

安装Docker必要组件

运行命令

    sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-11.png)

添加Docker所需要的GPG Key

运行命令

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-12.png)

验证现在是否拥有指纹密钥：9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88

运行命令

    sudo apt-key fingerprint 0EBFCD88

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-13.png)

添加Docker仓库

运行命令

    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-14.png)

更新软件包列表

运行命令

    sudo apt-get update -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-15.png)

查看Docker版本信息

运行命令

    sudo apt-cache madison docker-ce

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-16.png)

指定安装Docker 19.03版本

运行命令

    sudo apt-get install docker-ce=5:19.03.14~3-0~ubuntu-bionic docker-ce-cli=5:19.03.14~3-0~ubuntu-bionic containerd.io -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-17.png)

检查Docker版本

运行命令

    sudo docker version

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-18.png)

修改Docker cgroup driver，从默认的cgroupfs改为systemd

运行命令

    cat <<EOF | sudo tee /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
      "max-size": "100m"
    },
    "storage-driver": "overlay2"
    }
    EOF
    sudo cat /etc/docker/daemon.json

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-21.png)

运行命令

    sudo mkdir -p /etc/systemd/system/docker.service.d
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    sudo systemctl status docker
    sudo systemctl enable docker

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-22.png)

使用国内阿里云作为Kubernetes源，添加Kubernetes的GPG Key

运行命令

    curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-19.png)

使用国内阿里云作为Kubernetes源，添加Kubernetes仓库

运行命令

    sudo add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-20.png)

更新软件包列表

运行命令

    sudo apt-get update -y

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-23.png)

安装kubectl, kubeadm, kubectl 1.19.7版本

运行命令

    sudo apt-get install kubelet=1.19.7-00 kubeadm=1.19.7-00 kubectl=1.19.7-00 -y
    kubelet --version

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-24.png)

完成Kubernetes安装。

## 8. Master上Kubernetes配置

初始化Kubernetes集群

运行命令

    sudo kubeadm init --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --pod-network-cidr=192.168.204.0/24 --apiserver-advertise-address=192.168.204.11

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-25.png)

注意：集群初始化成功后，保存输出信息，会用于Worker加入集群

运行命令

    sudo mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-26.png)

由于没有安装网络插件，Node状态为NotReady

运行命令

    kubectl get nodes

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-27.png)

安装网络插件Weave

运行命令

    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-28.png)

等待约30秒后，检查集群状态

运行命令

    kubectl get nodes

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-29.png)

Master节点配置完成。

## 9. Worker上Kubernetes配置

输入刚才从Master节点上保存的命令

例如，运行命令

    sudo kubeadm join 192.168.204.11:6443 --token 5tiulq.1mcaes63v935n7v2 \
        --discovery-token-ca-cert-hash sha256:ddd73bd3e2fca1b5799ea070af555285f18964a59125a02f7dfd305508206d80

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-30.png)

回到Master节点命令行，检查集群状态

运行命令

    kubectl get nodes -o wide

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-new-31.png)

至此为止，Kubernetes集群已经搭建完成，我们部署了一个Master节点和两个Worker节点。

# LAB01: 安装配置Kubernetes环境

## 1. 实验拓扑
TBD.

## 2. 准备工作
Kubernetes V1.16版本兼容性：https://v1-16.docs.kubernetes.io/docs/setup/release/notes/

Citrix ADC(CPX) 版本兼容性：https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/support-matrix/

Citrix ADC(CPX) V13.0官方文档：https://docs.citrix.com/en-us/citrix-adc-cpx/13/about.html

注册Docker免费帐号：https://hub.docker.com/

Ubuntu Server下载：https://ubuntu.com/download/server

## 3. Kubernetes集群信息
|Host                              |Linux                                |Docker                                |Kubernetes                                |
|------------------------------------|------------------------------------|------------------------------------|------------------------------------|
|Master                              |Ubuntu Server 18.04.3                                |18.09                                |1.16                                |
|Worker1                              |Ubuntu Server 18.04.3                                |18.09                                |1.16                                |
|Worker2                              |Ubuntu Server 18.04.3                                |18.09                                |1.16                                |

## 4. Kubernetes Cluster Networking信息
TBD.

## 5. Citrix ADC信息
TBD.

## 6. Linux安装配置
### 6.1 硬件资源

    CPU: 2 vCPUs
    Memory: 2 GB
    Disk: 20 GB
    
### 6.2 Master, Worker上Linux系统安装

选择系统默认语言为English

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-01.png)

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-02.png)

配置主机静态IP地址

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-03.png)

创建管理员帐号

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-04.png)

安装SSH服务

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-05.png)

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

![LAB01: Setup Kubernetes](https://github.com/yazshen/citrix-adc-kubernetes/blob/master/images/lab01-setup-kubernetes-09.png)





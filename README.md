# kubernetes-install
guild for kubernetes install

Kubernetes 1.18.2 安装



## 第一章： Linux 系统准备



提前准备3台CentOS的虚拟机。网络连接模式全部是NAT



### 1. 设置主机名

Master1 

```shell
[root@localhost ~]# hostnamectl set-hostname master1
[root@localhost ~]# cat /etc/hostname
```



Node1

```shell
[root@localhost ~]# hostnamectl set-hostname node1
[root@localhost ~]# cat /etc/hostname
```



Node2

```shell
[root@localhost ~]# hostnamectl set-hostname node2
[root@localhost ~]# cat /etc/hostname
```



### 2. 配置主机IP



#### 修改以下的参数

BOOTPROTO=static

PEERDNS=no

#### 增加以下参数

IPADDR=192.168.56.50
NETMASK=255.255.255.0
GATEWAY=192.168.56.2



#### Master1 的网络配置

```shell
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=a8dcdd74-f28b-43c3-b479-3a1dab1df29e
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.56.50
NETMASK=255.255.255.0
GATEWAY=192.168.56.2
DNS1=192.168.56.2
```



#### Node1的网络配置

```shell
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=5b51b615-afa1-4208-bf30-2f504ebe6724
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.56.51
NETMASK=255.255.255.0
GATEWAY=192.168.56.2
DNS1=192.168.56.2
```



Node2的网络配置

```shell
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=3d171cb1-99d4-48f7-bdf5-ab71e8e28292
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.56.52
NETMASK=255.255.255.0
GATEWAY=192.168.56.2
DNS1=192.168.56.2
```



### 3. 修改 /etc/hosts 文件，增加本机主机名解析记录：



3台主机都要相同的配置

```
[root@master ~]#  cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.56.50 master1
192.168.56.51 node1 
192.168.56.52 node2 
```



```
[root@master ~]#  scp /etc/hosts node1:/etc/hosts
[root@master ~]#  scp /etc/hosts node2:/etc/hosts
```



### 4. 修改 DNS 名称解析：

3台主机都要相同的配置, 如果步骤2已经配置过DNS1=192.168.56.2， 则可以跳过次步骤

```shell
[root@master ~]# cat /etc/resolv.conf
# Generated by NetworkManager
search example.com
nameserver 192.168.56.2
```



```
[root@master ~]#  scp /etc/resolv.conf node1:/etc/resolv.conf
[root@master ~]#  scp /etc/resolv.conf node2:/etc/resolv.conf
```





### 5. 设置NetworkManager 和 firewalld 服务不开机自启动

3个节点都要做相同的设置

```shell
[root@master1 ~]# systemctl disable NetworkManager
[root@master1 ~]# systemctl disable firewalld
[root@master1 ~]# systemctl stop firewalld
[root@master1 ~]# firewall-cmd --state
not running
```



```shell
[root@master1 ~]# yum -y install iptables-services && systemctl start iptables && systemctl enable iptables && iptables -F && service iptables save
```





### 6. 重启网络

```shell
[root@master1 ~]# systemctl restart network
```



```shell
[root@node1 ~]# systemctl restart network
```



```shell
[root@node2 ~]# systemctl restart network
```



重启网络后测试网络的连通性

```shell
# 如果baidu可以ping通，则说明网络已经连通。
ping baidu.com

```





###  7. 配置 YUM 源和常用命令

```shell
# 配置YUM源
[root@master ~]# rpm -ivh https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
```



### 7. 安装常用软件

```shell
[root@master ~]# yum install -y conntrack ntpdate ntp ipvsadm jq iptables curl systat libsecommp wget vim net-tools git

```



### 8. 关闭 SELinux 、更新软件包后重启：

```shell
[root@master ~]# vim /etc/sysconfig/selinux

```

```properties
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

```



配置完成后重启机器

```shell
[root@master ~]# reboot 

```

验证启动后状态：

```shell
[root@master ~]# getenforce
Disabled

```



### 10. 永久关闭swap分区

```shell
[root@master ~]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Sat Apr 11 03:16:20 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=54d9fb19-2ca6-4879-8bbc-906a5c810bc9 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

```



注释掉/dev/mapper/centos-swap



重启机器之前

```shell
[root@master ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1819          83        1601           9         134        1580
Swap:          2047           0        2047


```



重启机器

```shell
[root@master ~]# reboot

```



重启机器之后

```shell
[root@master ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1819         100        1576           9         142        1558
Swap:             0           0           0


```



或者

```
swapoff -a && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

setenforce 0 && sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```



### 11. 调整内核参数，

```
cat > kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF

cp kubernetes.conf /etc/sysctl.d/kubernetes.conf
sysctl -p /etc/sysctl.d/kubernetes.conf
```





### 12. 调整时区

```shell
# 设置系统时区为 中国/上海
timedatectl set-timezone Asia/Shanghai

# 将当前的UTC时间写入硬件时钟
timedatectl set-local-rtc 0

# 重启依赖于系统时间的服务
systemctl restart rsyslog
systemctl restart crond
```





### 13. 关闭系统不需要服务

```shell
[root@master ~]# systemctl stop postfix && systemctl disable postfix

```





### 14. 设置rsyslogd和systemd journald

```
mkdir /var/log/journal
mkdir /etc/systemd/journal.conf.d
cat > /etc/systemd/journal.conf.d/99-prophet.conf <<EOF
[Journal]
Storage=persistent
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000
SystemMaxUse=10G
SystemMaxFileSize=200M
MaxRetentionSec=2week
ForwardToSyslog=no
EOF

systemctl restart systemd-journald
```



### 15. 升级系统内核为4.4

```shell
[root@master ~]# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

# 安装完成后检查 /boot/grub2/grub.cfg中对应内核menuentry中是否包含initrd16配置， 如果没有，再安装一次

[root@master ~]# yum --enablerepo=elrepo-kernel install -y kernel-lt

# 设置开机从新内核启动
[root@master ~]# grub2-set-default 'CentOS Linux (4.4.189-1.el7.elrepo.x86_64) 7 (Core)' && reboot


```



## 第二章： kubeadm部署安装



### 1. kube-proxy 开启ipvsde 前置条件

```
modprobe br_netfilter

cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```



### 2. 安装docker

```
yum install -y yum-utils device-mapper-persistent-data lvm2
 
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum update -y && yum install -y docker-ce
 
mkdir /etc/docker

cat > /etc/docker/daemon.json <<EOF
{
   "exec-opts":["native.cgroupdriver=systemd"],
   "log-driver": "json-file", 
   "log-opts": {
     "max-size": "100m"
   }
}
EOF
 
 mkdir -p /etc/systemd/system/docker.service.d
 
 systemctl daemon-reload && systemctl restart docker && systemctl enable docker
 
```



安装完成后重启

```
grub2-set-default 'CentOS Linux (4.4.189-1.el7.elrepo.x86_64) 7 (Core)' && reboot

```



重启结束后再次确认内核版本

```
uname -r

```



### 3. 安装Kubeadm (主从配置)

查看kubeadm kubelet kubectl 版本信息

```
yum list kubeadm.x86_64 --showduplicates | sort -r
```



```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum -y install kubeadm-1.18.2 kubectl-1.18.2 kubelet-1.18.2

systemctl daemon-reload && systemctl enable kubelet.service


检查kubelet的状态
systemctl status kubelet.service
```





查看集群中使用的镜像

```shell
[root@master1 ~]# kubeadm config images list
W0418 12:19:56.727453    4623 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
k8s.gcr.io/kube-apiserver:v1.18.2
k8s.gcr.io/kube-controller-manager:v1.18.2
k8s.gcr.io/kube-scheduler:v1.18.2
k8s.gcr.io/kube-proxy:v1.18.2
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7

```



### 4. 下载镜像



查看集群中使用的镜像

```shell
[root@master1 ~]# kubeadm config images list
W0418 12:19:56.727453    4623 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
k8s.gcr.io/kube-apiserver:v1.18.2
k8s.gcr.io/kube-controller-manager:v1.18.2
k8s.gcr.io/kube-scheduler:v1.18.2
k8s.gcr.io/kube-proxy:v1.18.2
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7

```



由于google的镜像仓库被墙， 而集群中又需要google的kubernetes镜像， 所不能直接下载， 需要间接下载。 



创建一个txt文档。

```shell
# 3台主机都要做
vim img_list.txt

#!/bin/bash

docker pull gotok8s/kube-apiserver:v1.18.2
docker pull gotok8s/kube-controller-manager:v1.18.2
docker pull gotok8s/kube-scheduler:v1.18.2
docker pull gotok8s/kube-proxy:v1.18.2
docker pull gotok8s/pause:3.2
docker pull gotok8s/etcd:3.4.3-0
docker pull gotok8s/coredns:1.6.7
docker pull gotok8s/kubernetes-dashboard-amd64:v1.10.1
docker pull gotok8s/tiller:v2.14.2
docker pull gotok8s/defaultbackend-amd64:1.5


docker tag gotok8s/kube-apiserver:v1.18.2              k8s.gcr.io/kube-apiserver:v1.18.2
docker tag gotok8s/kube-controller-manager:v1.18.2     k8s.gcr.io/kube-controller-manager:v1.18.2
docker tag gotok8s/kube-scheduler:v1.18.2              k8s.gcr.io/kube-scheduler:v1.18.2
docker tag gotok8s/kube-proxy:v1.18.2                  k8s.gcr.io/kube-proxy:v1.18.2
docker tag gotok8s/pause:3.2                           k8s.gcr.io/pause:3.2
docker tag gotok8s/etcd:3.4.3-0                        k8s.gcr.io/etcd:3.4.3-0 
docker tag gotok8s/coredns:1.6.7                       k8s.gcr.io/coredns:1.6.7 
docker tag gotok8s/kubernetes-dashboard-amd64:v1.10.1  k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
docker tag gotok8s/tiller:v2.14.2                      k8s.gcr.io/tiller:v2.14.2
docker tag gotok8s/defaultbackend-amd64:1.5            k8s.gcr.io/defaultbackend-amd64:1.5


docker rmi gotok8s/kube-apiserver:v1.18.2
docker rmi gotok8s/kube-controller-manager:v1.18.2
docker rmi gotok8s/kube-scheduler:v1.18.2
docker rmi gotok8s/kube-proxy:v1.18.2
docker rmi gotok8s/pause:3.2
docker rmi gotok8s/etcd:3.4.3-0
docker rmi gotok8s/coredns:1.6.7
docker rmi gotok8s/kubernetes-dashboard-amd64:v1.10.1
docker rmi gotok8s/tiller:v2.14.2
docker rmi gotok8s/defaultbackend-amd64:1.5

```



运行img_list.txt

```shell
sh ./img_list
```





### 5. 初始化主节点

```yaml
kubeadm config print init-defaults > kubeadm-config.yaml

localAPIEndpoint:
  advertiseAddress: 192.168.56.50
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: master1
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.18.2
networking:
  dnsDomain: cluster.local
  podSubnet: "10.244.0.0/16"
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: kubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
```



```yaml
# 主要修改的内容
advertiseAddress: 192.168.56.50
kubernetesVersion: v1.18.2
podSubnet: "10.244.0.0/16"

---

apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs

```

```shell
# 运行初始化命令
kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log

### 切记一定要用 tee kubeadm-init.log， 否则下面的信息需要手动保存。
```

```
[root@master1 ~]# kubeadm init --config=kubeadm-config.yaml | tree kubeadm-init.log
-bash: tree: command not found
error converting YAML to JSON: yaml: line 14: mapping values are not allowed in this context
To see the stack trace of this error execute with --v=5 or higher
[root@master1 ~]# kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log
error converting YAML to JSON: yaml: line 14: mapping values are not allowed in this context
To see the stack trace of this error execute with --v=5 or higher
[root@master1 ~]# vim kubeadm-config.yaml 
[root@master1 ~]# vim kubeadm-config.yaml 
[root@master1 ~]# kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log
W0418 03:54:58.400601    2597 strict.go:47] unknown configuration schema.GroupVersionKind{Group:"kubeproxy.config.k8s.io", Version:"v1alpha1", Kind:"kubeProxyConfiguration"} for scheme definitions in "k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/scheme/scheme.go:31" and "k8s.io/kubernetes/cmd/kubeadm/app/componentconfigs/scheme.go:28"
no kind "kubeProxyConfiguration" is registered for version "kubeproxy.config.k8s.io/v1alpha1" in scheme "k8s.io/kubernetes/cmd/kubeadm/app/componentconfigs/scheme.go:28"
To see the stack trace of this error execute with --v=5 or higher
[root@master1 ~]# vim kubeadm-config.yaml 

No manual entry for kubeProxyConfiguration

shell returned 16

Press ENTER or type command to continue
[root@master1 ~]# kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log
W0418 03:56:59.799235    2708 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.22
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
^C
[root@master1 ~]# kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log
W0418 03:59:37.299562    3099 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [master1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.56.50]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [master1 localhost] and IPs [192.168.56.50 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [master1 localhost] and IPs [192.168.56.50 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
W0418 03:59:40.969695    3099 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0418 03:59:40.972583    3099 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 26.006134 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master1 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node master1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.50:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:c1d95c8070b9c61308bbb75e7edb3a34e65272e7d3a1fe7745102021bb4bbcd7 
```





```shell
### 复制以上关键信息

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
[root@master1 ~]# mkdir -p $HOME/.kube
[root@master1 ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master1 ~]# chown $(id -u):$(id -g) $HOME/.kube/config

```



### 5. 安装fannal网络



```shell
# 下载kube-flannel.yml配置文件

wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑坑
# 由于quay.io被墙， 所以把所有的quay.io域名改为国内的域名quay-mirror.qiniu.com， 貌似有2个地方要改

image: quay-mirror.qiniu.com/coreos/flannel:v0.12.0-amd64

kubectl apply -f kube-flannel.yml
```





### 6. 加入集群



node1

```
[root@node1 ~]# kubeadm join 192.168.56.50:6443 --token abcdef.0123456789abcdef \
>     --discovery-token-ca-cert-hash sha256:c1d95c8070b9c61308bbb75e7edb3a34e65272e7d3a1fe7745102021bb4bbcd7
W0418 12:20:39.922907    3119 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.



```





node2

```
[root@node2 ~]# kubeadm join 192.168.56.50:6443 --token abcdef.0123456789abcdef \
>     --discovery-token-ca-cert-hash sha256:c1d95c8070b9c61308bbb75e7edb3a34e65272e7d3a1fe7745102021bb4bbcd7
W0418 12:20:50.044268    3117 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.


```





### 7. 节点使用kubectl 无法查看node信息



坑坑坑坑

error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable

```shell
# 在/etc/profile末尾增加

export KUBECONFIG=/etc/kubernetes/kubelet.conf

# 添加完后执行

source /etc/profile
```





## 第三章： kubernetes-dashboard 安装



官方的github

https://github.com/kubernetes/dashboard 



Docker hub

https://hub.docker.com/r/kubernetesui/dashboard 



### 1. Pull Images

#### Kubernetes Dashboard

```
docker pull kubernetesui/dashboard:v2.0.0-rc7

```

#### Metrics Scraper

```
docker pull kubernetesui/metrics-scraper:v1.0.4
```



### 2. Installation

Before installing the new beta, remove the previous version by deleting its namespace:

```
kubectl delete ns kubernetes-dashboard
```



update dashborad.yaml to master /root

Then deploy new beta:

```shell
[root@master1 ~]# kubectl apply -f recommended.yaml 
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```



```shell
[root@master1 k8s-test]# kubectl get pod -n kubernetes-dashboard
NAME                                        READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-dc6947fbf-jrppj   1/1     Running   0          43m
kubernetes-dashboard-5d4dc8b976-nlblb       1/1     Running   0          43m

```





### 3. 本地访问

```shell
[root@master1 ~]# kubectl proxy
Starting to serve on 127.0.0.1:8001

然后访问即可：http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```



### 4. 验证登录



 查看Token，然后使用token登录。 

```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep kubernetes-dashboard-token | awk '{print $1}')
```





## 第四章： kubernetes 镜像打包与导入



### 镜像导出： 

```

docker save -o kube-apiserver.tar k8s.gcr.io/kube-apiserver:v1.18.2
docker save -o kube-controller-manager.tar k8s.gcr.io/kube-controller-manager:v1.18.2
docker save -o kube-scheduler:v1.18.2.tar k8s.gcr.io/kube-scheduler:v1.18.2
docker save -o kube-proxy.tar k8s.gcr.io/kube-proxy:v1.18.2
docker save -o pause.tar k8s.gcr.io/pause:3.2
docker save -o etcd.tar k8s.gcr.io/etcd:3.4.3-0 
docker save -o coredns.tar k8s.gcr.io/coredns:1.6.7 
docker save -o kubernetes-dashboard.tar k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
docker save -o tiller.tar k8s.gcr.io/tiller:v2.14.2
docker save -o defaultbackend.tar k8s.gcr.io/defaultbackend-amd64:1.5


```





### 镜像导入：

```
docker load -i kube-apiserver.tar
docker load -i kube-controller-manager.tar
docker load -i kube-scheduler:v1.18.2.tar
docker load -i kube-proxy.tar
docker load -i pause.tar
docker load -i etcd.tar
docker load -i coredns.tar
docker load -i kubernetes-dashboard.tar
docker load -i tiller.tar
docker load -i defaultbackend.tar

```
























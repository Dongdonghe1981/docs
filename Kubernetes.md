# Kubernetes

docker 镜像

```json
  "registry-mirrors": [
    "https://kfwkfulq.mirror.aliyuncs.com",
    "https://2lqq34jg.mirror.aliyuncs.com",
    "https://pee6w651.mirror.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://k7da99jp.mirror.aliyuncs.com/",
    "https://dockerhub.azk8s.cn",
    "https://registry.docker-cn.com"
],
```



查看镜像源，设置root账号密码

```shell
sudo apt-get update

sudo passwd root

#切换到root账号
ubuntu@ubuntu:~$ su
Password:
root@ubuntu:/home/ubuntu#
```

设置允许远程登录Root

```shell
vi /etc/ssh/sshd_config
#添加下面一行，保存
PermitRootLogin yes  

#重启ssh
service ssh restart

```

对虚拟机系统的配置：

- 关闭交换空间：`sudo swapoff -a` 因为交换空间对虚拟机影响效率
- 避免开机启动交换空间：注释掉 `/etc/fstab` 中的 `swap`
- 关闭防火墙：`ufw disable`

使用 APT 安装 Docker


更新软件源

sudo apt-get update

安装所需依赖

sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

安装 GPG 证书

curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

新增软件源信息

sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

再次更新软件源

sudo apt-get -y update

安装 Docker CE 版

sudo apt-get -y install docker-ce
#验证
docker info

配置加速器

对于使用 **systemd** 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

> 多加几个镜像库，这样能下载快些


```json
{
  "registry-mirrors": [
    "http://hub-mirror.c.163.com",
    "https://k7da99jp.mirror.aliyuncs.com/",
    "https://dockerhub.azk8s.cn",
    "https://registry.docker-cn.com"
  ]
}
```

验证加速器是否配置成功：

```shell
sudo systemctl restart docker
docker info
```

到目前为止，镜像虚拟机做好了，关机，拷贝3份，一个Master，两个Slave。

打开拷贝好的Master，修改主机名。

```shell
# 查看当前主机名
hostnamectl

# 使用 hostnamectl 命令修改，其中 kubernetes-master 为新的主机名
hostnamectl set-hostname kubernetes-master
```

修改 cloud.cfg

```shell
# 如果有该文件
vi /etc/cloud/cloud.cfg

# 该配置默认为 false，修改为 true 即可
preserve_hostname: true
```

安装 kubeadm

kubeadm 是 kubernetes 的集群安装工具，能够快速安装 kubernetes 集群。

配置软件源

```shell
# 安装系统工具
apt-get update && apt-get install -y apt-transport-https
# 安装 GPG 证书
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
# 写入软件源；注意：我们用系统代号为 bionic，但目前阿里云不支持，所以沿用 16.04 的 xenial
cat << EOF >/etc/apt/sources.list.d/kubernetes.list
> deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
> EOF
```

安装 kubeadm，kubelet，kubectl

```shell
# 安装
apt-get update  
apt-get install -y kubelet kubeadm kubectl
```

- kubeadm：用于初始化 Kubernetes 集群
- kubectl：Kubernetes 的命令行工具，主要作用是部署和管理应用，查看各种资源，创建，删除和更新组件
- kubelet：主要负责启动 Pod 和容器

配置 kubeadm

```yaml
# 修改配置为如下内容

  # 修改为主节点 IP
  advertiseAddress: 192.168.112.128
  
  # 国内不能访问 Google，修改为阿里云
imageRepository: registry.aliyuncs.com/google_containers

# 修改版本号
kubernetesVersion: v1.18.3

  # 配置成 Calico 的默认网段   v1.18版本没有该项
  podSubnet: "192.168.0.0/16"
  
  # 开启 IPVS 模式   不需要修改
apiVersion: kubeproxy.config.k8s.io/v1alpha1
```

查看和拉取镜像

```shell
# 查看所需镜像列表
kubeadm config images list --config kubeadm.yml
# 拉取镜像
kubeadm config images pull --config kubeadm.yml
```

安装 kubernetes 主节点

```shell
kubeadm init  --kubernetes-version=v1.18.3 --image-repository=registry.aliyuncs.com/google_containers  --apiserver-advertise-address=192.168.112.128 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.1.0.0/16 | tee kubeadm-init.log

#输出中有下面一段，后面子节点加入需要如下命令
kubeadm join 192.168.112.128:6443 --token fj11jf.xb4j0txzovs162h2 \
    --discovery-token-ca-cert-hash sha256:78047791e0f136f81199666fe73123481e8eb25117cdd61b03ce8fb54a91cde5

```

查看启动日志

```shell
journalctl -f  
journalctl -xe
```

解决初始化问题

```shell

vi /etc/docker/daemon.json
# 在daemon.json加入如下配置

{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}

### reload docker
mkdir -p /etc/systemd/system/docker.service.d

systemctl daemon-reload
systemctl restart docker

```

配置 kubectl

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config
#安装calico
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

```

 验证是否成功

```shell
kubectl get node

# 能够打印出节点信息即表示成功
NAME                STATUS     ROLES    AGE     VERSION
kubernetes-master   NotReady   master   8m40s   v1.18.4
```

```shell
#出现下面错误的解决办法
root@kubernetes-master:/usr/local/docker/kubernetes# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
# 出现上面错误的解决办法
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```

### 使用 kubeadm 配置 node 节点

修改IP

修改  `/etc/netplan/50-cloud-init.yaml` 文件 ，桌面版的是`/etc/netplan/01-network-manager-all.yaml`

```yaml
network:
    ethernets:
        ens33:
            addresses: [192.168.112.130/24]
            gateway4: 192.168.112.2
          	nameservers:
              addresses: [192.168.141.2]
    version: 2
```

使得配置文件生效

```shell
netplan apply
```

修改 DNS

vi /etc/systemd/resolved.conf

DNS=114.114.114.114

停止 `systemd-resolved` 服务：`systemctl stop systemd-resolved`

修改 DNS：`vi /etc/resolv.conf`，将 `nameserver` 修改为如 `114.114.114.114` 可以正常使用的 DNS 地址

验证

```shell
root@kubernetes-slave1:/etc# ping www.baidu.com
PING www.a.shifen.com (61.135.169.125) 56(84) bytes of data.
64 bytes from 61.135.169.125 (61.135.169.125): icmp_seq=1 ttl=128 time=104 ms

```

修改主机名

```shell
# 查看当前主机名
hostnamectl

# 使用 hostnamectl 命令修改，其中 kubernetes-slave1 为新的主机名
hostnamectl set-hostname kubernetes-slave1
```

修改cloud.cfg    `参考master节点`

配置软件源  `参考master节点` 

安装 kubeadm，kubelet，kubectl  `参考master节点`

将 slave 加入到集群

```shell
kubeadm join 192.168.112.128:6443 --token fj11jf.xb4j0txzovs162h2 \
    --discovery-token-ca-cert-hash sha256:78047791e0f136f81199666fe73123481e8eb25117cdd61b03ce8fb54a91cde5
    
    
#在主节点上验证
root@kubernetes-master:~# kubectl get node
NAME                STATUS     ROLES    AGE   VERSION
kubernetes-master   NotReady   master   46m   v1.18.4
kubernetes-slave1   NotReady   <none>   7s    v1.18.4
```

如果想删除slave的话

```shell
kubectl delete nodes kubernetes-slave1
```

token 可能过期，下面可以获取token

```shell
root@kubernetes-master:~# kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
fj11jf.xb4j0txzovs162h2   23h         2020-06-26T08:04:53Z   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token

```

如果 token 过期，可以使用 `kubeadm token create` 命令创建新的 token

可以通过 `openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'` 命令查看 sha256 信息

查看pod状态

```shell
kubectl get pod -n kube-system -o wide
```

### Calico

Master不运行pod，在slave（即node）里的Docker运行很多pod，每个pod里运行容器如Nginx、Tomcat等，各个node中的pod需要通信，这时就需要master里的网络插件，如Calico。

参考官方文档安装：https://docs.projectcalico.org/v3.7/getting-started/kubernetes/

在master节点运行

wget https://docs.projectcalico.org/manifests/calico.yaml

```shell
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
```

下载`https://docs.projectcalico.org/manifests/custom-resources.yaml`，修改cidr为之前安装 kubernetes 主节点时，设置好的网段`--pod-network-cidr=10.244.0.0/16`。

```yaml
cidr: 10.244.0.0/16
```

验证

```shell
watch kubectl get pods --all-namespaces

NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-58b656d69f-rnrdc    1/1     Running   0          10m
kube-system   calico-node-76csw                           1/1     Running   0          9m28s
kube-system   calico-node-b7ndk                           1/1     Running   0          10m
kube-system   calico-node-tlc5p                           1/1     Running   0          9m40s
kube-system   coredns-7ff77c879f-g67nd                    1/1     Running   0          19m
kube-system   coredns-7ff77c879f-xgp4j                    1/1     Running   0          19m
kube-system   etcd-kubernetes-master                      1/1     Running   0          19m
kube-system   kube-apiserver-kubernetes-master            1/1     Running   0          19m
kube-system   kube-controller-manager-kubernetes-master   1/1     Running   0          19m
kube-system   kube-proxy-jf7c6                            1/1     Running   0          9m40s
kube-system   kube-proxy-mflfg                            1/1     Running   0          9m28s
kube-system   kube-proxy-pbqfr                            1/1     Running   0          19m
kube-system   kube-scheduler-kubernetes-master            1/1     Running   0          19m

#
root@kubernetes-master:/usr/local/docker/kubernetes# kubectl get node
NAME                STATUS   ROLES    AGE     VERSION
kubernetes-master   Ready    master   19m     v1.18.4
kubernetes-slave1   Ready    <none>   9m54s   v1.18.4
kubernetes-slave2   Ready    <none>   9m42s   v1.18.4

```



如果安装失败，从新安装

1. Master和所有Slave都要`kubeadm reset`，然后重启机器

2. Master上删除 kubectl 配置 `rm -fr ~/.kube/`

3. 所有机器 启用 IPVS

   ```shell
   modprobe -- ip_vs
   modprobe -- ip_vs_rr
   modprobe -- ip_vs_wrr
   modprobe -- ip_vs_sh
   modprobe -- nf_conntrack_ipv4
   ```

4. kubeadm 初始化

### 第一个 Kubernetes 容器

检查组件运行状态

```shell
kubectl get cs

# 输出如下
NAME                 STATUS    MESSAGE             ERROR
# 调度服务，主要作用是将 POD 调度到 Node
scheduler            Healthy   ok                  
# 自动化修复服务，主要作用是 Node 宕机后自动修复 Node 回到正常的工作状态
controller-manager   Healthy   ok                  
# 服务注册与发现
etcd-0               Healthy   {"health":"true"} 
```

检查 Master 状态

```shell
kubectl cluster-info

# 输出如下
# 主节点状态
Kubernetes master is running at https://192.168.112.128:6443
# DNS 状态
KubeDNS is running at https://192.168.112.128:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

检查 Nodes 状态

```shell
kubectl get nodes
# 输出如下，STATUS 为 Ready 即为正常状态
NAME                STATUS   ROLES    AGE   VERSION
kubernetes-master   Ready    master   27m   v1.18.4
kubernetes-slave1   Ready    <none>   17m   v1.18.4
kubernetes-slave2   Ready    <none>   17m   v1.18.4

```

创建资源

```shell
kubectl create deployment nginx --image=nginx
kubectl create deployment nginx2 --image=nginx
```

暴露服务

```shell
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl expose deployment nginx2 --port=80 --type=NodePort
```

检查pod与svc

kubectl get pods,svc

```shell
root@kubernetes-master:/usr/local/docker/kubernetes# kubectl get pods,svc
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx                   1/1     Running   0          45m
pod/nginx-f89759699-fcpd5   1/1     Running   0          78s
pod/nginx2                  1/1     Running   0          31m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.1.0.1     <none>        443/TCP        73m
service/nginx        NodePort    10.1.7.182   <none>        80:32142/TCP   64s
#检查端口是否发布
root@kubernetes-master:/usr/local/docker/kubernetes# netstat -ntlp|grep 32142
tcp        0      0 0.0.0.0:32142           0.0.0.0:*               LISTEN      5607/kube-proxy

#查看已发布的服务
kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.1.0.1     <none>        443/TCP        77m
nginx        NodePort    10.1.7.182   <none>        80:32142/TCP   4m13s


#验证Nginx服务
curl http://192.168.112.128:32142/
```

在slave1和slave2中查看容器

```shell
#slave1
root@kubernetes-slave1:~# docker ps
CONTAINER ID        IMAGE                                               COMMAND                  CREATED             STATUS              PORTS               NAMES
360039127de2        nginx                                               "/docker-entrypoint.…"   13 minutes ago      Up 13 minutes                           k8s_nginx_nginx_default_b5ab4991-538c-4f65-a387-4f696ce4f594_0
#slave2
root@kubernetes-slave2:~# docker ps
CONTAINER ID        IMAGE                                               COMMAND                  CREATED              STATUS              PORTS               NAMES
d90fb02dd55a        nginx                                               "/docker-entrypoint.…"   About a minute ago   Up About a minute                       k8s_nginx2_nginx2_default_0a12f01a-0e0a-485a-86e9-094ad2dc1bfb_0

```

停止服务

```shell
kubectl delete deployment nginx
kubectl delete service nginx
```



## Kubernetes 高可用集群

负载均衡：能够实现轮询

集群：必须要实现数据同步

高可用：一直可以用，必须实现崩溃恢复。

奇数部署：1,3,5,7,9

### 节点配置

| 主机名                  | IP              | 角色   | 系统                | CPU/内存 | 磁盘 |
| ----------------------- | --------------- | ------ | ------------------- | -------- | ---- |
| kubernetes-ha-master-01 | 192.168.112.140 | Master | Ubuntu Server 18.04 | 2核2G    | 20G  |
| kubernetes-ha-master-02 | 192.168.112.141 | Master | Ubuntu Server 18.04 | 2核2G    | 20G  |
| kubernetes-ha-master-03 | 192.168.112.142 | Master | Ubuntu Server 18.04 | 2核2G    | 20G  |
| kubernetes-ha-node-01   | 192.168.112.145 | Node   | Ubuntu Server 18.04 | 2核2G    | 20G  |
| kubernetes-ha-node-02   | 192.168.112.146 | Node   | Ubuntu Server 18.04 | 2核2G    | 20G  |
| kubernetes-ha-node-03   | 192.168.112.147 | Node   | Ubuntu Server 18.04 | 2核2G    | 20G  |
| Kubernetes VIP          | 192.168.112.149 | -      | -                   | -        | -    |

### 制作镜像

基础镜像，从上一节拷贝一个master作为基础镜像，在此基础上，继续做一个master和node的镜像

同步时间

设置时区

```bash
dpkg-reconfigure tzdata
```

选择 **Asia（亚洲）**   ->   选择 **Shanghai（上海）**

时间同步

```shell
# 安装 ntpdate
apt-get install ntpdate

# 设置系统时间与网络时间同步（cn.pool.ntp.org 位于中国的公共 NTP 服务器）
ntpdate cn.pool.ntp.org

# 将系统时间写入硬件时间
hwclock --systohc
```

配置 IPVS

```shell
# 安装系统工具
apt-get install -y ipset ipvsadm

# 配置并加载 IPVS 模块
mkdir -p /etc/sysconfig/modules/
vi /etc/sysconfig/modules/ipvs.modules

# 输入如下内容
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

# 执行脚本，注意：如果重启则需要重新运行该脚本
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4

# 执行脚本输出如下
ip_vs_sh               16384  0
ip_vs_wrr              16384  0
ip_vs_rr               16384  0
ip_vs                 147456  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack_ipv4      16384  3
nf_defrag_ipv4         16384  1 nf_conntrack_ipv4
nf_conntrack          131072  8 xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_ipv4,nf_nat,ipt_MASQUERADE,nf_nat_ipv4,nf_conntrack_netlink,ip_vs
libcrc32c              16384  4 nf_conntrack,nf_nat,raid456,ip_vs
```

配置内核参数

```shell
# 配置参数
vi /etc/sysctl.d/k8s.conf

# 输入如下内容
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
vm.swappiness=0

# 应用参数
sysctl --system

# 应用参数输出如下（找到 Applying /etc/sysctl.d/k8s.conf 开头的日志）
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
* Applying /etc/sysctl.d/10-link-restrictions.conf ...
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/10-lxd-inotify.conf ...
fs.inotify.max_user_instances = 1024
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
kernel.sysrq = 176
* Applying /etc/sysctl.d/10-network-security.conf ...
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_syncookies = 1
* Applying /etc/sysctl.d/10-ptrace.conf ...
kernel.yama.ptrace_scope = 1
* Applying /etc/sysctl.d/10-zeropage.conf ...
vm.mmap_min_addr = 65536
* Applying /usr/lib/sysctl.d/50-default.conf ...
net.ipv4.conf.all.promote_secondaries = 1
net.core.default_qdisc = fq_codel
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
* Applying /etc/sysctl.conf ...
```

修改 cloud.cfg

```shell
vi /etc/cloud/cloud.cfg

# 该配置默认为 false，修改为 true 即可
preserve_hostname: true
```

基础镜像制作完成。

### 修改IP和hostname

master和node每台机器分别做一下配置

配置 IP

编辑 `vi /etc/netplan/50-cloud-init.yaml` 配置文件，修改内容如下

```yaml
network:
    ethernets:
        ens33:
          # 我的 Master 是 140 - 142，Node 是 145 - 147
          addresses: [192.168.112.140/24]
          gateway4: 192.168.141.2
          nameservers:
            addresses: [192.168.141.2]
    version: 2
```

配置主机名

```shell
# 修改主机名
hostnamectl set-hostname kubernetes-ha-master-01

# 配置 hosts
cat >> /etc/hosts << EOF
192.168.112.140 kubernetes-ha-master-01
EOF
```

### 安装 HAProxy + Keepalived

在master上安装 HAProxy + Keepalived

创建 HAProxy 启动脚本 

> 该步骤在 `kubernetes-ha-master-01` 执行

```shell
mkdir -p /usr/local/kubernetes/lb
vi /usr/local/kubernetes/lb/start-haproxy.sh

# 输入内容如下
#!/bin/bash
# 修改为你自己的 Master 地址
MasterIP1=192.168.112.140
MasterIP2=192.168.112.141
MasterIP3=192.168.112.142
# 这是 kube-apiserver 默认端口，不用修改
MasterPort=6443

# 容器将 HAProxy 的 6444 端口暴露出去
docker run -d --restart=always --name HAProxy-K8S -p 6444:6444 \
        -e MasterIP1=$MasterIP1 \
        -e MasterIP2=$MasterIP2 \
        -e MasterIP3=$MasterIP3 \
        -e MasterPort=$MasterPort \
        wise2c/haproxy-k8s
#设置可执行权限 
chmod +x start-haproxy.sh
```

创建 Keepalived 启动脚本

> 该步骤在 `kubernetes-ha-master-01` 执行

```shell
mkdir -p /usr/local/kubernetes/lb
vi /usr/local/kubernetes/lb/start-keepalived.sh

# 输入内容如下
#!/bin/bash
# 修改为你自己的虚拟 IP 地址
VIRTUAL_IP=192.168.112.149
# 虚拟网卡设备名
INTERFACE=ens33
# 虚拟网卡的子网掩码
NETMASK_BIT=24
# HAProxy 暴露端口，内部指向 kube-apiserver 的 6443 端口
CHECK_PORT=6444
# 路由标识符
RID=10
# 虚拟路由标识符
VRID=160
# IPV4 多播地址，默认 224.0.0.18
MCAST_GROUP=224.0.0.18

docker run -itd --restart=always --name=Keepalived-K8S \
        --net=host --cap-add=NET_ADMIN \
        -e VIRTUAL_IP=$VIRTUAL_IP \
        -e INTERFACE=$INTERFACE \
        -e CHECK_PORT=$CHECK_PORT \
        -e RID=$RID \
        -e VRID=$VRID \
        -e NETMASK_BIT=$NETMASK_BIT \
        -e MCAST_GROUP=$MCAST_GROUP \
        wise2c/keepalived-k8s
#设置可执行权限 
chmod +x start-keepalived.sh
```

复制脚本到其它 Master 地址

> 分别在 `kubernetes-ha-master-02` 和 `kubernetes-ha-master-03` 执行创建工作目录命令`

```shell
mkdir -p /usr/local/kubernetes/lb
```

将 `kubernetes-ha-master-01` 中的脚本拷贝至其它 Master

```shell
scp start-haproxy.sh start-keepalived.sh 192.168.112.141:/usr/local/kubernetes/lb
scp start-haproxy.sh start-keepalived.sh 192.168.112.142:/usr/local/kubernetes/lb
```

分别在 3 个 Master 中启动容器（执行脚本）

```shell
sh /usr/local/kubernetes/lb/start-haproxy.sh && sh /usr/local/kubernetes/lb/start-keepalived.sh
```

验证是否成功

查看容器

```shell
docker ps

# 输出如下
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                    NAMES
f50df479ecae        wise2c/keepalived-k8s   "/usr/bin/keepalived…"   About an hour ago   Up About an hour                             Keepalived-K8S
75066a7ed2fb        wise2c/haproxy-k8s      "/docker-entrypoint.…"   About an hour ago   Up About an hour    0.0.0.0:6444->6444/tcp   HAProxy-K8S
```

查看网卡绑定的虚拟 IP

```shell
ip a | grep ens33

# 输出如下
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.112.141/24 brd 192.168.112.255 scope global ens33
    inet 192.168.112.149/24 scope global secondary ens33
```

### 部署 Kubernetes 集群

初始化 Master

创建工作目录并导出配置文件

```bash
# 创建工作目录
mkdir -p /usr/local/docker/kubernetes

# 导出配置文件到工作目录
kubeadm config print init-defaults --kubeconfig ClusterConfiguration > kubeadm.yml
```

 修改配置文件

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  #修改
  advertiseAddress: 192.168.112.140
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  #修改
  name: kubernetes-HA-master-01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: "192.168.112.149:6444"
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
#修改
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
#修改
kubernetesVersion: v1.18.3
networking:
  dnsDomain: cluster.local
  #添加
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
#添加 开启 IPVS 模式
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs

```

kubeadm 初始化

```shell
# kubeadm 初始化
kubeadm init --config=kubeadm.yml | tee kubeadm-init.log

# 配置 kubectl
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 验证是否成功
kubectl get node

#安装calico
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# 验证是否成功
watch kubectl get pods --all-namespaces
```

加入 Master 节点

将上面成功初始化的kubeadm.yml拷贝到master02和master03

```shell
scp -p kubeadm.yml 192.168.112.141:/usr/local/docker/kubernetes
scp -p kubeadm.yml 192.168.112.142:/usr/local/docker/kubernetes
```

从master01中 `kubeadm-init.log` 中获取命令，分别将 `kubernetes-master-02` 和 `kubernetes-master-03` 加入 Master

```shell
You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join 192.168.112.149:6444 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:5dc97791d3bee2739e9a21102411e319b23ee7ca1297496f8969c9fe12175e99 \
    --control-plane
```

如果出现错误，可能需要配置密钥登录

配置master01到master02、master03免密登录

```shell
#创建秘钥
ssh-keygen -t rsa
#将秘钥同步至master02，master03
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.112.141
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.112.142
#免密登陆测试
ssh 192.168.112.142
```

master01分发证书

在master01上运行脚本cert-main-master.sh，将证书分发至master02和master03

cert-main-master.sh脚本内容

```shell
USER=root # customizable
CONTROL_PLANE_IPS="192.168.112.141 192.168.112.142"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
    # Quote this line if you are using external etcd
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
done
```

执行脚本

```shell
./cert-main-master.sh
```

在master02，master03上运行脚本cert-other-master.sh，将证书移至指定目录

cert-other-master.sh脚本内容

```shell
USER=root # customizable
mkdir -p /etc/kubernetes/pki/etcd
mv /${USER}/ca.crt /etc/kubernetes/pki/
mv /${USER}/ca.key /etc/kubernetes/pki/
mv /${USER}/sa.pub /etc/kubernetes/pki/
mv /${USER}/sa.key /etc/kubernetes/pki/
mv /${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
mv /${USER}/front-proxy-ca.key /etc/kubernetes/pki/
mv /${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
# Quote this line if you are using external etcd
mv /${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
```

执行脚本

```shell
./cert-other-master.sh
```

验证集群状态

查看 Node

```bash
kubectl get nodes -o wide
```

查看 Pod

```bash
kubectl -n kube-system get pod -o wide
```

验证高可用

对任意一台 Master 机器执行关机操作

```bash
shutdown -h now
```

在任意一台 Master 节点上查看 Node 状态

```bash
kubectl get node

# 输出如下，除已关机那台状态为 NotReady 其余正常便表示成功
NAME                  	  STATUS   ROLES    AGE   VERSION
kubernetes-ha-master-01   NotReady master   18m   v1.18.4
kubernetes-ha-master-02   Ready    master   17m   v1.18.4
kubernetes-ha-master-03   Ready    master   16m   v1.18.4
```

查看 VIP 漂移，三台master中，只有一台有`secondary`输出，说明是VIP漂移到的机器，其他两台没有

```bash
ip a |grep ens33

# 输出如下
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.112.141/24 brd 192.168.112.255 scope global ens33
    inet 192.168.112.149/24 scope global secondary ens33
```

从master01中 `kubeadm-init.log` 中获取命令，分别在3个Node机器上执行，加入3个node节点

```shell
#Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.112.149:6444 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:5dc97791d3bee2739e9a21102411e319b23ee7ca1297496f8969c9fe12175e99
```

## 通过资源配置运行容器

我们知道通过 `run` 命令启动容器非常麻烦，Docker 提供了 Compose 为我们解决了这个问题。那 Kubernetes 是如何解决这个问题的呢？其实很简单，使用 `kubectl create` 命令就可以做到和 Compose 一样的效果了，该命令可以通过配置文件快速创建一个集群资源对象

创建 YAML 配置文件

以部署 Nginx 为例

 部署 Deployment

在Master01创建一个名为 `nginx-deployment.yml` 的配置文件

 ```yaml
# API 版本号：由 extensions/v1beta1 修改为 apps/v1
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  # 增加了选择器配置
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        # 设置标签由 name 修改为 app
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
 ```

```shell
# 部署
kubectl create -f nginx-deployment.yml

# 删除
kubectl delete -f nginx-deployment.yml
```

验证

```shell
kubectl get pod
NAME                        READY   STATUS              RESTARTS   AGE
nginx-app-d46f5678b-7lxkf   0/1     ContainerCreating   0          22s
nginx-app-d46f5678b-zqntz   0/1     ContainerCreating   0          22s

```

问题1：如果状态是`ErrImagePull`等，在node上执行

```shell
root@kubernetes-HA-node-01:~# kubectl get node
W0626 21:34:01.229855   26520 loader.go:223] Config not found: /etc/kubernetes/admin.conf
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

将master01的admin.conf拷贝到出现上面错误个node上

```shell
scp -p /etc/kubernetes/admin.conf 192.168.112.145:/etc/kubernetes/
```

执行

```shell
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
#重启
source ~/.bash_profile
```

问题2：如果镜像下载失败，需要增加配置node的docker镜像，然后删除刚才的deployment，再重新创建。

### 发布 Service

创建一个名为 `nginx-service.yml` 的配置文件，为k8s暴露外部网络端口

```shell
apiVersion: v1
kind: Service
metadata:
  name: nginx-http
spec:
  ports:
    - port: 80
      # service的80，对应deployment的80，两者才能互通
      targetPort: 80
  type: LoadBalancer
  selector:
    # 标签选择器由 name 修改为 app
    app: nginx
```

```shell
# 部署
kubectl create -f nginx-service.yml

# 删除
kubectl delete -f nginx-service.yml
```

查看service

```shell
root@kubernetes-HA-master-01:/usr/local/docker/kubernetes/yaml# kubectl describe service nginx-http
Name:                     nginx-http
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     LoadBalancer
IP:                       10.109.142.236
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30270/TCP
Endpoints:                10.244.100.68:80,10.244.107.3:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

```

所有节点的30270的端口都可以访问

```shell
#master
curl http://192.168.112.140:30270
curl http://192.168.112.141:30270
#node
curl http://192.168.112.145:30270
curl http://192.168.112.146:30270
```

修改默认的端口范围

Kubernetes 服务的 NodePort 默认端口范围是 30000-32767，在某些场合下，这个限制不太适用，我们可以自定义它的端口范围，操作步骤如下：

编辑 `vi /etc/kubernetes/manifests/kube-apiserver.yaml` 配置文件，增加配置 `--service-node-port-range=2-65535`

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    # 在这里增加配置即可
    - --service-node-port-range=2-65535
    - --advertise-address=192.168.141.150
```

使用 `docker ps` 命令找到 `kube-apiserver` 容器，再使用 `docker restart <ApiServer 容器 ID>` 即可生效。

可以直接用IP访问

```shell
curl http://192.168.112.141
```

## Ingress 统一访问入口

部署 Tomcat

部署 Tomcat 但仅允许在内网访问，我们要通过 Ingress 提供的反向代理功能路由到 Tomcat 之上tomcat.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-app
spec:
  selector:
    matchLabels:
      app: tomcat
  replicas: 2
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-http
spec:
  ports:
    - port: 8080
      targetPort: 8080
  # ClusterIP, NodePort, LoadBalancer
  type: ClusterIP
  selector:
    app: tomcat

```

 安装 Nginx Ingress Controller

`https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image/`

Ingress Controller 有许多种，我们选择最熟悉的 Nginx 来处理请求

使用 Helm

```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx/
helm install --name ingress-nginx ingress-nginx/ingress-nginx
```

验证，状态为`running`要等待几分钟

```shell
kubectl get pods --all-namespaces

ingress-nginx   nginx-ingress-controller-77db54fc46-fs7x4         0/1     running   5          11m

```

部署 Ingress

Ingress 翻译过来是入口的意思，说白了就是个 API 网关（想想之前学的 Zuul 和 Spring Cloud Gateway）

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-web
  annotations:
    # 指定 Ingress Controller 的类型
    kubernetes.io/ingress.class: "nginx"
    # 指定我们的 rules 的 path 可以使用正则表达式
    nginx.ingress.kubernetes.io/use-regex: "true"
    # 连接超时时间，默认为 5s
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "600"
    # 后端服务器回转数据超时时间，默认为 60s
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    # 后端服务器响应超时时间，默认为 60s
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
    # 客户端上传文件，最大大小，默认为 20m
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    # URL 重写
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  # 路由规则
  rules:
  # 主机名，只能是域名，修改为你自己的
  - host: k8s.test.com
    http:
      paths:
      - path:
        backend:
          # 后台部署的 Service Name，与上面部署的 Tomcat 对应
          serviceName: tomcat-http
          # 后台部署的 Service Port，与上面部署的 Tomcat 对应
          servicePort: 8080
```

部署及验证

```shell
#部署
kubectl create -f ingress.yaml
ingress.networking.k8s.io/nginx-web created

#验证
kubectl get ingress
NAME        CLASS    HOSTS          ADDRESS   PORTS   AGE
nginx-web   <none>   k8s.test.com             80      17s


#验证
kubectl get pods -n default -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP              NODE                    NOMINATED NODE   READINESS GATES
ingress-nginx-controller-6d6c44ddf-phkds   1/1     Running   0          18m   10.244.107.25   kubernetes-ha-node-01   <none>           <none>>

```

删除ingress-nginx

```shell
helm delete ingress-nginx
helm ls --all ingress-nginx
helm delete --purge ingress-nginx
helm ls --all ingress-nginx
```



### 安装Helm

1、安装Helm Client

```shell
wget https://get.helm.sh/helm-v2.16.6-linux-amd64.tar.gz
tar -zxvf helm-v2.16.6-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/
helm version
Client: &version.Version{SemVer:"v2.16.6", GitCommit:"dd2e5695da88625b190e6b22e9542550ab503a47", GitTreeState:"clean"}
```

2、安装tiller

```shell
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.6 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts --service-account tiller
```

3、验证

```shell
helm version
#输出如下，版本一致
Client: &version.Version{SemVer:"v2.16.6", GitCommit:"dd2e5695da88625b190e6b22e9542550ab503a47", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.6", GitCommit:"dd2e5695da88625b190e6b22e9542550ab503a47", GitTreeState:"clean"}

```

helm的客户端和服务器版本不一致的情况，删除tiller，重新安装

删除方法

```shell
#step1
kubectl get -n kube-system secrets,sa,clusterrolebinding -o name|grep tiller|xargs kubectl -n kube-system delete
#step2
kubectl get all -n kube-system -l app=helm -o name|xargs kubectl delete -n kube-system
```



### Dashboard

得到配置yaml

```shell
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml

```

修改recommended.yaml

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
#作为NodePort访问，这样每个Node就可以访问了
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      #声明nodePort端口号
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard

```

创建

```
kubectl create -f recommended.yam
```

访问地址`https://192.168.112.145:30001/`

采用 Token 方式登录，为了获取Token，需要以下步骤

创建Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

```

创建ClusterRoleBinding

```yaml
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
  namespace: kubernetes-dashboard
```

得到Token

```shell
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

将Token拷贝到页面上，点击`登录`。

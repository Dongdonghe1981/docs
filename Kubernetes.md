## kubernetes

http://mirrors.aliyun.com/ubuntu

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

```bash
# 更新软件源
sudo apt-get update
# 安装所需依赖
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# 安装 GPG 证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# 新增软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# 再次更新软件源
sudo apt-get -y update
# 安装 Docker CE 版
sudo apt-get -y install docker-ce
#验证
docker info
```

配置加速器

对于使用 **systemd** 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirrors": [
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

在控制平面节点上配置 kubelet 使用的 cgroup 驱动程序

```shell
vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=

---修改后 ---------------
[root@localhost tao]# cat /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=--cgroup-driver=systemd

systemctl daemon-reload
systemctl restart kubelet
```

配置 kubectl

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config
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

#### 使用 kubeadm 配置 slave 节点

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

#### 第一个 Kubernetes 容器

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



### Kubernetes 高可用集群

负载均衡：能够实现轮询

集群：必须要实现数据同步

高可用：一直可以用，必须实现崩溃恢复。

奇数部署：1,3,5,7,9

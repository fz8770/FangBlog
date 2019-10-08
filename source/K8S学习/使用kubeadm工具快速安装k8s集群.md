# 使用kubeadm工具快速安装k8s集群

## 机器配置

master  192.168.252.79
node1  192.168.252.88
node2  192.168.252.89

## 准备工作(三台机器执行)

### 关闭selinux

setenforce 0
sed --follow-symlinks -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

### 关闭防火墙

systemctl stop iptables && systemctl disable iptables
systemctl stop firewalld  && systemctl disable firewalld


### 修改hostname

hostnamectl set-hostname 主机名

### 所有机器添加hosts

echo "192.168.252.79 master1" >> /etc/hosts
echo "192.168.252.88 node1" >> /etc/hosts
echo "192.168.252.89 node2" >> /etc/hosts

### 安装gcc开发工具等
yum install -y  gcc*   

yum install -y vim-enhanced wget  bash-completion  lrzsz ntpdate sysstat iftop htop dstat lsof chkconfig unzip telnet nmap net-tools git bzip2  bind-utils

yum install -y expat-devel  pcre-devel libxml2-devel  openssl openssl-devel  bzip2-devel  libjpeg-devel  libpng-devel   freetype-devel    libXpm-devel  libmcrypt-devel   libaio  libaio-devel  php-mysqlnd   mysql-devel gd-devel  gdbm-devel  glib2-devel  libdb4-devel    libdb4-devel  libicu-devel   libxslt-devel   readline-devel    xmlrpc-c   xmlrpc-c-devel curl-devel yum-utils device-mapper-persistent-data lvm2  conntrack-tools

### 时间同步

ntpdate 1.cn.pool.ntp.org

echo "*/15 * * * * /usr/sbin/ntpdate 1.cn.pool.ntp.org  >/dev/null 2>&1"  >>/var/spool/cron/root

### 升级/重启

yum update -y 
reboot

### 配置免密登录

把公钥传去其他每台机器，当然如果借助ansible或者脚本之类更方便

ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.252.62
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.252.63
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.252.64

### 升级最新稳定版内核(选做)

```shell
#！/bin/bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum history new
yum -y --disablerepo=* --enablerepo=elrepo-kernel install kernel-lt   #lt长期稳定版 ml最新版
grub2-set-default 0
```
reboot


### 机器参数修改

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl -p /etc/sysctl.d/k8s.conf


### 安装docker

yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

####  查看最新的Docker版本
yum list docker-ce.x86_64 --showduplicates |sort -r 

yum makecache fast
yum install  --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y
systemctl start docker && systemctl enable docker

#### 卸载docker
yum remove docker docker-client  docker-client-latest docker-common docker-latest  docker-latest-logrotate \
docker-logrotate docker-selinux docker-engine-selinux docker-engine

### 安装kubeadm和相关工具

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF

yum install -y kubelet kubeadm kubectl --disableexcludes=Kubernetes 或者
yum install kubelet-1.15.4 kubeadm-1.15.4 kubectl-1.15.4 --disableexcludes=kubernetes

systemctl enable kubelet.service && systemctl start kubelet.service

## 安装master

kubeadm config print init-defaults > init-config.yaml #获取默认的初始化参数文件

编辑 init-config.yaml , 定制镜像仓库地址,pod地址范围,service地址范围, 完整内容如下

```shell
apiVersion: kubeadm.k8s.io/v1beta2
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.15.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: "172.16.0.0/16"
  podSubnet: "172.32.0.0/16"
scheduler: {}

```

```shell
kubeadm config images pull  --config=init.config.yaml  #下载镜像
kubeadm init  --config=init.config.yaml #初始化安装
```

安装成功最后底下文字

```shell
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.252.79:6443 --token fu6k4l.5b4jspxqxajsd2la \
    --discovery-token-ca-cert-hash sha256:ee0836e5ac9db0780a190a19c46323d0a32909d758632703b3340a0c30b34228 

```

```shell
#执行操作
[root@master1 ~]# mkdir -p $HOME/.kube
[root@master1 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master1 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 去除master上污点

kubectl taint nodes --all node-role.kubernetes.io/master-

## 安装node

kubeadm config print join-defaults > join-confog.yaml

编辑 join-confog.yaml, apiServerEndpoint值为master的ip, token和tlsBootstrapToken的值来自使用kubeadm init 安装master的最后一段信息

```shell

apiVersion: kubeadm.k8s.io/v1beta2
caCertPath: /etc/kubernetes/pki/ca.crt
discovery:
  bootstrapToken:
    apiServerEndpoint: 192.168.252.79:6443
    token: fu6k4l.5b4jspxqxajsd2la
    unsafeSkipCAVerification: true
  timeout: 5m0s
  tlsBootstrapToken: fu6k4l.5b4jspxqxajsd2la
kind: JoinConfiguration
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: node1
  taints: null

```

```shell
kubeadm join --config=join-defaults.yaml #将node加入集群
```

## 安装网络插件 

wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

修改 kube-flannel.yml, pod范围要和之前init-config.yaml中匹配

```shell
  net-conf.json: |
    {
      "Network": "172.32.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```

```shell
kubectl apply -f kube-flannel.yml 
```
## 验证

```shell

[root@master1 ~]# kubectl get -n kube-system configmap
NAME                                 DATA   AGE
coredns                              1      45m
extension-apiserver-authentication   6      45m
kube-flannel-cfg                     2      36m
kube-proxy                           2      45m
kubeadm-config                       2      45m
kubelet-config-1.15                  1      45m

[root@master1 ~]# kubectl get node
NAME      STATUS   ROLES    AGE   VERSION
master1   Ready    master   45m   v1.15.4
node1     Ready    <none>   43m   v1.15.4
node2     Ready    <none>   35m   v1.15.4

[root@master1 ~]# kubectl get -n kube-system pod
NAME                              READY   STATUS    RESTARTS   AGE
coredns-6967fb4995-dtjcj          1/1     Running   0          45m
coredns-6967fb4995-g27s2          1/1     Running   0          45m
etcd-master1                      1/1     Running   0          44m
kube-apiserver-master1            1/1     Running   0          44m
kube-controller-manager-master1   1/1     Running   0          44m
kube-flannel-ds-amd64-9245r       1/1     Running   0          35m
kube-flannel-ds-amd64-95gnl       1/1     Running   0          37m
kube-flannel-ds-amd64-r29pl       1/1     Running   0          37m
kube-proxy-6rls8                  1/1     Running   0          45m
kube-proxy-c5zs7                  1/1     Running   0          35m
kube-proxy-lhsxn                  1/1     Running   0          43m
kube-scheduler-master1            1/1     Running   0          44m


```

## 其它命令 

```shell
kubeadm reset #重置
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
```

## 参考

https://blog.51cto.com/michaelkang/2432048
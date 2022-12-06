安装前必读：

1. 请不要使用带中文的服务器和克隆的虚拟机
2. 生产环境建议使用二进制安装方式
3. 文档中的IP地址请统一替换，不要一个一个替换！！！
4. 文档中的IP地址请统一替换，不要一个一个替换！！！
5. 文档中的IP地址请统一替换，不要一个一个替换！！！
6. 文档中的IP地址请统一替换，不要一个一个替换！！！

---

### :christmas_tree: 础环境配置

Kubeadm安装方式自1.14版本以后，安装方法几乎没有任何变化，此文档可以尝试安装最新的k8s集群，centos采用的是7.x版本

K8S官网：https://kubernetes.io/docs/setup/

最新版高可用安装：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

表1-1  高可用Kubernetes集群规划

| 主机名            | IP地址                | 说明             |
| ----------------- | --------------------- | ---------------- |
| k8s-master01 ~ 03 | 192.168.200.201 ~ 203 | master节点 * 3   |
| k8s-master-lb     | 192.168.200.236       | keepalived虚拟IP |
| k8s-node01 ~ 02   | 192.168.200.204 ~ 205 | worker节点 * 2   |

| **配置信息** | 备注          |
| ------------ | ------------- |
| 系统版本     | CentOS 7.9    |
| Docker版本   | 20.10.x       |
| Pod网段      | 172.16.0.0/12 |
| Service网段  | 10.10.0.0/16  |

注意事项：

+ 宿主机网段、K8s Service网段、Pod网段不能重复

公有云上搭建VIP是公有云的负载均衡的IP，比如阿里云的内网SLB的地址，腾讯云内网ELB的地址。不需要再搭建keepalived和haproxy

> 所有节点配置hosts，修改/etc/hosts如下：
>

```bash
192.168.200.201 k8s-master01
192.168.200.202 k8s-master02
192.168.200.203 k8s-master03
192.168.200.236 k8s-master-lb # 如果不是高可用集群，该IP为Master01的IP
192.168.200.204 k8s-node01
192.168.200.205 k8s-node02
```

> 所有机器执行

CentOS 7安装yum源如下：

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

#docker 需要用的

yum install -y yum-utils device-mapper-persistent-data lvm2
#docker的yum源
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#docker的源 阿里云自己提供的步骤
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

> 必备工具安装
>

yum install wget jq psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 git -y

> 所有节点关闭防火墙、selinux、dnsmasq、swap。服务器配置如下
>

```shell
systemctl disable --now firewalld 
systemctl disable --now dnsmasq
systemctl disable --now NetworkManager 

setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
```

注意事项：

+ systemctl disable --now dnsmasq  可能会报错没有这个服务
+ systemctl disable --now NetworkManager 公有云不要关闭

> 关闭swap分区

```
swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```

> 安装ntpdate:同步服务器时间，必须同步，涉及到证书的签发`
>

```
rpm -ivh http://mirrors.wlnmp.com/centos/wlnmp-release-centos.noarch.rpm
yum install ntpdate -y
```

所有节点同步时间。时间同步配置如下：

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' >/etc/timezone
ntpdate time2.aliyun.com
# 加入到crontab (CRONTAB -e)
*/5 * * * * /usr/sbin/ntpdate time2.aliyun.com
```

> 所有节点配置limit：
>

一个服务器可能部署很多的容器，如果不设置很大的值，默认1024，在运行一段时间后就无法建立文件句柄

ulimit -SHn 65535

```bash
vim /etc/security/limits.conf
# 末尾添加如下内容
* soft nofile 65536
* hard nofile 131072
* soft nproc 65535
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```

> 免密证书
>

Master01节点免密钥登录其他节点，安装过程中生成配置文件和证书均在Master01上操作，集群管理也在Master01上操作，阿里云或者AWS上需要单独一台kubectl服务器。密钥配置如下：

```
ssh-keygen -t rsa
for i in k8s-master01 k8s-master02 k8s-master03 k8s-node01 k8s-node02;do ssh-copy-id -i .ssh/id_rsa.pub $i;done
```

> 下载安装所有的源码文件

#可能下载不下来，会下载速度慢，如果无法下载就下载：https://gitee.com/dukuan/k8s-ha-install.git

```bash
cd /root/ ; git clone https://github.com/dotbalo/k8s-ha-install.git
[root@k8s-master01 ~]# cd k8s-ha-install/
[root@k8s-master01 k8s-ha-install]# git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/manual-installation
  remotes/origin/manual-installation-v1.16.x
  remotes/origin/manual-installation-v1.17.x
  remotes/origin/manual-installation-v1.18.x
  remotes/origin/manual-installation-v1.19.x
  remotes/origin/manual-installation-v1.20.x
  remotes/origin/manual-installation-v1.20.x-csi-hostpath
  remotes/origin/manual-installation-v1.21.x
  remotes/origin/manual-installation-v1.22.x
  remotes/origin/manual-installation-v1.23.x
  remotes/origin/manual-installation-v1.24.x
  remotes/origin/manual-installation-v1.25.x
  remotes/origin/master
```

所有节点升级系统并重启，此处升级没有升级内核，会单独升级内核：

yum update -y --exclude=kernel*



### :christmas_tree: 内核配置 ###

CentOS7 需要升级内核至4.18+，本地升级的版本为4.19 （生产环境必须升级）

默认3.10:对应k8s和docker而言，版本太低了！

> 在master01节点下载内核：
>

```sh
cd /root
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm
```

从master01节点传到其他节点：

```sh
for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02;do scp kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm $i:/root/ ; done
```

所有节点安装内核

```sh
cd /root && yum localinstall -y kernel-ml*
```

所有节点更改内核启动顺序

```sh
grub2-set-default  0 && grub2-mkconfig -o /etc/grub2.cfg
grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"
```

检查默认内核是不是4.19

```
grubby --default-kernel
```

所有节点安装ipvsadm

+ 可以看到k8s的ipvs的配置时什么样的，路由配置、转发配置

yum install ipvsadm ipset sysstat conntrack libseccomp -y

所有节点配置ipvs模块，在内核4.19+版本nf_conntrack_ipv4已经改为nf_conntrack， 4.18以下使用nf_conntrack_ipv4即可



> 所有节点重启，然后检查内核是不是4.19
>

```sh
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
vim /etc/modules-load.d/ipvs.conf # 加入以下内容
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
```

然后执行:

systemctl enable --now systemd-modules-load.service

开启一些k8s集群中必须的内核参数，所有节点配置k8s内核：

```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
net.ipv4.conf.all.route_localnet = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF
sysctl --system
```

 所有节点配置完内核后，重启服务器，保证重启后内核依旧加载

```sh
reboot
lsmod | grep --color=auto -e ip_vs -e nf_conntrack
```

验证成功：内核加载ok，拍摄快照！！！！



### :christmas_tree:Rutime安装

##### :taco: 安装Containerd #####

如果安装的版本低于1.24，选择Docker和Containerd均可，高于1.24选择Containerd作为Runtime。

在1.24 可能废弃docker，直接装Containerd

1> 所有节点安装docker-ce-20.10：

```
yum install docker-ce-20.10.* docker-ce-cli-20.10.* -y
```

可以无需启动Docker，只需要配置和启动Containerd即可

2> 首先配置Containerd所需的模块（所有节点）：

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

3> 所有节点加载模块：

```
modprobe -- overlay
modprobe -- br_netfilter
```

5> 所有节点，配置Containerd所需的内核：

```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

6> 所有节点加载内核：**sysctl --system**

7> 所有节点配置Containerd的配置文件：

```
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml
```

8> 所有节点将Containerd的Cgroup改为Systemd：

```
vim /etc/containerd/config.toml
```

找到**containerd.runtimes.runc.options**，添加**SystemdCgroup = true****（如果已存在直接修改，否则会报错）**

所有节点将sandbox_image的Pause镜像改成符合自己版本的地址registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6：

默认k8s的镜像地址，改成国内的

所有节点启动Containerd，并配置开机自启动：

```
systemctl daemon-reload
systemctl enable --now containerd
```

所有节点配置crictl客户端连接的运行时位置：

```
cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

查看：

[root@k8s-master01 ~]# ctr images ls
REF TYPE DIGEST SIZE PLATFORMS LABELS



### :christmas_tree: 安装Kubernetes组件

1> 首先在Master01节点查看最新的Kubernetes版本是多少：

```
 yum list kubeadm.x86_64 --showduplicates | sort -r
```

2> 所有节点安装1.24最新版本kubeadm、kubelet和kubectl：

```
 yum install kubeadm-1.24* kubelet-1.24* kubectl-1.24* -y
```

3> 如果选择的是Containerd作为的Runtime，需要更改Kubelet的配置使用Containerd作为Runtime：

```
cat >/etc/sysconfig/kubelet<<EOF
KUBELET_KUBEADM_ARGS="--container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
EOF
```

注意：如果不是采用Containerd作为的Runtime，请不要执行上述命令。



所有节点设置Kubelet开机自启动（由于还未初始化，没有kubelet的配置文件，此时kubelet无法启动，无需管理）：

```
 systemctl daemon-reload
 systemctl enable --now kubelet
```

此时kubelet是起不来的，日志会有报错不影响！



### :christmas_tree: 高可用组件安装 ###

（注意：如果不是高可用集群，haproxy和keepalived无需安装）

公有云要用公有云自带的负载均衡，比如阿里云的SLB，腾讯云的ELB，用来替代haproxy和keepalived，因为公有云大部分都是不支持keepalived的，另外如果用阿里云的话，kubectl控制端不能放在master节点，推荐使用腾讯云，因为阿里云的slb有回环的问题，也就是slb代理的服务器不能反向访问SLB，但是腾讯云修复了这个问题。

1> 所有Master节点通过yum安装HAProxy和KeepAlived：

yum install keepalived haproxy -y

2> 所有Master节点配置HAProxy（详细配置参考HAProxy文档，所有Master节点的HAProxy配置相同）：

mkdir /etc/haproxy
vim /etc/haproxy/haproxy.cfg 

> 默认配置全部删掉！
>

```
global
  maxconn  2000
  ulimit-n  16384
  log  127.0.0.1 local0 err
  stats timeout 30s

defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s

frontend monitor-in
  bind *:33305
  mode http
  option httplog
  monitor-uri /monitor

frontend k8s-master
  bind 0.0.0.0:16443
  bind 127.0.0.1:16443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend k8s-master

backend k8s-master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master01	192.168.200.201:6443  check
  server k8s-master02	192.168.200.202:6443  check
  server k8s-master03	192.168.200.203:6443  check
```

检查haproxy有没有被监听：netstat -ntlp

所有Master节点配置KeepAlived，配置不一样，注意区分 vim /etc/keepalived/keepalived.conf ，注意每个节点的IP和网卡（interface参数）

Master01节点的配置：

mkdir /etc/keepalived

vim /etc/keepalived/keepalived.conf 

+ 注意：网卡名称（ip a）

```
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
rise 1
}
vrrp_instance VI_1 {
    state MASTER
    interface ens32
    mcast_src_ip 192.168.200.201
    virtual_router_id 51
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.200.236
    }
    track_script {
       chk_apiserver
    }
}
```

Master02节点的配置：

```
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
   interval 5
    weight -5
    fall 2  
rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    mcast_src_ip 192.168.200.202
    virtual_router_id 51
    priority 100
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.200.236
    }
    track_script {
       chk_apiserver
    }
}

```

Master03节点的配置：

```
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
 interval 5
    weight -5
    fall 2  
rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    mcast_src_ip 192.168.200.203
    virtual_router_id 51
    priority 100
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.200.236
    }
    track_script {
       chk_apiserver
    }
}
```

所有master节点配置KeepAlived健康检查文件：

```shell
#!/bin/bash

err=0
for k in $(seq 1 3)
do
    check_code=$(pgrep haproxy)
    if [[ $check_code == "" ]]; then
        err=$(expr $err + 1)
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ $err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi
```

chmod +x /etc/keepalived/check_apiserver.sh

启动haproxy和keepalived

```
systemctl daemon-reload
systemctl enable --now haproxy
systemctl enable --now keepalived
```

重要：如果安装了keepalived和haproxy，需要测试keepalived是否是正常的

所有节点：

```SH
ping 192.168.200.236 -c 4
```

如果ping不通且telnet没有出现 ] ，则认为VIP不可以，不可在继续往下执行，需要排查keepalived的问题，比如防火墙和selinux，haproxy和keepalived的状态，监听端口等

所有节点查看防火墙状态必须为disable和inactive：systemctl status firewalld

所有节点查看selinux状态，必须为disable：getenforce

master节点查看haproxy和keepalived状态：systemctl status keepalived haproxy

master节点查看监听端口：netstat -lntp



### :christmas_tree: 集群初始化 ###

官方初始化文档：

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

> 以下操作只在master01节点执行

Master01节点创建kubeadm-config.yaml配置文件如下：

	Master01：（# 注意，如果不是高可用集群，192.168.200.236:16443改为master01的地址，16443改为apiserver的端口，默认是6443，注意更改kubernetesVersion的值和自己服务器kubeadm的版本一致：kubeadm version）

注意!!!!

+ 以下文件内容，宿主机网段、podSubnet网段、serviceSubnet网段不能重复，

以下操作在master01：

vim kubeadm-config.yaml

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.200.201
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock	
  name: k8s-master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - 192.168.200.236
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.200.236:16443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.24.0
networking:
  dnsDomain: cluster.local
  podSubnet: 172.16.0.0/12
  serviceSubnet: 10.103.0.0/16
scheduler: {}
```

如果沾出来格式有问题就：set paste

更新kubeadm文件

```sh
kubeadm config migrate --old-config kubeadm-config.yaml --new-config new.yaml
```

会生成一个new.ymal

将new.yaml文件复制到其他master节点

```
for i in k8s-master02 k8s-master03; do scp new.yaml $i:/root/; done
```

之后所有Master节点提前下载镜像，可以节省初始化时间（其他节点不需要更改任何配置，包括IP地址也不需要更改）：

```sh
kubeadm config images pull --config /root/new.yaml 

[root@k8s-master01 ~]# kubeadm config images pull --config /root/new.yaml
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.24.0
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.24.0
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.24.0
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.24.0
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.7
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.5-0
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.6
```



> 所有节点设置开机自启动kubelet

```SH
systemctl enable --now kubelet
```

+ 如果启动失败无需管理，初始化成功以后即可启动）

> kubeadm初始化

Master01节点初始化，初始化以后会在/etc/kubernetes目录下生成对应的证书和配置文件，之后其他Master节点加入Master01即可：

```
kubeadm init --config /root/new.yaml --upload-certs
```

如果初始化失败，需要重新初始化：

```
kubeadm reset -f ; ipvsadm --clear  ; rm -rf ~/.kube
```

初始化成功以后，会产生Token值，用于其他节点加入时使用，因此要记录下初始化成功生成的token值（令牌值）：

master01:

```sh
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.200.236:16443 --token 7t2weq.bjbawausm0jaxury \
        --discovery-token-ca-cert-hash sha256:cf76ea0cfa8c684be6b367d71d6caa2256a74ce452fac8b6bd0a51171e432f62 \
        --control-plane --certificate-key 455da6422592293025e5ce2a2075a9c995adeeae1a9374ad7a4389de29e6d04c

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.200.236:16443 --token 7t2weq.bjbawausm0jaxury \
        --discovery-token-ca-cert-hash sha256:cf76ea0cfa8c684be6b367d71d6caa2256a74ce452fac8b6bd0a51171e432f62
You have new mail in /var/spool/mail/root

```

```SH
[root@k8s-master01 ~]# kubectl get pod
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

原因：kubectl 连接不到，需要有一个证书的文件，位于 ls /etc/kubernetes/admin.conf ，需要这个文件才能连接到k8s集群

连接方法：

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

或者：可以Master01节点配置环境变量，用于访问Kubernetes集群：

```
cat <<EOF >> /root/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf
EOF
source /root/.bashrc
```

采用初始化安装方式，所有的系统组件均以容器的方式运行并且在kube-system命名空间内，此时可以查看Pod状态



### :christmas_tree: 高可用master ###

##### :taco: master node加入集群 #####

Token没有过期直接执行Join就行了

其他master加入集群，master02和master03分别执行

master使用：

```
  kubeadm join 192.168.200.236:16443 --token 7t2weq.bjbawausm0jaxury \
        --discovery-token-ca-cert-hash sha256:6ccde99bd514e1f7a8796f423ea38c56b4e425ad55ccda9dcf127146408025d7 \
        --control-plane --certificate-key b1cc11c5b9b8a969f19ae836ecc0219c1d6f73f17596f188be3410e8a50a9f89
```

node使用：

```
kubeadm join 192.168.200.236:16443 --token 7t2weq.bjbawausm0jaxury \
        --discovery-token-ca-cert-hash sha256:6ccde99bd514e1f7a8796f423ea38c56b4e425ad55ccda9dcf127146408025d7
```



#### 4.6.2 token过期处理 ####

假设午休吃了个饭或者长时间未操作，token可能会过期！

新建的集群token有效期是24个h，过了时间会提示：This Secret might have expired

查看过期时间：

```bash
[root@k8s-master01 ~]# kubectl get secret -n kube-system
NAME                     TYPE                            DATA   AGE
bootstrap-token-7t2weq   bootstrap.kubernetes.io/token   6      5h42m
```

```sh
[root@k8s-master01 ~]# cat new.yaml | grep token
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
 [root@k8s-master01 ~]# kubectl get secret -n kube-system | grep 7t2weq
bootstrap-token-7t2weq   bootstrap.kubernetes.io/token   6      5h46m
```

加密的，需要解密

时间解密

> 显过期
>

注意：以下步骤是上述init命令产生的Token过期了才需要执行以下步骤，如果没有过期不需要执行，直接join即可`

Token过期后生成新的token：

kubeadm token create --print-join-command

```
[root@k8s-master01 ~]# kubeadm token create --print-join-command
kubeadm join 192.168.200.236:16443 --token 5hqdqy.edcv9fovkjayivaj --discovery-token-ca-cert-hash sha256:6ccde99bd514e1f7a8796f423ea38c56b4e425ad55ccda9dcf127146408025d7
```

可以使用加入node，加入master还需要下面的key

Master需要生成--certificate-key

kubeadm init phase upload-certs --upload-certs

```

[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
7ab87ae92326db2719786e30f8aafde32fbc8e1012597d256879ffba0a6db8c3
```

> 使用的命令：
>

```:taco:
kubeadm join 192.168.200.236:16443 --token 5hqdqy.edcv9fovkjayivaj --discovery-token-ca-cert-hash sha256:6ccde99bd514e1f7a8796f423ea38c56b4e425ad55ccda9dcf127146408025d7 --control-plane --certificate-key 7ab87ae92326db2719786e30f8aafde32fbc8e1012597d256879ffba0a6db8c3
```



##### :taco: k8s集群安装失败故障排查

[root@k8s-node01 ~]# tail -f /var/log/messages

```shell
Aug 13 18:11:21 k8s-node01 kubelet: E0813 18:11:21.608052   19572 kubelet.go:2349] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
```

因为cni网络插件Calico未安装



### :christmas_tree: node节点配置 ###

cni插件 

Node节点上主要部署公司的一些业务应用，生产环境中不建议Master节点部署系统组件之外的其他Pod，测试环境可以允许Master节点部署Pod以节省系统资源。

所有节点初始化完成后，查看集群状态

+ NotReady不影响

###  :christmas_tree: Calco组件安装

支持网络策略

以下步骤只在master01执行

```
cd /root/k8s-ha-install && git checkout manual-installation-v1.24.x && cd calico/
```

修改Pod网段：

```
POD_SUBNET=`cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep cluster-cidr= | awk -F= '{print $NF}'`


sed -i "s#POD_CIDR#${POD_SUBNET}#g" calico.yaml
kubectl apply -f calico.yaml
```

 查看Pod

```SH
[root@k8s-master01 calico]# kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS        AGE
kube-system   calico-kube-controllers-647fd5f6d5-sbdj7   1/1     Running   0               6m33s
kube-system   calico-node-65q2p                          1/1     Running   1 (2m29s ago)   6m33s
kube-system   calico-node-9cng9                          1/1     Running   1 (2m31s ago)   6m33s
kube-system   calico-node-fnff7                          1/1     Running   1 (3m14s ago)   6m33s
kube-system   calico-node-hhnvx                          1/1     Running   1 (2m33s ago)   6m33s
kube-system   calico-node-mn6tv                          1/1     Running   1 (2m42s ago)   6m33s
kube-system   calico-typha-5c5cb6cd95-2d9hx              1/1     Running   0               6m33s
kube-system   coredns-7f74c56694-cjnnk                   1/1     Running   0               21m
kube-system   coredns-7f74c56694-xznww                   1/1     Running   0               21m
kube-system   etcd-k8s-master01                          1/1     Running   2 (8m35s ago)   21m
kube-system   etcd-k8s-master02                          1/1     Running   1 (8m27s ago)   16m
kube-system   etcd-k8s-master03                          1/1     Running   3 (8m4s ago)    14m
kube-system   kube-apiserver-k8s-master01                1/1     Running   2 (8m34s ago)   21m
kube-system   kube-apiserver-k8s-master02                1/1     Running   1 (8m28s ago)   16m
kube-system   kube-apiserver-k8s-master03                1/1     Running   2 (7m51s ago)   14m
kube-system   kube-controller-manager-k8s-master01       1/1     Running   5 (3m8s ago)    21m
kube-system   kube-controller-manager-k8s-master02       1/1     Running   3 (5m32s ago)   16m
kube-system   kube-controller-manager-k8s-master03       1/1     Running   1 (8m30s ago)   14m
kube-system   kube-proxy-2rzdd                           1/1     Running   1 (8m35s ago)   21m
kube-system   kube-proxy-bwlvj                           1/1     Running   1               14m
kube-system   kube-proxy-h8wd6                           1/1     Running   1 (8m27s ago)   18m
kube-system   kube-proxy-ncvzr                           1/1     Running   1 (8m19s ago)   14m
kube-system   kube-proxy-wf6cn                           1/1     Running   1 (8m59s ago)   14m
kube-system   kube-scheduler-k8s-master01                1/1     Running   6 (2m54s ago)   21m
kube-system   kube-scheduler-k8s-master02                1/1     Running   3 (4m41s ago)   16m
kube-system   kube-scheduler-k8s-master03                1/1     Running   1 (8m30s ago)   14m

```



注意：使用kubectl get po -n kube-system -owide 查看pod的IP地址,有些是宿主机ip，有些是pod网段地址

原因：和pod的部署相关

pod采用的是一个hostnetwork的参数，在宿主机监听pod的端口号，如果没采用那就是pod的网段地址

到此 k8s 已经安装完成，还有一些很好用的组件可以安装；



### :christmas_tree: Metrics部署 ###

在新版的Kubernetes中系统资源的采集均使用Metrics-server，可以通过Metrics采集节点和Pod的内存、磁盘、CPU和网络的使用率。

> 安装之前需要个证书，拷贝到其他节点
>

将Master01节点的front-proxy-ca.crt复制到所有Node节点：

```
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node01:/etc/kubernetes/pki/front-proxy-ca.crt
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node(其他节点自行拷贝):/etc/kubernetes/pki/front-proxy-ca.crt
```

> 安装metrics server
>

```sh
cd /root/k8s-ha-install/kubeadm-metrics-server

# kubectl  create -f comp.yaml 
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

查看状态：随机部署在一个node节点

```sh
kubectl get po -n kube-system -l k8s-app=metrics-server

[root@k8s-master01 kubeadm-metrics-server]# kubectl get po -n kube-system -l k8s-app=metrics-server
NAME                              READY   STATUS    RESTARTS   AGE
metrics-server-6c48d8f6d6-9q8kv   1/1     Running   0          81s
```

> Running后：
>

+ kubectl top node
+ kubectl top po -A

```SH
[root@k8s-master01 ~]# kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
k8s-master01   655m         32%    1307Mi          34%
k8s-master02   1411m        70%    1037Mi          27%
k8s-master03   611m         30%    1153Mi          30%
k8s-node01     1358m        67%    509Mi           13%
k8s-node02     129m         6%     463Mi           12%

[root@k8s-master01 ~]# kubectl top po -A
NAMESPACE     NAME                                       CPU(cores)   MEMORY(bytes)
kube-system   calico-kube-controllers-647fd5f6d5-sbdj7   5m           19Mi
kube-system   calico-node-65q2p                          96m          101Mi
kube-system   calico-node-9cng9                          819m         133Mi
kube-system   calico-node-fnff7                          49m          107Mi
kube-system   calico-node-hhnvx                          86m          117Mi
kube-system   calico-node-mn6tv                          71m          108Mi
kube-system   calico-typha-5c5cb6cd95-2d9hx              5m           19Mi
kube-system   coredns-7f74c56694-cjnnk                   40m          28Mi
kube-system   coredns-7f74c56694-xznww                   3m           19Mi
kube-system   etcd-k8s-master01                          36m          77Mi
kube-system   etcd-k8s-master02                          210m         84Mi
kube-system   etcd-k8s-master03                          63m          84Mi
kube-system   kube-apiserver-k8s-master01                245m         367Mi
kube-system   kube-apiserver-k8s-master02                98m          370Mi
kube-system   kube-apiserver-k8s-master03                45m          462Mi
kube-system   kube-controller-manager-k8s-master01       6m           24Mi
kube-system   kube-controller-manager-k8s-master02       2m           22Mi
kube-system   kube-controller-manager-k8s-master03       19m          61Mi
kube-system   kube-proxy-2rzdd                           1m           24Mi
kube-system   kube-proxy-bwlvj                           1m           27Mi
kube-system   kube-proxy-h8wd6                           13m          22Mi
kube-system   kube-proxy-ncvzr                           1m           25Mi
kube-system   kube-proxy-wf6cn                           1m           21Mi
kube-system   kube-scheduler-k8s-master01                9m           21Mi
kube-system   kube-scheduler-k8s-master02                134m         20Mi
kube-system   kube-scheduler-k8s-master03                10m          23Mi
kube-system   metrics-server-6c48d8f6d6-8m5qg            4m           23Mi
```



### :christmas_tree: Dashboard部署 ###

Dashboard用于展示集群中的各类资源，同时也可以通过Dashboard实时查看Pod的日志和在容器中执行一些命令等。

##### :taco: 开启自动生成Token（1.24版本上必做） #####

如果安装的K8s版本以上是1.24以上的（包含1.24），需要修改apiserver的以下配置：

> 所有master节点修改apiserver配置：

```SH
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```



然后在这个文件的command参数的第二行，添加 --feature-gates=LegacyServiceAccountTokenNoAutoGeneration=false （如果有feature-gates参数，直接在后面添加,LegacyServiceAccountTokenNoAutoGeneration=false即可：

同样的方式修改controller-manager：

vim /etc/kubernetes/manifests/kube-controller-manager.yaml



之后重启kubelet，然后按照dashboard即可 （ systemctl restart kubelet）

##### :taco: 安装指定版本dashboard #####

master-01先进入dashboard目录

```
cd /root/k8s-ha-install/dashboard/
kubectl  create -f .
```



##### :taco: 登录dashboard #####

在谷歌浏览器（Chrome）启动文件中加入启动参数，用于解决无法访问Dashboard的问题，参考图1-1：

--test-type --ignore-certificate-errors

> 更改dashboard的svc为NodePort：
>

将ClusterIP更改为NodePort（如果已经为NodePort忽略此步骤）：

查看端口号：kubectl get svc kubernetes-dashboard -n kubernetes-dashboard

```SH
[root@k8s-master01 ~]# kubectl get svc kubernetes-dashboard -n kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.103.39.180   <none>        443:32702/TCP   2m11s
```

根据自己的实例端口号，通过任意安装了kube-proxy的宿主机的IP+端口即可访问到dashboard：

node访问：https://192.168.200.204:32702/

使用谷歌浏览器

访问Dashboard：https://192.168.200.201:32702（请更改32702为自己的端口），选择登录方式为令牌（即token方式），参考图1-2



查看token值：

```SH
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

将token值输入到令牌后，单击登录即可访问Dashboard

 

### :christmas_tree: 一些必须的配置更改 ###

servver两种模式：

1. ipvs
2. iptables

将Kube-proxy改为ipvs模式，因为在初始化集群的时候注释了ipvs配置，所以需要自行修改一下：

> 在master01节点执行

```sh
kubectl edit cm kube-proxy -n kube-system
mode: ipvs
```

> 更新Kube-Proxy的Pod：
>

```sh
kubectl patch daemonset kube-proxy -p "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"`date +'%s'`\"}}}}}" -n kube-system
```

`验证Kube-Proxy模式`

```sh
[root@k8s-master01 ~]# curl 127.0.0.1:10249/proxyMode
ipvs
```



### :christmas_tree: 注意事项

注意：kubeadm安装的集群，证书有效期默认是一年。master节点的kube-apiserver、kube-scheduler、kube-controller-manager、etcd都是以容器运行的。可以通过kubectl get po -n kube-system查看。

```sh
[root@k8s-master01 ~]# cd /etc/kubernetes/manifests/
[root@k8s-master01 manifests]# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

启动和二进制不同的是，

kubelet的配置文件在/etc/sysconfig/kubelet和/var/lib/kubelet/config.yaml，修改后需要重启kubelet进程

其他组件的配置文件在/etc/kubernetes/manifests目录下，比如kube-apiserver.yaml，该yaml文件更改后，kubelet会自动刷新配置，也就是会重启pod。不能再次创建该文件

> kube-proxy的配置在kube-system命名空间下的configmap中，可以通过
>

```
kubectl edit cm kube-proxy -n kube-system
```

进行更改，更改完成后，可以通过patch重启kube-proxy

```
kubectl patch daemonset kube-proxy -p "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"`date +'%s'`\"}}}}}" -n kube-system
```

> Kubeadm安装后，master节点默认不允许部署pod，可以通过以下方式删除Taint，即可部署Pod：

污点:

```
[root@k8s-master01 ~]# kubectl  taint node  -l node-role.kubernetes.io/control-plane node-role.kubernetes.io/master:NoSchedule-
…
[root@k8s-master01 ~]# kubectl  taint node  -l node-role.kubernetes.io/control-plane node-role.kubernetes.io/control-plane:NoSchedule-
…
```





### :christmas_tree:  kubectl命令 ###

常用的命令，查看故障时什么引起的

链接：https://kubernetes.io/zh-cn/docs/reference/kubectl/cheatsheet/

```bash
# get 命令的基本输出
kubectl get services                          # 列出当前命名空间下的所有 services
kubectl get pods --all-namespaces             # 列出所有命名空间下的全部的 Pods
kubectl get pods -o wide                      # 列出当前命名空间下的全部 Pods，并显示更详细的信息
kubectl get deployment my-dep                 # 列出某个特定的 Deployment
kubectl get pods                              # 列出当前命名空间下的全部 Pods
kubectl get pod my-pod -o yaml                # 获取一个 pod 的 YAML
```

```
kubectl get node
kubectl get node -owide
```

##### :taco: 自动补全 #####

敲：kubectl get clusterrole

apt-get  install *bash-completion*

kubectl怎么发现连接到哪个集群：cat /etc/kubernetes/admin.conf 通过这个文件找到api-service地址（或者设置了一个环境变量，读取变量）

没有配置，默认：cat ~/.kube/config 同一个文件

kubectl config view *# 显示合并的 kubeconfig 配置*

 

切换集群：kubectl config use-context NAME

##### :taco: apply create

1. create  如果存在提示存在
2. apply   即使存在，也会重新应用一遍；创建多个两个 -f

在default 命名空间创建一个pod

使用一条命令创建查看yaml

```yaml
kubectl create deployment nginx --image=nginx
kubectl get deploy nginx -oyaml
```

> 重要！！：创建一个deplyment但是不想生成资源
>

 kubectl create deploy nginx2 --image=nginx --dry-run=client -oyaml

```yaml
[root@k8s-master01 ~]# kubectl create deploy nginx --image=nginx --dry-run -oyaml
W0813 23:42:40.009699   33429 helpers.go:636] --dry-run is deprecated and can be replaced with --dry-run=client.
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

重新生成：kubectl delete po  nginx-8f458dc5b-7kxdg

##### :taco: 删除 delete

+ kubectl delete deploy nginx
+ kubectl delete -f nginx.yam

```shell
[root@k8s-master01 ~]# kubectl create -f nginx.yaml
deployment.apps/nginx created
[root@k8s-master01 ~]# kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
nginx-8f458dc5b-tbbtk   1/1     Running   0          7s
[root@k8s-master01 ~]# kubectl delete -f nginx.yaml
deployment.apps "nginx" deleted
[root@k8s-master01 ~]# kubectl get po
No resources found in default namespace.
```

##### :taco: 查看 get 排序

大部分资源都是有命名空间隔离性！！

kubectl api-resources --namespace=true

```
kubectl get svc
kubectl get pod -A -owide
kubectl get node -owide
kubectl get clusterrole
```

查看资排序：

+ 名称：kubectl get po -A --sort-by=.metadata.name

+ 重启次数（切片）：kubectl get pods -A  --sort-by= '.status.containerStatuses[0].restartCount'

查看标签：

+ kubectl get pods -n kube-system --show-labels
+ kubectl get pods -n kube-system --show-labels -l k8s-app=calico-typha



##### :taco: 更新

首先：kubectl create deploy nginx --image=nginx

1. kubectl set
2. apply
3. edit
4. replace  //如果有一个yaml文件，vim后kubectl replace -f  nginx.yaml

kubectl set image  deploy nginx nginx=nginx:v2 



edit保存：shift按住不放+zz

kubectl edit deploy nginx



##### :taco: 日志

```
kubectl logs -f nginx-8f458dc5b-hvwvb
kubectl logs -f  calico-node-bwpjp -n kube-system
kubectl logs -f  calico-node-bwpjp -n kube-system --tail 20
```

使用edit再加一个容器

```
[root@k8s-master01 ~]# kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-577fcfc84c-h555p   2/2     Running   0          26s
```

查看pod中某个容器日志：

+ kubectl logs -f nginx-577fcfc84c-h555p -c redis\



##### :taco: 执行命令 #####

```sh
kubectl exec nginx-577fcfc84c-h555p -- pwd

*单容器场景*
kubectl exec nginx-577fcfc84c-h555p -- pwd

*多容器场景*
kubectl exec nginx-577fcfc84c-h555p -c redis  -- pwd
```

x

直接打开控制台：-ti

+ bash （优先）
+ sh （没有bash使用sh）

##### :taco: top #####

node： kubectl top node 注意：1颗cpu=1000m、1G=1024Mi

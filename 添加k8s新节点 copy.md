## 安装k8s工作节点在centos8.4上

- 安装前配置
   - 配置内核模块

```
    cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF
    modprobe overlay
    modprobe br_netfilter
```

   - 配置前向转发

```
    #Setup required sysctl params, these persist across reboots.
    cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.ipv4.ip_forward                 = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    EOF
    #Apply sysctl params without reboot
    sysctl --system
```
   - 关闭swap
```
    swapoff -a
    comment swap in /etc/fstab
```


- 为应用程序安装一个运行时环境
   - 选择一种CRI, 根据https://kubernetes.io/docs/setup/production-environment/container-runtimes/
   - 安装containerd
    https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd 安装containerd.io 根据https://docs.docker.com/engine/install/centos/

```
    yum install -y yum-utils
    yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    yum -y install 使用--allowrasing docker-ce docker-ce-cli containerd.io docker-compose-plugin
    #今天在centos8.4 上试不用--allowrasing 也可以安装, 主要 yum erase podman containerd.io runc在安装前
```

   - 添加native.cgroupdriver=systemd 在/etc/docker/daemon.json (在RHEL84上略过)
      根据https://kubernetes.io/docs/setup/production-environment/container-runtimes/

```
      {
            "exec-opts": ["native.cgroupdriver=systemd"], 
            "log-driver": "json-file", 
            "log-opts": { 
                    "max-size": "100m" 
                    }, 
            "storage-driver": "overlay2"
        }

      systemctl start docker
```

   - 编辑/etc/containerd/config.toml 加入# 注销disabled_plugins = ["cri"] [centos84 上已经disabled了]
    systemctl restart containerd

- 安装kubernetes 
    根据https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

```  
    cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    exclude=kubelet kubeadm kubectl
    EOF
    yum install kubelet-1.24.0-0 kubeadm-1.24.0-0 kubectl-1.24.0-0 --disableexcludes=kubernetes
```

- 获取token, 在master上运行

```
[root@n39-h79 ~]# kubeadm token create --print-join-command
kubeadm join 10.200.39.79:6443 --token 01duz3.34u69ohvkt5l3smn --discovery-token-ca-cert-hash sha256:25eb0937d62e37b82b40af2decd2d90efba74094c98dc05422c7a5822e43351e 
```

- 安装calcio
    kubectl apply -f calico.yaml
    要注意使用IP_AUTODETECTION_METHOD interface=bond0, 因为机器上有多个网卡，网络，还有VPN, 可以在每个节点上用calcio-node -show-status去调试查看https://projectcalico.docs.tigera.io/reference/node/configuration#ip-autodetection-methods


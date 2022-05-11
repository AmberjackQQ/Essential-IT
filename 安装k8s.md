安装k8s工作节点在centos7.9上
1. 选择一种CRI, 根据https://kubernetes.io/docs/setup/production-environment/container-runtimes/
2. 安装containerd
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
2.1 安装前配置
    cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF
    modprobe overlay
    modprobe br_netfilter

    # Setup required sysctl params, these persist across reboots.
    cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.ipv4.ip_forward                 = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    EOF

    # Apply sysctl params without reboot
    sysctl --system
2.2 安装containerd.io 根据https://docs.docker.com/engine/install/centos/
2.2.1. yum install -y yum-utils
2.2.2. yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
2.2.3. yum -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
2.2.4 添加native.cgroupdriver=systemd 在/etc/docker/daemon.json
      根据https://kubernetes.io/docs/setup/production-environment/container-runtimes/
      {
            "exec-opts": ["native.cgroupdriver=systemd"], 
            "log-driver": "json-file", 
            "log-opts": { 
                    "max-size": "100m" 
                    }, 
            "storage-driver": "overlay2"
        }

      systemctl start docker
2.2.5 编辑/etc/containerd/config.toml 加入# 注销disabled_plugins = ["cri"]
2.2.6 systemctl restart containerd
3.安装kubernetes 
    根据https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management
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
    yum -y install kubelet kubeadm kubectl --disableexcludes=kubernetes
4.获取加入令牌
kubeadm join 10.200.39.79:6443 --token sskfzl.puu6l9jozsve6cj4 --discovery-token-ca-cert-hash sha256:25eb0937d62e37b82b40af2decd2d90efba74094c98dc05422c7a5822e43351e


问题排查
1.现象
https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64/repodata/repomd.xml: [Errno -1] repomd.xml signature could not be verified for kubernetes
方案，关闭 gpgcheck=0 repo_gpgcheck=0


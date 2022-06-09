# Kubernetes（k8s) 简介
  Kubernetes 是用于自动部署，扩展和管理容器化应用程序的开源系统。它将组成应用程序的容器组合成逻辑单元，以便于管理和服务发现。Kubernetes 源自Google 15 年生产环境的运维经验，同时凝聚了社区的最佳创意和实践。
  - Google 每周运行数十亿个容器，Kubernetes 基于与之相同的原则来设计，能够在不扩张运维团队的情况下进行规模扩展。
  - 无论是本地测试，还是跨国公司，Kubernetes 的灵活性都能让你在应对复杂系统时得心应手。
  - Kubernetes 是开源系统，可以自由地部署在企业内部，私有云、混合云或公有云，让您轻松地做出合适的选择。

## 安装k8s工作节点在centos7.9上

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

- 为应用程序安装一个运行时环境
   - 选择一种CRI, 根据https://kubernetes.io/docs/setup/production-environment/container-runtimes/
   - 安装containerd
    https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd 安装containerd.io 根据https://docs.docker.com/engine/install/centos/

```
    yum install -y yum-utils
    yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    yum -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

   - 添加native.cgroupdriver=systemd 在/etc/docker/daemon.json
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

   - 编辑/etc/containerd/config.toml 加入# 注销disabled_plugins = ["cri"]
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
    yum -y install kubelet kubeadm kubectl --disableexcludes=kubernetes
```

- 安装calcio
    
    要注意使用IP_AUTODETECTION_METHOD interface=bond0, 因为机器上有多个网卡，网络，还有VPN, 可以在每个节点上用calcio-node -show-status去调试查看https://projectcalico.docs.tigera.io/reference/node/configuration#ip-autodetection-methods

- 安装ingress-nginx

```
    kubectl apply -f controller-v1.2.0.cloud.deploy.yaml
```

- 安装kubernetes dashboard

```
kubectl apply -f aio/deploy/recommended.yaml
```

- 配置kubernetes dashboard 到ingress-nginx

```
kubectl apply -f dashboard-ingress.yaml
```

   - 建立用户并绑定角色,参考https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

```   
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: admin-user
    namespace: kubernetes-dashboard
    ---
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

   - 获取令牌并登入dashboard

```
    kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"    
```

   - 获取令牌,加入节点

 ```   
    kubeadm join 10.200.39.79:6443 --token sskfzl.puu6l9jozsve6cj4 --discovery-token-ca-cert-hash sha256:25eb0937d62e37b82b40af2decd2d90efba74094c98dc05422c7a5822e43351e
```


- 安装helm 

    https://helm.sh/docs/intro/install/
    Download your desired version
    Unpack it (tar -zxvf helm-v3.0.0-linux-amd64.tar.gz)
    Find the helm binary in the unpacked directory, and move it to its desired destination (mv linux-amd64/helm /usr/local/bin/helm)

- 安装goharbor

https://goharbor.io/docs/2.5.0/install-config/harbor-ha-helm/




# 问题排查
- 出错信息
    https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64/repodata/repomd.xml: [Errno -1] repomd.xml signature could not be verified for kubernetes
    解决方法，关闭 gpgcheck=0 repo_gpgcheck=0
- 出错信息
    Error from server (InternalError): error when creating "dashboard-ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": context deadline exceeded
    解决方法,kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

# CoreDNS调试
bash-4.2# dig @10.7.0.10 -p 53 oatas.veritas.com
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.9 <<>> @10.7.0.10 -p 53 oatas.veritas.com
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached
https://kubernetes.io/docs/tasks/administer-cluster/coredns/



# ingress-nginx 和 nginx-ingress 区别
nginx学习 https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms

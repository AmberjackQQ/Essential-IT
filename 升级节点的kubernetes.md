## 升级一个工作节点kubernetes从1.23到1.24

- 安装
```
yum list --showduplicates kubeadm --disableexcludes=kubernetes
yum install -y kubeadm-1.24.12-0 --disableexcludes=kubernetes
kubeadm upgrade node
yum install -y kubelet-1.24.12-0 kubectl-1.24.12-0 --disableexcludes=kubernetes
systemctl daemon-reload
systemctl restart kubelet
systemctl status kubelet
```

参考链接
https://v1-24.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrade-worker-nodes

## 升级control-panel节点

### 错误1  Error: failed to parse kubelet flag: unknown flag: --network-plugin
    解决办法， 删除--network-plugin=cni在/var/lib/kubelet/kubeadm-flags.env中 ,以及启动参数详情见https://www.wulaoer.org/?p=2652
    
 
 ### 错误2
 ```
[root@n39-h79 ~]# kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0330 20:25:26.016530   12349 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/var/run/dockershim.sock". Please update your configuration!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[preflight] Running pre-flight checks.
[preflight] Some fatal errors occurred:
        [ERROR CoreDNSUnsupportedPlugins]: start version '1.9.2' not supported
        [ERROR CoreDNSMigration]: CoreDNS will not be upgraded: start version '1.9.2' not supported
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```
   解决办法 降低coredns kubectl set image deployment/coredns coredns/coredns:1.8.6
   
参考链接
https://v1-24.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrading-control-plane-nodes
https://www.wulaoer.org/?p=2652

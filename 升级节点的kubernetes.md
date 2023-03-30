升级节点kubernetes从1.23到1.24
yum list --showduplicates kubeadm --disableexcludes=kubernetes
yum install -y kubeadm-1.24.12-0 --disableexcludes=kubernetes
kubeadm upgrade node
yum install -y kubelet-1.24.12-0 kubectl-1.24.12-0 --disableexcludes=kubernetes
systemctl daemon-reload
systemctl restart kubelet
systemctl status kubelet

参考链接
https://v1-24.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrade-worker-nodes

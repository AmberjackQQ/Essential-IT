https://github.com/prometheus-operator/kube-prometheus
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus/
git release-0.11
kubectl apply -f manifest/setup/namespace.yml
配置存储
kubectl apply --server-side -f manifests/setup
kubectl get servicemonitors --all-namespaces
echo $?
kubectl apply -f manifests/

# 配置存储
- export /storage
- exportfs -ra
- 建立nfs-client storage class

    参考https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
    helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
    helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -n monitoring --set nfs.server=10.200.39.3 --set nfs.path=/storage/k8s/nfs
- kubectl apply -f grafana-pvc.yaml

# 设置ingress
- kubectl apply -f grafana-ingress.yaml
  grafana admin/admin default login

## 安装jb jsconnet, 若不定制配置，不需要用这个工具
go install -a github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb@latest
go install github.com/google/go-jsonnet/cmd/jsonnet@latest

https://github.com/prometheus-operator/kube-prometheus/blob/release-0.11/docs/customizations/exposing-prometheus-alertmanager-grafana-ingress.md


# 排错
## 不能ping通 grafana pod ip, 或者 访问 ingress配置的grafana UI, 显示504错误
kubectl -n monitoring delete networkpolicies.networking.k8s.io --all

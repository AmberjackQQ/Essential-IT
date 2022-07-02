https://github.com/prometheus-operator/kube-prometheus
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus/
git release-0.11
kubectl apply --server-side -f manifests/setup
kubectl get servicemonitors --all-namespaces
echo $?
kubectl apply -f manifests/

#配置存储
kind: PersistentVolume
apiVersion: v1
metadata:
  name: prometheus-pv
  namespace: monitoring
spec:
  capacity:
    storage: 1Gi
  nfs:
    server: 10.200.39.3
    path: /storage/prometheus
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-prometheus
  volumeMode: Filesystem
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: prometheus-pvc
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: nfs-prometheus

# 设置ingress
## 安装jb jsconnet, 若不定制配置，不需要用这个工具
go install -a github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb@latest
go install github.com/google/go-jsonnet/cmd/jsonnet@latest

https://github.com/prometheus-operator/kube-prometheus/blob/release-0.11/docs/customizations/exposing-prometheus-alertmanager-grafana-ingress.md

# 排错
## 不能ping通 grafana pod ip, 或者 访问 ingress配置的grafana UI, 显示504错误
kubectl -n monitoring delete networkpolicies.networking.k8s.io --all

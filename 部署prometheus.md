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

# 配置IPMI exporter
kubectl apply -f IPMIExporter-configuration.yaml
kubectl apply -f IPMIExporter-deployment.yaml
https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/additional-scrape-config.md
kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml --dry-run -oyaml  additional-scrape-configs.yaml
kubectl apply -f additional-scrape-configs.yaml -n monitoring
add additionalScrapeConfigs: under spec
    name: additional-scrape-configs
    key: prometheus-additional.yaml  
to prometheus-prometheus.yaml
kubectl apply -f manifests/prometheus-prometheus.yaml 
在k8s dashboard上修改这个secret, 保存之后，在proethemues pod 里面的/etc/prometheus/config_out/prometheus.env.yaml 会自动修改更新

# 部署pushgateway
1. 在k8s dashboard 上修改secret 加入
```
- job_name: pushgateway
  scrape_timeout: 30s
  scrape_interval: 1m
  metrics_path: /metrics
  honor_labels: true
  scheme: http
  static_configs:
  - targets:
    - 10.7.21.159:9091
    labels:
      instance: pushgateway
```
2. 进入prometheus pod shell run 
cat /etc/prometheus/config_out/prometheus.env.yaml 
确认pushgateway job 配上

3. 在node上运行如下命令，reload promethues
[root@harbor prometheus]# curl -X POST http://10.7.37.123:9090/-/reload

4. 查看prometheus target 确认pushgateway target 存在
 http://10.7.37.123:9090/targets?search= 

5. 测试 
[root@harbor prometheus]# echo "some_metric 3.14" | curl --data-binary @- http://10.7.21.159:9091/metrics/job/some_job
6. 访问 pushgateway node http://10.7.21.159:9091/
 ![alt "example"](/assets/img/pushgateway_metrics.png) 
7. 访问 promethues http://10.7.37.123:9090/graph?g0.expr=some_metric&g0.tab=0&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h
 ![alt "example"](/assets/img/promethues_metrics.png)


[root@harbor ~]# kubectl apply -f https://github.com/cncf/demo/blob/master/AddOns/Prometheus/pushgateway.yaml
error: error parsing https://github.com/cncf/demo/blob/master/AddOns/Prometheus/pushgateway.yaml: error converting YAML to JSON: yaml: line 28: mapping values are not allowed in this context


https://github.com/pierDipi/k8s-test-infra/blob/master/config/prow/cluster/pushgateway_deployment.yaml


# 排错
## 不能ping通 grafana pod ip, 或者 访问 ingress配置的grafana UI, 显示504错误
kubectl -n monitoring delete networkpolicies.networking.k8s.io --all

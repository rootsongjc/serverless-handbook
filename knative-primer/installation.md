# Knative 安装

我们在已有的 Kubernetes 集群上安装 Knative，因为 Knative 依赖 Istio，首先我们需要先安装 Istio。

## 安装 Istio

我们安装的 Istio 的基本情况：

- Kubernetes 1.15
- Istio v1.1.7
- 启动 Sidecar 自动注入
- 使用 Helm 安装
- 不启用 SDS

```bash
# Knative v0.9 在 Istio v1.1.7 验证过
export ISTIO_VERSION=1.1.7
curl -L https://git.io/getLatestIstio | sh -
cd istio-${ISTIO_VERSION}

# 安装 Istio CRD
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done

# 创建 istio-system namespace
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: istio-system
  labels:
    istio-injection: disabled
EOF

# 启用 sidecar 注入的模板
helm template --namespace=istio-system \
  --set sidecarInjectorWebhook.enabled=true \
  --set sidecarInjectorWebhook.enableNamespacesByDefault=true \
  --set global.proxy.autoInject=disabled \
  --set global.disablePolicyChecks=true \
  --set prometheus.enabled=false \
  `# 禁用 mixer prometheus adapter，删除 istio 默认的 metrics` \
  --set mixer.adapters.prometheus.enabled=false \
  `# 禁用 mixer policy check，我们的模板里不使用 policy` \
  --set global.disablePolicyChecks=true \
  `# 将 gateway pod 设置为 1 以规避最终一致性/readiness 问题` \
  --set gateways.istio-ingressgateway.autoscaleMin=1 \
  --set gateways.istio-ingressgateway.autoscaleMax=1 \
  --set gateways.istio-ingressgateway.resources.requests.cpu=500m \
  --set gateways.istio-ingressgateway.resources.requests.memory=256Mi \
  `# 多个 pilot replica 便于伸缩` \
  --set pilot.autoscaleMin=2 \
  `# 将 pilot 追踪采样设置为 100%` \
  --set pilot.traceSampling=100 \
  install/kubernetes/helm/istio \
  > ./istio.yaml

# 部署 istio
kubectl apply -f istio.yaml
```

验证 Istio 安装。

```bash
kubectl get pods --namespace istio-system
NAME                                      READY   STATUS      RESTARTS   AGE
istio-citadel-8559955d59-j5xx9            1/1     Running     0          40m
istio-cleanup-secrets-1.1.7-rb9hp         0/1     Completed   0          40m
istio-galley-fd84c8888-8kljq              1/1     Running     0          40m
istio-ingressgateway-8486b5db4f-hhqfg     1/1     Running     0          40m
istio-pilot-7846986bf5-5ql82              2/2     Running     0          40m
istio-pilot-7846986bf5-xgqhk              2/2     Running     0          40m
istio-policy-f7f8c578b-7rz2j              2/2     Running     2          40m
istio-security-post-install-1.1.7-nqm9z   0/1     Completed   0          40m
istio-sidecar-injector-c645cf64-8hfzg     1/1     Running     0          40m
istio-telemetry-575c8d8d66-pb27q          2/2     Running     3          40m
```

## 安装 Knative

我们安装的 Istio 的基本情况：

- Knative v0.9.0

```bash
# 安装 Knative CRD
kubectl apply --selector knative.dev/crd-install=true \
   --filename https://github.com/knative/serving/releases/download/v0.9.0/serving.yaml \
   --filename https://github.com/knative/eventing/releases/download/v0.9.0/release.yaml \
   --filename https://github.com/knative/serving/releases/download/v0.9.0/monitoring.yaml
   
# 再运行一遍 kubectl apply
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.9.0/serving.yaml \
   --filename https://github.com/knative/eventing/releases/download/v0.9.0/release.yaml \
   --filename https://github.com/knative/serving/releases/download/v0.9.0/monitoring.yaml
```

因为 `quay.io`、`gcr.io`、`k8s.gcr.io`、`docker.elastic.co` 等镜像仓库在中国大陆无法访问，位于中国大陆的用户可以使用本书仓库中的 `manifests/knative/0.9` 目录下的 YAML 文件安装，其中以上镜像已替换为 DockerHub 镜像源。

```bash
kubectl apply --selector knative.dev/crd-install=true  -f manifests/knative/0.9
kubectl apply -f manifests/knative/0.9
```

验证 Knative 安装。

```bash
kubectl get pods --namespace knative-serving
kubectl get pods --namespace knative-eventing
kubectl get pods --namespace knative-monitoring
NAME                               READY   STATUS    RESTARTS   AGE
activator-76f486c78-9k7l7          2/2     Running   3          34m
autoscaler-7495bcbc4b-j58n5        2/2     Running   2          34m
autoscaler-hpa-66b55c9688-w2x58    1/1     Running   0          34m
controller-859b7dbb8f-pdwfp        1/1     Running   0          34m
networking-istio-dc5f9b9f8-8qxfh   1/1     Running   0          34m
webhook-8dcd46846-kz78d            1/1     Running   0          34m
NAME                                   READY   STATUS    RESTARTS   AGE
eventing-controller-8466765fbf-djs8v   1/1     Running   0          34m
eventing-webhook-684467d8f5-klb7s      1/1     Running   0          34m
imc-controller-66dfdb6878-8mppw        1/1     Running   0          34m
imc-dispatcher-7db65bf44b-kwwhs        1/1     Running   0          34m
sources-controller-75c57459cc-xttxr    1/1     Running   0          34m
NAME                                  READY   STATUS    RESTARTS   AGE
elasticsearch-logging-0               1/1     Running   0          12m
elasticsearch-logging-1               1/1     Running   0          11m
grafana-85c86fb7b9-8b95g              1/1     Running   0          34m
kibana-logging-85569f954d-fc65f       1/1     Running   0          35m
kube-state-metrics-6fd74d89bf-7kmk6   4/4     Running   0          30m
node-exporter-6z6nk                   2/2     Running   0          34m
node-exporter-hm2gr                   2/2     Running   0          34m
node-exporter-rhc7r                   2/2     Running   0          34m
prometheus-system-0                   1/1     Running   0          34m
prometheus-system-1                   1/1     Running   0          34m
```

## 参考

- [Installing Istio for Knative - knative.dev](https://knative.dev/docs/install/installing-istio/)
- [Installing Knative - knative.dev](https://knative.dev/docs/install/index.html)
- [Performing a Custom Knative Installation - knative.dev](https://knative.dev/docs/install/knative-custom-install/)
# Monitor Operator Helm Chart

Monitor Operator 是一个 Kubernetes 原生的监控栈管理工具，可以自动化部署和管理 Prometheus 和 Grafana。

## 功能特性

- 🚀 **自动化部署**: 一键部署完整的监控栈
- 📊 **监控组件管理**: 支持 Prometheus 和 Grafana 的生命周期管理
- 🔧 **灵活配置**: 支持自定义资源配置和服务暴露方式
- 🛡️ **安全性**: 内置安全最佳实践和 RBAC 配置
- 📈 **高可用**: 支持多副本部署和 Pod 中断预算
- 🔍 **可观测性**: 内置 metrics 暴露和健康检查

## 前置条件

- Kubernetes 1.19+
- Helm 3.2.0+
- 具有集群管理员权限（用于创建 CRD 和 ClusterRole）

## 快速开始

### 1. 添加 Ciliverse Helm 仓库

```bash
# 添加 Ciliverse Charts 仓库
helm repo add ciliverse https://charts.cillian.website

# 更新仓库索引
helm repo update

# 搜索可用的 charts
helm search repo ciliverse
```

### 2. 安装 Monitor Operator

```bash
# 基础安装
helm install monitor-operator ciliverse/monitor-operator

# 安装到指定命名空间
helm install monitor-operator ciliverse/monitor-operator \
  --namespace monitoring \
  --create-namespace

# 使用自定义配置安装
helm install monitor-operator ciliverse/monitor-operator \
  --set deployment.replicas=2 \
  --set metrics.enabled=true \
  --namespace monitoring \
  --create-namespace
```

### 3. 验证安装

```bash
# 检查 Pod 状态
kubectl get pods -l app.kubernetes.io/name=monitor-operator

# 检查 CRD 是否创建
kubectl get crd monitorstacks.monitoring.cillian.website

# 查看 Operator 日志
kubectl logs -f deployment/monitor-operator-controller-manager
```

## 安装选项

### 开发环境安装

```bash
helm install monitor-operator ciliverse/monitor-operator \
  --set deployment.replicas=1 \
  --set deployment.resources.requests.cpu=50m \
  --set deployment.resources.requests.memory=64Mi \
  --set podDisruptionBudget.enabled=false \
  --namespace monitoring \
  --create-namespace
```

### 生产环境安装

```bash
helm install monitor-operator ciliverse/monitor-operator \
  --set deployment.replicas=2 \
  --set deployment.resources.requests.cpu=200m \
  --set deployment.resources.requests.memory=256Mi \
  --set podDisruptionBudget.enabled=true \
  --set leaderElection.enabled=true \
  --namespace monitoring \
  --create-namespace
```

### 使用自定义 Values 文件

创建 `values.yaml` 文件：

```yaml
# values.yaml
deployment:
  replicas: 2
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 256Mi

metrics:
  enabled: true

podDisruptionBudget:
  enabled: true
  minAvailable: 1

nodeSelector:
  kubernetes.io/os: linux

tolerations:
  - key: "monitoring"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

然后安装：

```bash
helm install monitor-operator ciliverse/monitor-operator -f values.yaml
```

## 配置参数

### 基础配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `image.repository` | 镜像仓库 | `cilliantech/monitor-operator` |
| `image.tag` | 镜像标签 | `v1.0.0` |
| `image.pullPolicy` | 镜像拉取策略 | `IfNotPresent` |
| `imagePullSecrets` | 镜像拉取密钥 | `[]` |

### 命名空间配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `namespace.create` | 是否创建命名空间 | `true` |
| `namespace.name` | 命名空间名称 | `""` (使用 Release.Namespace) |

### 服务账户配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `serviceAccount.create` | 是否创建服务账户 | `true` |
| `serviceAccount.name` | 服务账户名称 | `monitor-operator-controller-manager` |
| `serviceAccount.annotations` | 服务账户注解 | `{}` |

### 部署配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `deployment.replicas` | 副本数量 | `1` |
| `deployment.resources.limits.cpu` | CPU 限制 | `500m` |
| `deployment.resources.limits.memory` | 内存限制 | `128Mi` |
| `deployment.resources.requests.cpu` | CPU 请求 | `10m` |
| `deployment.resources.requests.memory` | 内存请求 | `64Mi` |

### 监控配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `metrics.enabled` | 是否启用 metrics | `true` |
| `metrics.service.port` | Metrics 服务端口 | `8443` |
| `metrics.service.targetPort` | Metrics 目标端口 | `8443` |

### RBAC 配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `rbac.create` | 是否创建 RBAC 资源 | `true` |

### 高可用配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `podDisruptionBudget.enabled` | 是否启用 PDB | `false` |
| `podDisruptionBudget.minAvailable` | 最小可用副本数 | `1` |
| `leaderElection.enabled` | 是否启用 Leader Election | `true` |

### 其他配置

| 参数 | 描述 | 默认值 |
|------|------|--------|
| `nodeSelector` | 节点选择器 | `{}` |
| `tolerations` | 容忍度 | `[]` |
| `podAnnotations` | Pod 注解 | `{}` |
| `commonLabels` | 通用标签 | `{}` |
| `commonAnnotations` | 通用注解 | `{}` |

## 使用示例

### 创建 MonitorStack 实例

安装 Monitor Operator 后，你可以创建 MonitorStack 自定义资源来部署监控栈：

```bash
# 创建一个简单的监控栈
kubectl apply -f - <<EOF
apiVersion: monitoring.cillian.website/v1
kind: MonitorStack
metadata:
  name: demo-monitor
  namespace: default
spec:
  prometheus:
    enabled: true
    retention: "7d"
    service:
      type: ClusterIP
      port: 9090
  grafana:
    enabled: true
    adminPassword: "admin123"
    service:
      type: ClusterIP
      port: 3000
EOF
```

### 访问监控服务

```bash
# 访问 Prometheus
kubectl port-forward svc/demo-monitor-prometheus 9090:9090

# 访问 Grafana
kubectl port-forward svc/demo-monitor-grafana 3000:3000
```

然后在浏览器中访问：
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000 (admin/admin123)

### 高级配置示例

```yaml
# 生产环境配置示例
apiVersion: monitoring.cillian.website/v1
kind: MonitorStack
metadata:
  name: production-monitor
  namespace: monitoring
spec:
  prometheus:
    enabled: true
    retention: "30d"
    service:
      type: LoadBalancer
      port: 9090
    resources:
      requests:
        cpu: 200m
        memory: 512Mi
      limits:
        cpu: 1000m
        memory: 2Gi
    storage:
      size: 50Gi
      storageClass: fast-ssd
  grafana:
    enabled: true
    adminPassword: "secure-password-123"
    service:
      type: LoadBalancer
      port: 3000
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi
```

## 升级

```bash
# 升级到新版本
helm upgrade monitor-operator ciliverse/monitor-operator

# 升级并修改配置
helm upgrade monitor-operator ciliverse/monitor-operator -f values.yaml

# 查看升级历史
helm history monitor-operator
```

## 卸载

### 标准卸载

```bash
# 卸载 Chart（保留 CRD 和数据）
helm uninstall monitor-operator

# 删除 MonitorStack 实例（可选）
kubectl delete monitorstacks --all --all-namespaces
```

### 完全卸载

```bash
# 1. 删除所有 MonitorStack 实例
kubectl delete monitorstacks --all --all-namespaces

# 2. 卸载 Helm Chart
helm uninstall monitor-operator

# 3. 删除 CRD（注意：这会永久删除所有相关数据）
kubectl delete crd monitorstacks.monitoring.cillian.website

# 4. 清理命名空间（如果需要）
kubectl delete namespace monitoring
```

## 故障排除

### 检查 Operator 状态

```bash
# 检查 Pod 状态
kubectl get pods -l app.kubernetes.io/name=monitor-operator

# 查看日志
kubectl logs -f deployment/monitor-operator-controller-manager

# 检查 CRD
kubectl get crd monitorstacks.monitoring.cillian.website
```

### 检查 MonitorStack 状态

```bash
# 列出所有 MonitorStack
kubectl get monitorstacks -A

# 查看详细状态
kubectl describe monitorstack example-monitor

# 检查 Operator 创建的资源
kubectl get all -l app.kubernetes.io/managed-by=monitor-operator
```

### 常见问题

1. **CRD 安装失败**
   - 确保有集群管理员权限
   - 检查 Kubernetes 版本兼容性

2. **Operator Pod 启动失败**
   - 检查镜像是否可访问
   - 验证 RBAC 权限配置

3. **MonitorStack 创建失败**
   - 检查 Operator 日志
   - 验证资源配置是否正确

## 开发

### 本地测试

```bash
# 验证 Chart 语法
helm lint ./monitor-operator

# 渲染模板（不安装）
helm template test-release ./monitor-operator

# 运行测试
helm test monitor-operator
```

### 打包

```bash
# 打包 Chart
helm package ./monitor-operator

# 生成索引文件
helm repo index .
```

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

本项目采用 MIT 许可证。

## 联系方式

- 项目主页: https://github.com/ciliverse/monitor-operator
- 维护者: cilliantech@gmail.com
- 网站: https://cillian.website

> 来源：[Kubernetes 官方文档](https://kubernetes.io/docs/)、[Kubernetes RBAC Good Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)、[Considerations for large clusters](https://kubernetes.io/docs/setup/best-practices/cluster-large/)

## 核心原则

1. **最小权限原则（Least Privilege）** — 仅授予用户和工作负载执行其角色所需的最低权限。优先使用 RoleBindings 而非 ClusterRoleBindings，避免使用通配符权限。
2. **资源配额与限制（Resource Management）** — 为每个容器定义 Requests 和 Limits，为每个命名空间配置 ResourceQuotas 和 LimitRanges。
3. **安全加固（Defense in Depth）** — 实施 Pod Security Standards、Network Policies、Secrets 加密、定期证书轮换。禁用 `system:masters` 组的滥用。
4. **高可用与容错（High Availability）** — 控制平面组件跨故障域部署，etcd 使用独立的奇数节点集群，使用 PodDisruptionBudget 保护关键工作负载。
5. **可观测性（Observability）** — 实施结构化日志、Prometheus 指标、分布式追踪。关键组件使用 `system-cluster-critical` PriorityClass。

## 项目目录/架构规范

```
k8s-manifests/
├── base/                      # 基础资源定义
│   ├── namespaces.yaml
│   ├── network-policies/
│   ├── rbac/
│   └── pod-security/
├── apps/
│   ├── app-a/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── hpa.yaml
│   │   └── pdb.yaml
│   └── app-b/
├── infra/
│   ├── ingress/
│   ├── cert-manager/
│   └── monitoring/
└── overlays/                  # Kustomize 环境覆盖
    ├── dev/
    ├── staging/
    └── prod/
```

## 标准代码示例

### 安全的 Deployment 配置

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      serviceAccountName: secure-app-sa
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        image: registry.io/app:v1.2.3
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: secure-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: secure-app
```

### RBAC 最小权限配置

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-operator
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-operator-binding
  namespace: production
subjects:
- kind: User
  name: operator@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: app-operator
  apiGroup: rbac.authorization.k8s.io
```

## 常见反模式

| 反模式 | 描述 | 后果 |
|--------|------|------|
| **使用 `cluster-admin` 作为日常账户** | 任何 `system:masters` 组成员绕过所有 RBAC 检查 | 应使用低权限账户配合 Impersonation |
| **不设置资源限制** | 导致 noisy neighbor 问题，某个 Pod 耗尽节点资源引发级联故障 | 必须设置 Requests/Limits |
| **在容器中以 root 运行** | 容器逃逸风险大幅增加 | 应始终设置 `runAsNonRoot: true` 和 `readOnlyRootFilesystem: true` |

---

**参考来源**：
- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes RBAC Good Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)
- [Kubernetes Large Clusters](https://kubernetes.io/docs/setup/best-practices/cluster-large/)

---

## 相关笔记

- [[etcd]] — Kubernetes 的数据存储后端
- [[运维与 SRE]] — Kubernetes 运维的最佳实践

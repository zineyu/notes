> 来源：[etcd 官方文档](https://etcd.io/docs/v3.5/)、[etcd Clustering Guide](https://etcd.io/docs/v3.5/op-guide/clustering/)、[etcd Hardware Recommendations](https://etcd.io/docs/v3.5/op-guide/hardware/)

## 核心原则

1. **奇数节点集群（Odd Member Count）** — 生产环境强烈建议运行 5 节点静态 etcd 集群（或至少 3 节点），确保在故障时仍能达成 Raft 共识。
2. **专用硬件资源（Dedicated Resources）** — etcd 对磁盘延迟极度敏感。必须使用 SSD/NVMe，独立磁盘用于数据存储，避免与应用程序共享 I/O。
3. **启用 TLS 加密（Transport Security）** — 所有 peer 通信和 client 通信必须启用 TLS，使用独立的 CA 证书分别签名 peer 和 client 证书。
4. **定期备份与快照（Regular Backups）** — 使用 `etcdctl snapshot save` 定期备份，定期执行 `etcdctl defrag` 整理数据文件，监控数据库大小。
5. **监控与告警（Monitoring）** — 监控 `etcd_server_has_leader`、`etcd_server_leader_changes_seen_total`、`etcd_disk_wal_fsync_duration_seconds` 等关键指标。

## 项目目录/架构规范

```
etcd-cluster/
├── certs/                     # TLS 证书目录
│   ├── ca/
│   ├── peer/
│   └── client/
├── config/
│   ├── etcd0.conf.yml
│   ├── etcd1.conf.yml
│   └── etcd2.conf.yml
├── scripts/
│   ├── backup.sh              # 定时快照脚本
│   ├── restore.sh             # 恢复脚本
│   └── health-check.sh
├── systemd/
│   └── etcd.service
└── monitoring/
    └── prometheus-alerts.yml
```

## 标准代码示例

### 3节点 etcd 集群静态配置（带 TLS）

```bash
# Node 0 (infra0)
etcd --name infra0 \
  --initial-advertise-peer-urls https://10.0.1.10:2380 \
  --listen-peer-urls https://10.0.1.10:2380 \
  --listen-client-urls https://10.0.1.10:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.0.1.10:2379 \
  --initial-cluster-token etcd-cluster-prod \
  --initial-cluster infra0=https://10.0.1.10:2380,infra1=https://10.0.1.11:2380,infra2=https://10.0.1.12:2380 \
  --initial-cluster-state new \
  --client-cert-auth \
  --trusted-ca-file=/etc/etcd/certs/ca-client.crt \
  --cert-file=/etc/etcd/certs/infra0-client.crt \
  --key-file=/etc/etcd/certs/infra0-client.key \
  --peer-client-cert-auth \
  --peer-trusted-ca-file=/etc/etcd/certs/ca-peer.crt \
  --peer-cert-file=/etc/etcd/certs/infra0-peer.crt \
  --peer-key-file=/etc/etcd/certs/infra0-peer.key \
  --snapshot-count 10000 \
  --heartbeat-interval 100 \
  --election-timeout 1000 \
  --data-dir=/var/lib/etcd
```

### etcd 备份与监控脚本

```bash
#!/bin/bash
# etcd-backup.sh
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/etcd/certs/ca-client.crt \
  --cert=/etc/etcd/certs/backup-client.crt \
  --key=/etc/etcd/certs/backup-client.key \
  --endpoints=https://10.0.1.10:2379 \
  snapshot save /backup/etcd-$(date +%Y%m%d-%H%M%S).db

# 保留最近 7 天备份
find /backup -name "etcd-*.db" -mtime +7 -delete
```

**Prometheus 告警规则：**

```yaml
groups:
- name: etcd
  rules:
  - alert: etcdNoLeader
    expr: etcd_server_has_leader{job=~".*etcd.*"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "etcd member has no leader"
      
  - alert: etcdHighFsyncDuration
    expr: histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])) > 0.5
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "etcd fsync duration is too high"
```

## 常见反模式

| 反模式 | 描述 | 后果 |
|--------|------|------|
| **使用偶数节点集群** | 4 节点集群在 2 节点故障时无法达成多数派，可用性反而低于 3 节点 | 始终使用奇数节点 |
| **与 Kubernetes 工作负载共享 etcd** | Kubernetes 控制平面的 etcd 应完全独立，不应与应用程序共享 | 大型集群应将 Event 对象存储到单独的 etcd 实例 |
| **忽略磁盘性能** | etcd 是写密集型应用，HDD 会导致高延迟和 leader 频繁切换 | 必须使用 SSD 并监控 `etcd_disk_wal_fsync_duration_seconds` |

---

**参考来源**：
- [etcd Official Documentation](https://etcd.io/docs/v3.5/)
- [etcd Clustering Guide](https://etcd.io/docs/v3.5/op-guide/clustering/)
- [etcd Hardware Recommendations](https://etcd.io/docs/v3.5/op-guide/hardware/)

---

## 相关笔记

- [[Kubernetes]] — 使用 etcd 作为数据存储的容器编排平台
- [[运维与 SRE]] — etcd 集群的运维与监控

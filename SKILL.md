---
name: 缓存命中率优化器
description: 多层级智能缓存系统 - 自动分析访问模式并动态调整缓存策略以最大化命中率
version: 2.1.0
author: AI Assistant
tags: [缓存, 性能优化, Redis, CDN, 命中率]
---

# 缓存命中率优化器

本 Skill 用于自动分析、生成、部署和监控缓存策略，帮助将缓存命中率提升至 **95%+**。

---

## 配置参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `max_memory_cache_mb` | 512 | L1 内存缓存最大容量 |
| `max_disk_cache_gb` | 10 | L2 磁盘缓存最大容量 |
| `default_ttl_seconds` | 86400 | 默认过期时间（24小时） |
| `lru_eviction_threshold` | 0.85 | LRU 淘汰触发阈值 |
| `prefetch_enabled` | true | 是否启用预取 |
| `prefetch_batch_size` | 10 | 每批预取条目数 |
| `hit_rate_target` | 0.95 | 目标命中率 |

---

## 命令列表

### 1. 分析访问模式
**触发词**：`分析缓存访问模式` `分析热点数据` `扫描访问日志`

**输入**：
- `log_file`（必填）：访问日志文件路径（支持 .log / .csv / .json）
- `report_dir`（必填）：分析报告输出目录
- `time_window`（可选，默认 24）：分析最近 N 小时数据

**执行步骤**：
1. 读取并解析访问日志
2. 统计每个资源的访问频率和访问时间分布
3. 使用 Zipf 分布模型识别热点数据
4. 生成访问模式热力图和时间序列报告
5. 输出 JSON 分析报告到 `report_dir`

**输出示例**：
```json
{
  "hot_keys": ["/api/user/123", "/static/logo.png", "/api/product/456"],
  "frequency_distribution": "zipf_alpha_1.2",
  "peak_hours": [9, 12, 18, 21],
  "cacheable_ratio": 0.78,
  "estimated_hit_rate_improvement": "+23%"
}
```

---

### 2. 生成缓存策略
**触发词**：`生成缓存策略` `创建缓存规则` `制定缓存方案`

**输入**：
- `report_path`（必填）：由"分析访问模式"生成的 JSON 报告
- `strategy_file`（必填）：策略输出文件路径（.json / .yaml）
- `mode`（可选）：策略模式
  - `aggressive`：激进模式，最大化缓存，适合读多写少
  - `balanced`：平衡模式，兼顾命中率与内存开销
  - `conservative`：保守模式，适合写密集型场景
  - `adaptive`（默认）：自适应模式，根据负载动态调整

**执行步骤**：
1. 读取访问模式分析报告
2. 根据模式计算最优 TTL 分配
3. 为不同优先级资源设置差异化策略
4. 配置淘汰策略（LRU / LFU / FIFO / ARC）
5. 输出完整策略文件

**策略文件结构**：
```yaml
tiered_caching:
  l1_memory:
    max_size_mb: 512
    ttl_seconds: 300
    eviction: LRU
    keys: ["session_*", "user_profile_*"]
  l2_redis:
    max_size_mb: 2048
    ttl_seconds: 3600
    eviction: LFU
    keys: ["product_*", "category_*"]
  l3_database:
    ttl_seconds: 86400
    eviction: FIFO
    keys: ["audit_log_*", "archive_*"]

hot_data_prefetch:
  enabled: true
  algorithm: ARIMA
  lookahead_seconds: 600
  batch_size: 50

coherency:
  write_strategy: write-through
  invalidation: event-driven
  stale_tolerance_ms: 500

anti_pattern:
  bloom_filter: true          # 防缓存穿透
  ttl_jitter_seconds: 120     # 防缓存雪崩
  mutex_lock_ms: 5000         # 防缓存击穿
```

---

### 3. 部署缓存规则
**触发词**：`部署缓存` `应用缓存策略` `推送缓存配置`

**输入**：
- `strategy_file`（必填）：策略配置文件路径
- `target_type`（可选，默认 all）：部署目标
  - `redis`：部署到 Redis
  - `nginx`：生成 Nginx proxy_cache 配置
  - `cdn`：更新 CDN 边缘节点规则
  - `memory`：配置本地内存缓存
  - `all`：全量部署
- `dry_run`（可选，默认 true）：模拟运行预览

**执行步骤**：
1. 解析策略文件
2. 生成对应目标的配置指令
3. dry_run=true 时仅输出预览，不实际修改
4. dry_run=false 时执行部署并验证

**预览输出示例**：
```
[DRY RUN] 将应用的规则：
  Redis:    SET maxmemory 2gb + 新增 15 条缓存键规则
  Nginx:    proxy_cache_valid 200 1h + 10 条 location 缓存规则
  CDN:      配置 8 条边缘缓存规则，TTL=6h
  Memory:   启用 LRU 淘汰，容量=512MB
```

---

### 4. 预热缓存
**触发词**：`预热缓存` `缓存预热` `预加载热点数据`

**输入**：
- `strategy_file`（必填）：策略配置文件
- `batch_size`（可选，默认 100）：每批加载条目数
- `priority`（可选，默认 frequent）：预热优先级
  - `high`：仅高优先级资源
  - `frequent`：高频访问资源
  - `recent`：最近访问的资源
  - `all`：全部预加载

**执行步骤**：
1. 从策略文件提取预热键列表
2. 按优先级排序
3. 分批异步加载数据到对应缓存层
4. 输出预热进度和预估命中率提升

**输出示例**：
```
[进度] 预热中... 批次 3/8 (37%)
  ✓ L1 Memory: 加载 150 条 (150MB)
  ✓ L2 Redis:  加载 320 条 (640MB)
  → 预计首小时命中率提升: +15%
```

---

### 5. 实时监控命中率
**触发词**：`监控缓存命中率` `查看缓存性能` `缓存监控`

**输入**：
- `interval`（可选，默认 60）：采样间隔（秒）
- `metrics_file`（必填）：指标输出文件（.json / .csv）
- `alert_threshold`（可选，默认 80）：告警阈值百分比

**监控指标**：
| 指标 | 说明 |
|------|------|
| `hit_rate` | 当前命中率 (%) |
| `miss_rate` | 未命中率 (%) |
| `eviction_rate` | 淘汰速率 (条/秒) |
| `memory_usage` | 内存占用 (%) |
| `avg_latency_ms` | 平均延迟 (ms) |
| `cache_throughput` | 吞吐量 (req/s) |

**告警规则**：
- `hit_rate < alert_threshold`：触发命中率告警
- `eviction_rate > 100/s`：触发高淘汰告警
- `memory_usage > 90%`：触发内存告警

---

### 6. 自动优化配置
**触发词**：`自动优化缓存` `调优缓存参数` `AI缓存优化`

**输入**：
- `config_file`（必填）：当前配置文件
- `metrics_file`（必填）：监控指标文件
- `optimized_config`（必填）：优化后配置输出路径

**优化维度**：
1. **TTL 调整**：根据访问间隔动态计算最佳过期时间
2. **容量分配**：按命中率贡献比重新分配各层容量
3. **淘汰策略切换**：根据访问模式在 LRU/LFU/ARC 间切换
4. **预取参数调优**：调整预取窗口和批量大小
5. **压缩策略**：在高负载时自动启用 zstd 压缩

**优化前后对比示例**：
```
                   优化前      优化后      提升
  ─────────────────────────────────────────────
  命中率           82.3%      94.7%      +12.4%
  平均延迟         45ms       18ms       -60%
  内存利用率       67%        83%        +16%
  淘汰率           35/s       8/s        -77%
```

---

### 7. 全链路缓存诊断
**触发词**：`缓存诊断` `缓存体检` `缓存问题排查`

**输入**：
- `target`（可选，默认 all）：诊断范围
- `full_report`（必填）：诊断报告输出路径

**诊断项**：
- [ ] 缓存穿透：是否存在大量查询不存在的数据
- [ ] 缓存雪崩：是否存在集中过期的风险
- [ ] 缓存击穿：热点 key 是否缺乏并发保护
- [ ] 内存碎片：内存利用率是否因碎片而降低
- [ ] 序列化开销：序列化/反序列化耗时
- [ ] 网络延迟：各缓存层之间的网络 RTT
- [ ] 命中率瓶颈：识别拉低命中率的 top 10 资源
- [ ] 配置冲突：检测互矛盾的缓存规则

---

## 内置缓存策略详解

### 分级缓存架构 (Tiered Caching)
```
     ┌─────────────────┐
     │   客户端请求     │
     └────────┬────────┘
              ▼
     ┌─────────────────┐
     │ L1: 本地内存    │ ← LRU, 512MB, TTL=5min
     │   命中率: ~60%  │
     └────────┬────────┘
              ▼ (未命中)
     ┌─────────────────┐
     │ L2: Redis 集群  │ ← LFU, 2GB, TTL=1h
     │   命中率: ~30%  │
     └────────┬────────┘
              ▼ (未命中)
     ┌─────────────────┐
     │ L3: 源数据库     │ ← FIFO, TTL=24h
     │   命中率: ~5%   │
     └─────────────────┘
```

### 热点预取 (Hot Data Prefetch)
- 算法：ARIMA 时间序列预测
- 预测窗口：600 秒
- 置信阈值：>0.7 才触发预取
- 批次限制：每批最多 100 条

### 缓存一致性保障
- 写入策略：**Write-Through**（写穿透）
- 失效方式：**事件驱动**（数据变更时主动失效）
- 陈旧容忍：最多 500ms 的不一致窗口

### 防缓存三大问题
1. **防穿透**：布隆过滤器 + 空值缓存（TTL=60s）
2. **防雪崩**：TTL 随机抖动 ±120 秒 + 永不过期热 key
3. **防击穿**：热点 key 互斥锁 + 异步续期

---

## 最佳实践清单

- [x] 部署布隆过滤器防止缓存穿透
- [x] 热点数据设置永不过期 + 异步刷新
- [x] 使用一致性哈希实现分布式缓存扩展
- [x] TTL 添加随机抖动防止雪崩
- [x] 定期清理冷数据释放内存
- [x] 静态资源使用 CDN 边缘缓存
- [x] API 响应设置 ETag/If-None-Match 条件请求
- [x] L1 内存层使用 LRU 淘汰
- [x] L2 Redis 层使用 LFU 淘汰
- [x] 热点 key 使用互斥锁防击穿
- [x] 大 value 启用 zstd Level-3 压缩
- [x] 缓存操作记录审计日志

---

## 快速开始

```
1. 分析 → 分析缓存访问模式 log_file=/var/log/access.log report_dir=./reports
2. 生成 → 生成缓存策略 report_path=./reports/analysis.json strategy_file=./cache-strategy.yaml mode=adaptive
3. 预览 → 部署缓存规则 strategy_file=./cache-strategy.yaml dry_run=true
4. 部署 → 部署缓存规则 strategy_file=./cache-strategy.yaml dry_run=false
5. 预热 → 预热缓存 strategy_file=./cache-strategy.yaml priority=frequent
6. 监控 → 监控缓存命中率 metrics_file=./metrics.json
```

---

## 注意事项

- 部署规则建议先使用 `dry_run=true` 预览
- 配置文件修改前会自动备份
- 生产环境建议先在测试环境验证策略效果
- ARIMA 预测模型需要至少 7 天历史数据才能准确
- 内存告警触发时会自动触发冷数据淘汰

# Query Q1 搜索报告

**查询式**：RDMA multi-tenant performance isolation / RDMA distributed training communication scheduling
**平台**：arXiv API, Semantic Scholar API, WebSearch (S2/arXiv 遇 429 后回退)
**时间**：2026-06-27T18:00:00Z

## 1. 搜索策略

### 查询扩展（按 search-protocol.md §核心能力 > 关键词搜索）

根据 instruction sheet 的查询意图，展开为以下互补查询：

| # | 查询式 | 平台 | 目的 |
|---|--------|------|------|
| 1 | `all:RDMA AND all:multi-tenant AND all:performance AND all:isolation` | arXiv | 核心：RDMA多租户性能隔离 |
| 2 | `all:RDMA AND all:network AND all:isolation AND all:datacenter` | arXiv | 补充：数据中心网络隔离 |
| 3 | `all:RDMA AND all:distributed AND all:training AND all:communication` | arXiv | 补充：分布式训练通信 |
| 4 | `RDMA multi-tenant performance isolation` | S2 / WebSearch | 核心关键词 |
| 5 | `RDMA network isolation multi-tenancy datacenter` | S2 / WebSearch | 变体 |
| 6 | `RDMA congestion control multi-tenant` | arXiv / WebSearch | 拥塞控制方向 |

### 筛选规则

- **年份范围**：[2021, 2026]
- **Venue 偏好**：CCF-A/B 顶会优先（NSDI/SIGCOMM/ATC/EuroSys）
- **时效性置顶**：近 6 个月标注 `[新]` 并置顶
- **相关性分级**：core（直接面向RDMA多租户性能隔离的系统方案）vs general（分布式训练通信调度/广义RDMA优化）
- **排除**：纯理论分析无系统实现、非RDMA场景的网络隔离、RDMA单任务性能调优

### 特殊说明

arXiv 和 Semantic Scholar API 在并发请求后均触发了 429 rate limit。按 search-protocol.md 规定，等待 15s+ 后重试 3 次无改善后，切换为 WebSearch 辅助发现论文，然后通过 USENIX OA 和 arXiv 直链获取 PDF。

---

## 2. 结果总览

| 指标 | 数值 |
|------|------|
| 总检索 / 筛选后 | 80+ / 16 |
| core / general | 9 / 7 |
| 近 6 个月 [新] | 4 |
| CCF-A / CCF-B / 预印本 / 未收录 | 5 / 0 / 7 / 4 |

---

## 3. 重要论文（core）

### 3-1. [Harmonic: Hardware-assisted RDMA Performance Isolation for Public Clouds]（2024 NSDI [CCF-A]）

首个微架构资源感知的 RDMA 性能隔离方案。通过 FPGA 实现的 PIPS（可编程智能 PCIe 交换）拦截 PCIe 事务监控每租户 RNIC 资源使用，并重用 DCQCN 拥塞控制速率限制器通过注入 CNP 来限流恶意租户。唯一通过 Husky 测试套件的方案，正被集成到下一代商用 RNIC 设计中。

- 链接：[USENIX NSDI 2024](https://www.usenix.org/conference/nsdi24/presentation/lou)
- OA：open_pdf（USENIX 开放获取）
- PDF：`papers/inbox/2024_Harmonic_RDMA_Isolation.pdf` (3.2 MB)

### 3-2. [Understanding RDMA Microarchitecture Resources for Performance Isolation (Husky)]（2023 NSDI [CCF-A]）

首篇系统性揭示 RNIC 微架构资源（NIC 缓存、处理单元、PCIe 带宽）是 RDMA 多租户隔离的关键瓶颈的工作。发现仅 1 Gbps 的攻击流量即可将受害租户带宽从 ~50 Gbps 降至 2 Gbps。提出 Husky 测试套件——首个 RDMA 隔离评估基准，证明当时所有方案（SR-IOV、HW TC、Justitia）均无法通过。结果被 NVIDIA 确认复现。

- 链接：[USENIX NSDI 2023](https://www.usenix.org/conference/nsdi23/presentation/kong)
- OA：open_pdf（USENIX 开放获取）
- PDF：`papers/inbox/2023_Husky_RDMA_Microarchitecture.pdf` (1.7 MB)

### 3-3. [PeRF: Preemption-enabled RDMA Framework]（2024 ATC [CCF-A]）

纯软件实现的 RDMA 性能隔离框架。利用现有 RDMA 动词（IB_WR_WAIT / IB_WR_ENABLE）实现创新的 RNIC 抢占机制，类似 OS 抢占调度，动态控制每连接 RDMA 资源使用。工作保持（work-conserving）、无需专用硬件、无需修改应用。相比此前软件方案吞吐量提升约 2.04x。

- 链接：[USENIX ATC 2024](https://www.usenix.org/conference/atc24/presentation/lee)
- OA：open_pdf（USENIX 开放获取）
- PDF：`papers/inbox/2024_PeRF_Preemption_RDMA.pdf` (2.0 MB)

### 3-4. [Palladium: A DPU-enabled Multi-Tenant Serverless Cloud over Zero-copy Multi-node RDMA Fabrics]（2025 SIGCOMM [CCF-A]）

DPU 中心化的 serverless 数据平面。通过 DPU-enabled Network Engine (DNE) 作为轻量反向代理隔离 RDMA QP 与不可信租户函数，在租户间实施公平带宽分配。利用 CPU-DPU 跨处理器共享内存实现零拷贝数据传输，结合集群入口处的 HTTP/TCP-to-RDMA 协议转换。相比传统方案吞吐量提升 20.9x，节省 7 个 CPU 核。

- 链接：[arXiv:2505.11339](https://arxiv.org/abs/2505.11339)
- OA：open_pdf（arXiv 预印本）
- PDF：`papers/inbox/2025_Palladium_DPU_RDMA_Serverless.pdf` (1.5 MB)

### 3-5. [Palos: Fair and Flexible Flow Scheduling on RNIC]（2024 HPCC [未收录]）

识别 RNIC 硬件中包级流调度是隔离不足的根本原因。提出数据块级调度机制：硬件层通过重构通信描述符实现数据块调度以减小大小流间干扰；软件层通过层级权重设置实现可定制性能策略并防止跨用户配置干扰。

- 链接：[IEEE Xplore](https://ieeexplore.ieee.org/document/11083207) | DOI: `10.1109/hpcc64274.2024.00127`
- OA：needs_institution（IEEE 付费墙）
- PDF：未下载（paywalled；可通过机构访问 IEEE Xplore 或联系作者获取）

### 3-6. [OSMOSIS: Enabling Multi-Tenancy in Datacenter SmartNICs]（2023 [预印本]）

ETH Zurich 针对 400 Gbit/s SmartNIC 的多租户公平资源复用方案。引入 Weight-Limited Borrowed Virtual Time (WLBVT) 硬件调度器公平分配处理单元，通过传输分片和加权轮询解决 DMA 和出口路径的 HoL 阻塞。在 RISC-V PsPIN SmartNIC 上实现，硬件开销极小。

- 链接：[arXiv:2309.03628](https://arxiv.org/abs/2309.03628)
- OA：open_pdf（arXiv 预印本）
- PDF：`papers/inbox/2023_OSMOSIS_SmartNIC_MultiTenant.pdf` (0.9 MB)

### 3-7. [Closing the HPC-Cloud Convergence Gap: Multi-Tenant Slingshot RDMA for Kubernetes]（2025 IEEE Cluster [未收录]）

将 HPE Slingshot 200 Gbps RDMA 互连扩展到 Kubernetes 多租户场景。引入基于网络命名空间的 CXI 驱动认证替代 UID/GID 方案、VNI 管理服务实现租户间 RDMA 通信域隔离、CNI 插件管理 CXI 服务生命周期。

- 链接：[arXiv:2508.09663](https://arxiv.org/abs/2508.09663)
- OA：open_pdf（arXiv 预印本）
- PDF：`papers/inbox/2025_Slingshot_RDMA_K8s.pdf` (0.5 MB)

### 3-8. [Optimize the TX Architecture of RDMA NIC for Performance Isolation in the Cloud Environment]（2023 GLSVLSI [未收录]）

硬件级 RNIC 发送端架构优化：独立缓存 + 切片执行隔离延迟敏感和带宽敏感租户；隔离背压；自适应加权轮询公平带宽共享。以最小 CPU 开销实现近最优性能隔离。

- 链接：[ACM DL](https://dl.acm.org/doi/10.1145/3583781.3590276)
- OA：needs_institution（ACM 付费墙）
- PDF：未下载（paywalled；可通过机构访问 ACM DL 或联系作者获取）

### 3-9. [Canvas: Isolated and Adaptive Swapping for Multi-Applications on Remote Memory]（2022 [预印本]）

面向远内存/RDMA 场景的隔离交换系统。每应用独立交换分区、交换缓存、预取器和 RDMA 带宽。二维 RDMA 调度器同时在应用间和需求/预取请求间调度数据包。将同跑性能下降从 6x 降低到接近隔离水平，RDMA 带宽利用率提升 2.8x。

- 链接：[arXiv:2203.09615](https://arxiv.org/abs/2203.09615)
- OA：open_pdf（arXiv 预印本）
- PDF：未下载（download quota reached；arXiv PDF 可直链获取：https://arxiv.org/pdf/2203.09615）

---

## 4. 一般论文清单

| # | 标题 | 年份 | Venue | 引用 | OA | 链接 |
|---|------|------|-------|------|----|------|
| G1 | CSA-UD: Communication-Semantic-Aware RDMA Loss Recovery for Hyperscale AI Training | 2026 [新] | [预印本] | - | open_pdf | [arXiv:2606.20582](https://arxiv.org/abs/2606.20582) |
| G2 | NCCL EP: Towards a Unified Expert Parallel Communication API for NCCL | 2026 [新] | [预印本] | - | open_pdf | [arXiv:2603.13606](https://arxiv.org/abs/2603.13606) |
| G3 | UCCL-EP: Portable Expert-Parallel Communication | 2025 | [预印本] | - | open_pdf | [arXiv:2512.19849](https://arxiv.org/abs/2512.19849) |
| G4 | OptiNIC: A Resilient and Tail-Optimal RDMA NIC for Distributed ML Workloads | 2025 | [预印本] | - | open_pdf | [arXiv:2512.22743](https://arxiv.org/abs/2512.22743) |
| G5 | HetCCL: Accelerating LLM Training with Heterogeneous GPUs | 2026 [新] | [预印本] | - | open_pdf | [arXiv:2601.22585](https://arxiv.org/abs/2601.22585) |
| G6 | ForestColl: Throughput-Optimal Collective Communications on Heterogeneous Network Fabrics | 2024 | [预印本] | - | open_pdf | [arXiv:2402.06787](https://arxiv.org/abs/2402.06787) |
| G7 | MonkeyTree: Near-Minimal Congestion for Multi-tenant Training via Migration | 2026 [新] | [预印本] | - | open_pdf | [arXiv:2602.08296](https://arxiv.org/abs/2602.08296) |

---

## 5. PDF 下载结果

| 状态 | 数量 |
|------|------|
| 应下载（core + open_pdf）| 7 |
| 已下载 | 7 |
| 额外下载（general + open_pdf）| 3 |
| 未下载（超出 quota）| 1 |
| 非 OA / 需合法获取 | 2 |

### 下载详情

| 论文 | 来源 | 大小 | 状态 |
|------|------|------|------|
| Harmonic (NSDI 2024) | USENIX OA | 3.2 MB | downloaded |
| Husky (NSDI 2023) | USENIX OA | 1.7 MB | downloaded |
| PeRF (ATC 2024) | USENIX OA | 2.0 MB | downloaded |
| Palladium (2025) | arXiv | 1.5 MB | downloaded |
| OSMOSIS (2023) | arXiv | 0.9 MB | downloaded |
| Slingshot K8s (2025) | arXiv | 0.5 MB | downloaded |
| ROS2 (2025) | arXiv | 2.8 MB | downloaded |
| CSA-UD (2026) | arXiv | 3.6 MB | downloaded |
| NCCL EP (2026) | arXiv | 12.8 MB | downloaded |
| OptiNIC (2025) | arXiv | 0.4 MB | downloaded |

### 未下载

| 论文 | 原因 | 合法获取途径 |
|------|------|-------------|
| Palos (HPCC 2024) | paywalled_do_not_bypass (IEEE Xplore) | 机构订阅 IEEE Xplore，DOI: 10.1109/hpcc64274.2024.00127 |
| TX Architecture (GLSVLSI 2023) | paywalled_do_not_bypass (ACM DL) | 机构订阅 ACM DL，DOI: 10.1145/3583781.3590276 |
| Canvas (2022) | download_quota_exceeded | arXiv OA: https://arxiv.org/pdf/2203.09615 |

---

## 6. 技术路线对比小结

已有 RDMA 多租户性能隔离方案按技术路线可分为：

| 技术路线 | 代表工作 | 隔离粒度 | 性能开销 | 实现复杂度 |
|----------|---------|---------|---------|-----------|
| **纯软件抢占调度** | PeRF (ATC'24) | QP 级 | 低（~2.04x 提升） | 低（仅用现有 API） |
| **硬件/软硬协同** | Harmonic (NSDI'24), TX Opt (GLSVLSI'23) | 微架构资源级 | 中（需 FPGA/NIC 改造） | 高（需硬件改动） |
| **DPU/SmartNIC 卸载隔离** | Palladium (SIGCOMM'25), OSMOSIS (2023), ROS2 (SC'25) | 租户级 QP 隔离 | 低-中 | 中（需 DPU 部署） |
| **RNIC 调度重构** | Palos (HPCC'24) | 数据块级 | 中 | 高（需 NIC 硬件重设计） |
| **容器/平台级隔离** | Slingshot K8s (Cluster'25) | 容器级 | 低 | 中（驱动/CNI 扩展） |

核心趋势：从粗粒度带宽隔离（SR-IOV）向微架构资源感知的细粒度隔离演进，DPU/SmartNIC 成为关键隔离执行点。

## 7. 问题与备注

1. arXiv 和 Semantic Scholar API 均触发 429 rate limit，部分论文元数据通过 WebSearch 补充，引用数等字段可能不完整（标注为 `-`）
2. Palos (HPCC 2024) 为 IEEE 付费内容，[CCF 未收录]，但作为 2024 年 RNIC 调度方向的代表性系统方案值得纳入
3. GLSVLSI TX Architecture 论文虽有系统实现但属于硬件电路级工作，隔离性评估规模较小
4. Canvas 虽聚焦 RDMA 远内存交换而非通用 RDMA 隔离，但其二维调度器设计对多租户 RDMA 资源调度有参考价值
5. 分布式训练通信方向（G1-G7）与本 query 的核心多租户隔离目标相关但非直接对应，纳入作为应用背景补充

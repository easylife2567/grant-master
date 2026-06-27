# 第 01 轮文献查找 — 综合调研报告

## 调研概览

| 指标 | 数值 |
|------|------|
| 调度 query 数 | 5 条 |
| 并行 searcher agent 数 | 5 |
| 搜索平台 | arXiv, Semantic Scholar, WebSearch |
| 时间范围 | 2018-2026（重点 2021-2026） |
| 原始命中 | ~754 篇（去重前） |
| 候选论文总数 | **48 篇**（core 26，general 22） |
| 2025-2026 最新论文 | 19 篇 |
| 已自动下载 PDF | 44 篇（papers/inbox/） |
| 非 OA 需机构获取 | 2 篇（Husky NSDI'23, Palos HPCC'24） |
| 已识别重复论文 | 约 33 篇跨 query 重复已合并去重 |

## 各 Query 摘要

### Q1 — RDMA 多租户性能隔离（C1 主查询）
- **命中/筛选**：80 → 16 篇（core 9，general 7）
- **核心发现**：RDMA 多租户隔离已形成 5 条技术路线：
  1. **软件抢占**（PeRF ATC'24）：纯软件 RNIC 抢占，吞吐提升 2.04x
  2. **微架构感知硬软协同**（Husky NSDI'23 + Harmonic NSDI'24）：首篇揭示 RNIC 微架构资源是隔离瓶颈，FPGA 辅助实现
  3. **DPU/SmartNIC 卸载**（Palladium SIGCOMM'25, OSMOSIS 2023）：DPU 反向代理隔离租户对 RDMA 的访问
  4. **RNIC 调度重构**（Palos HPCC'24, TX Architecture GLSVLSI'23）：从包级改为数据块级调度
  5. **平台级容器隔离**（Slingshot K8s 2025）：HPE 互连在 Kubernetes 的多租户方案
- **下载**：10 篇 OA PDF 已下载；2 篇非 OA（Palos, TX Architecture）记录为 needs_institution

### Q2 — 分布式训练通信调度（C2 主查询）
- **命中/筛选**：128 → 15 篇（core 8，general 7）
- **核心发现**：通信调度覆盖三个维度：
  1. **优先级调度**（TicTac MLSys'18, P3 SysML'19, DLCP 2020, DynaComm JSAC'21）
  2. **压缩协同**（MergeComp 2021）
  3. **拓扑感知调度**（Arnold NeurIPS'25, CrossPipe ATC'25）
- **趋势**：近 3 年明显转向 LLM 训练场景（DHelix 2024, Arnold 2025, ScaleAcross 2026）
- **下载**：10 篇 OA PDF 已下载

### Q3 — NCCL 多任务/资源隔离
- **命中/筛选**：110 → 15 篇（core 8，general 7）
- **核心发现**：关键洞察——NCCL 自身不区分不同训练任务，其 process group 设计假设固定 GPU 集合和共享故障域。多租户 GPU 集群中的通信资源管理和隔离完全依赖上层调度系统。
- **实证分析关键工作**：Compute-Comm Overlap（2025）定量证明计算-通信重叠可导致平均 18.9% 计算减速；When Scaling Fails（2026）揭示生产集群中扩展失败的重复性故障模式
- **下载**：8 篇 OA PDF 已下载

### Q4 — RDMA 隔离 × ML 训练交叉
- **命中/筛选**：272 → 20 篇（core 8，general 12）
- **核心发现**：三个新趋势：
  1. **ML-感知 RDMA 传输**（OptiNIC 2025, Celeris 2025）：重新审视 RDMA 可靠性假设以适配 ML 容错
  2. **软件传输层可编程 RDMA**（GPU Networking 2025）：将 NIC 控制路径解耦至主机 CPU
  3. **跨平台可移植通信**（UCCL-EP, HetCCL 2026）：统一 NVIDIA/AMD GPU 的 RDMA 通信后端
- **下载**：7 篇 OA PDF 已下载；2 篇非 OA（Husky NSDI'23, Harmonic NSDI'24）记录为 needs_institution

### Q5 — 多任务训练网络干扰实证
- **命中/筛选**：164 → 15 篇（core 8，general 7）
- **核心发现**：多任务网络干扰的定量证据链已形成：
  - 拥塞控制方案对训练性能影响有限（Khan et al. 2022）：集合通信天然避免 incast → 需要训练专有方案
  - RL-based 调度可降低作业完成时间 18.2%（Ryu & Eo 2023）
  - vClos（2023）+ Arnold（2025）提供了网络拓扑与通信联合优化的完整方案
- **下载**：8 篇 OA PDF 已下载

## 重要论文摘要（Core，按技术路线分组）

### 组 1：RDMA 多租户微架构与隔离（C1 核心）

**Husky — Understanding RDMA Microarchitecture Resources for Performance Isolation** (NSDI 2023)
- 首篇系统性揭示 RNIC 微架构资源（缓存、处理单元、PCIe 带宽）是多租户隔离瓶颈的工作
- 提出 Husky 测试套件，证明当时所有方案均无法通过
- 结果被 NVIDIA 确认复现
- 状态：NSDI OA 已下载

**Harmonic — Hardware-assisted RDMA Performance Isolation for Public Clouds** (NSDI 2024)
- 首个微架构资源感知的 RDMA 性能隔离方案
- 基于 FPGA 可编程智能 PCIe 交换机 + RDMA 友好速率限制器
- 唯一通过 Husky 测试套件的方案，正集成到下一代商用 RNIC
- 状态：NSDI OA 已下载

**PeRF — Preemption-enabled RDMA Framework** (USENIX ATC 2024)
- 纯软件 RDMA 性能隔离框架
- 利用 IB_WR_WAIT/IB_WR_ENABLE 动词实现 RNIC 抢占
- 工作保持（work-conserving）+ 无需专用硬件，吞吐提升约 2.04x
- 状态：USENIX OA 已下载

**Palos — Fair and Flexible Flow Scheduling on RNIC** (IEEE HPCC 2024)
- 揭示 RNIC 包级流调度是隔离不足根因
- 数据块级调度 + 层级权重配置，软件层可配置的灵活策略
- 状态：非 OA，需机构访问 IEEE Xplore

### 组 2：分布式训练通信调度（C2 核心）

**TicTac — Accelerating Distributed Deep Learning with Communication Scheduling** (MLSys 2018)
- 奠基性工作：针对 Parameter Server 架构中参数到达顺序随机的问题
- 通过优先级调度强制传输顺序，消除 straggler 效应，吞吐提升 19.2%
- 状态：arXiv OA 已下载

**P3 — Priority-based Parameter Propagation for Distributed DNN Training** (SysML 2019)
- 细粒度优先级参数同步：不同参数可容忍不同同步延迟
- ResNet-50/Sockeye/VGG-19 吞吐分别提升 25%/38%/66%
- 状态：arXiv OA 已下载

**DLCP — Domain-specific Communication Optimization for Distributed DNN Training** (2020)
- 利用 DL 领域特性做细粒度通信优化：SGD 有界损失容忍度 + 包级优先级调度和丢弃 + 包间无序逐包负载均衡
- 最高额外提供 84.3% 加速
- 状态：arXiv OA 已下载

**Arnold — Efficient Pre-Training of LLMs via Topology-Aware Communication Alignment** (NeurIPS 2025)
- 生产级拓扑感知调度系统，9600+ GPU 上端到端性能提升 10.6%
- 将 LLM 预训练通信模式与数据中心物理拓扑对齐
- 状态：arXiv OA 已下载

**CrossPipe — Towards Optimal Pipeline Schedules for Cross-Datacenter Training** (USENIX ATC 2025)
- 跨数据中心 LLM 训练的最优 pipeline 调度框架
- 统一分析 PP 与 DP 通信重叠，基于求解器的最优和贪心近最优算法
- 训练时间减少最高 33.6%
- 状态：arXiv OA 已下载

### 组 3：多租户训练的网络争用与隔离（C1×C2 交叉）

**vClos — Isolated Scheduling for Distributed Training Tasks in GPU Clusters** (2023)
- 系统研究 ECMP 哈希冲突导致 NCCL all-reduce 性能下降的机制
- 提出联合优化网络拓扑与通信调度消除多任务网络争用
- 状态：arXiv OA 已下载

**BandPilot — Performance- and Contention-Aware GPU Dispatching in AI Clusters** (2025)
- 分层设计：从稀疏 NCCL 测量中学习带宽模型
- 在实时调度中预估多租户干扰下的通信带宽，32-GPU H100 上达 92-97% 带宽效率
- 状态：arXiv OA 已下载

**When Scaling Fails — Network and Fabric Effects on Distributed GPU Training** (2026)
- 实证研究生产级 GPU 集群中分布式训练扩展失效的根因
- 识别同步放大、拓扑诱导争用、局部性驱动性能方差等重复性故障模式
- 状态：arXiv OA 已下载

**Network Contention-Aware Cluster Scheduling with Reinforcement Learning** (2023)
- 将 GPU 集群调度形式化为 RL 问题
- 学习捕获训练任务间网络争用敏感度的调度策略，平均 JCT 降低 18.2%
- 状态：arXiv OA 已下载

### 组 4：ML-感知 RDMA 传输新范式（创新方向）

**OptiNIC — A Resilient and Tail-Optimal RDMA NIC for Distributed ML Workloads** (2025)
- 直接面向分布式 ML 工作负载重新设计 RDMA NIC 传输层
- 取消重传和保序、允许尽力而为乱序传输，利用 ML 对部分数据丢失的容忍度
- 训练时间-精度提升 2x，P99 延迟降低 3.5x
- 状态：arXiv OA 已下载

**Reimagining RDMA Through the Lens of ML** (Celeris, 2025)
- 提出 ML-感知的 RDMA 可靠性：传统 RDMA 强可靠性在 ML 场景中造成不必要尾延迟
- Best-effort RDMA 传输，P99 延迟降低 2.3x，BRAM 使用降低 67%
- 状态：arXiv OA 已下载

**An Extensible Software Transport Layer for GPU Networking** (2025)
- 将 RDMA NIC 控制路径从硬件解耦至主机 CPU 的软件传输层
- 实现 ML 训练集合通信的多路径传输和定制化拥塞控制，最高 4.5x 性能提升
- 状态：arXiv OA 已下载

### 组 5：DPU/平台级多租户隔离方案

**Palladium — DPU-enabled Multi-Tenant Serverless Cloud over Zero-copy Multi-node RDMA Fabrics** (SIGCOMM 2025)
- DPU 上运行轻量级反向代理隔离各租户对 RDMA 资源的访问
- 实现公平带宽分配，RPS 提升 20.9x
- 状态：arXiv OA 已下载

**Slingshot RDMA for Kubernetes** (IEEE Cluster 2025)
- HPE Slingshot 200Gbps 互连在 K8s 上的多租户 RDMA 安全访问方案
- 容器粒度 RDMA 资源隔离，网络命名空间认证 + VNI 管理
- 状态：arXiv OA 已下载

## 一般论文清单（General，22 篇）

| # | 论文 | 年份 | 来源 |
|---|------|------|------|
| 1 | CSA-UD: Communication-Semantic-Aware RDMA Loss Recovery | 2026 | Q1 |
| 2 | NCCL EP: Unified Expert Parallel Communication API | 2026 | Q1/Q4 |
| 3 | UCCL-EP: Portable Expert-Parallel Communication | 2025 | Q1/Q4 |
| 4 | HetCCL: Accelerating LLM Training with Heterogeneous GPUs | 2026 | Q1/Q4 |
| 5 | ForestColl: Throughput-Optimal Collective Communications | 2024 | Q1 |
| 6 | MonkeyTree: Near-Minimal Congestion via Migration | 2026 | Q1 |
| 7 | Demystifying NCCL: In-depth Analysis of GPU Communication | 2025 | Q2 |
| 8 | EmbRace: Accelerating Sparse Communication for Distributed Training | 2023 | Q2 |
| 9 | vClos RailS: Topology-Aware Communication Scheduling | 2023 | Q2 |
| 10 | TACCL: Guiding Collective Algorithm Synthesis | NSDI 2023 | Q3 |
| 11 | TE-CCL: ML Collective Communication as Multi-Commodity Flow | SIGCOMM 2024 | Q3 |
| 12 | MSCCL++: Rethinking GPU Communication Abstractions | 2025 | Q3 |
| 13 | MFS: Multi-stage Flow Scheduling for LLM Serving | 2026 | Q3 |
| 14 | PCCL: Photonic circuit-switched collective communication | 2025 | Q3 |
| 15 | ScaleAcross Explorer: Communication Optimization for Scale-Across | 2026 | Q3/Q5 |
| 16 | Ethereal: Divide and Conquer Network Load Balancing | 2024 | Q5 |
| 17 | Hopper: Congestion-Aware Path Selection for AI Clusters | 2025 | Q4/Q5 |
| 18 | Adaptra: Straggler-Resilient Hybrid-Parallel Training | 2025 | Q4/Q5 |
| 19 | Nezha: Breaking Multi-Rail Network Barriers | 2024 | Q4 |
| 20 | GPU-Initiated Networking for NCCL (GIN) | 2025 | Q4 |
| 21 | FlexLink: Boosting NVLink Bandwidth by 27% | 2025 | Q4 |
| 22 | Resilient AI Supercomputer Networking using MRC and SRv6 | 2026 | Q5 |

## PDF 下载总览

| 状态 | 数量 | 说明 |
|------|------|------|
| 已下载 OA PDF | 44 篇 | papers/inbox/，来自 arXiv 和 USENIX OA |
| 非 OA 需机构获取 | 2 篇 | Palos（IEEE HPCC 2024）、TX Architecture（GLSVLSI 2023）|
| 未下载（非 core general 论文） | 12 篇 | 按策略 general 论文仅记录不下载 |

## 跨 Query 发现与建议

### 1. 研究格局已趋成熟，需聚焦差异化

RDMA 多租户性能隔离已形成 5 条清晰技术路线（软件抢占/硬软协同/DPU卸载/调度重构/平台隔离），2023-2025 年顶会密集发表说明该方向正处上升期。**建议**：在 synthesis 阶段重点对比这些方案的适用场景、实现复杂度和对训练场景的适配程度，明确本课题的差异化空间。

### 2. "训练感知"是关键的差异化方向

现有 RDMA 隔离方案（Husky、Harmonic、PeRF、Palos）大多针对通用云场景，未充分考虑分布式训练的以下特征：
- 通信模式的可预测性（all-reduce 周期性突发）
- ML 对部分数据丢失的容忍度（OptiNIC/Celeris 2025 验证）
- 集合通信算法的天然 anti-incast 特性（Khan et al. 2022 验证）

**建议**：将"训练通信语义感知的网络资源调度"作为本课题的核心差异化方向。

### 3. 多任务干扰的定量证据链已形成

多任务训练网络干扰的严重性已由多篇实证研究佐证（Compute-Comm Overlap 2025: 18.9% 减速；When Scaling Fails 2026: 同步放大和拓扑争用；vClos 2023: ECMP 哈希冲突导致 all-reduce 性能下降），为申请书问题陈述提供充分文献支撑。

### 4. 智能调度的方法论基础

RL-based 调度（Ryu & Eo 2023: JCT 降低 18.2%）和 BandPilot（2025: 学习带宽模型实现 92-97% 效率）展示了"智能"方法在该场景的可行性，可为本课题的方法选择提供 baseline。

### 5. 工业实践趋势

Meta 的跨数据中心训练优化（ScaleAcross 2026: 64.62% 加速）、OpenAI/Microsoft 的 MRC 传输协议（2026）、NVIDIA 的 GIN 和 NCCL EP（2025-2026）表明工业界正积极投入分布式训练的网络优化。这既说明问题的重要性，也提示需要关注学术创新与工业方案的差异。

### 6. 下一轮调研建议

- **深度对比**：精读 C1 核心论文（Husky、Harmonic、PeRF），对比隔离粒度和训练场景适配度
- **补充方向**：若本轮 synthesis 发现训练通信特征利用不足的 gap 明确，可安排 T1（拥塞控制公平性）和 T2（智能调度方法）的精读
- **T3（工业实践调研）可考虑跳过**：工业界方案已在搜索中部分覆盖，且以工程方案为主，学术创新空间有限

# Query Q2 搜索报告

**查询式**：distributed training communication scheduling network optimization
**平台**：arXiv (Semantic Scholar 不可用 -- 429 速率限制)
**时间**：2026-06-27T17:45:00+08:00

## 1. 搜索策略

### 搜索目标

系统调研分布式训练中针对通信效率的调度和网络优化系统工作。按搜索协议展开为多轮互补 query：
1. 主 query：`distributed training communication scheduling network optimization` (99 结果)
2. NCCL/collection：`NCCL optimization communication collective` (17 结果)
3. 拓扑感知：`topology-aware communication distributed training scheduling` (3 结果)
4. 通信压缩：`communication compression gradient distributed training scheduling` (9 结果)

### 筛选标准

- **年份**：2019-2026 (一篇 2018 奠基性工作 TicTac 例外纳入)
- **Venue 偏好**：CCF-A/B，优先 MLSys/NSDI/SC/EuroSys
- **相关性过滤**：排除纯训练算法优化、联邦学习、无线网络调度等非系统工作
- **多样性**：覆盖拓扑感知调度、压缩协同、优先级调度、NCCL 优化等维度

### 平台说明

Semantic Scholar API 在整个 session 中持续返回 429，无法获取引用数数据。所有元数据来自 arXiv API，引用数标注为 N/A。

## 2. 结果总览

| 指标 | 数值 |
|------|------|
| 总检索 | ~128 (4 queries 合并) |
| 去重后相关论文 | 28 |
| 筛选后输出 | 15 |
| core (重要) | 8 |
| general (一般) | 7 |

## 3. 重要论文 (core)

### [TicTac: Accelerating Distributed Deep Learning with Communication Scheduling](https://arxiv.org/abs/1803.03288) (2018 MLSys)

分布式训练通信调度的奠基性工作。针对 Parameter Server 架构中参数到达顺序随机的方差问题，TicTac 通过优先级调度强制传输顺序，在保证近优计算-通信重叠的同时消除 straggler 效应。在 TensorFlow 上实现，训练吞吐提升 19.2%，straggler 效应降低 2.3x。虽发表于 2018 年，但其优先级调度思想深刻影响了后续工作。

- 链接：https://arxiv.org/abs/1803.03288 | OA：open_pdf | PDF：papers/inbox/2018_TicTac_..._1b759e2b78e6.pdf | 引用：N/A

### [P3: Priority-based Parameter Propagation for Distributed DNN Training](https://arxiv.org/abs/1905.03960) (2019 SysML)

提出细粒度的优先级参数同步机制 P3。核心洞察：(1) 通信与模型实现可使用不同数据表示粒度；(2) 不同参数可容忍不同同步延迟。P3 在更细粒度上调度数据传输，使训练过程通信延迟最小化。ResNet-50/Sockeye/VGG-19 吞吐分别提升 25%/38%/66%。

- 链接：https://arxiv.org/abs/1905.03960 | OA：open_pdf | PDF：papers/inbox/2019_Priority-based_..._cba7a84c34a0.pdf | 引用：N/A

### [DLCP: Domain-specific Communication Optimization for Distributed DNN Training](https://arxiv.org/abs/2008.08445) (2020)

利用 DL 领域特性进行细粒度通信优化。核心创新：(1) 利用 SGD 有界损失容忍度优化尾部通信延迟；(2) 基于梯度层级和大小的包级优先级调度和丢弃；(3) 利用包间无序性做逐包负载均衡。兼容 PS 和 collective 通信，在 10 卡 V100 测试床上额外提供最高 84.3% 加速。

- 链接：https://arxiv.org/abs/2008.08445 | OA：open_pdf | PDF：papers/inbox/2020_Domain-specific_..._d6a21b768315.pdf | 引用：N/A

### [MergeComp: A Compression Scheduler for Scalable Communication-Efficient Distributed Training](https://arxiv.org/abs/2103.15195) (2021)

针对梯度压缩在某些情况下反而降低训练性能的问题，提出压缩调度器 MergeComp。无需模型架构或系统参数知识，自动调度压缩操作的时间点和方式以优化压缩算法性能。在 9 种主流压缩算法上验证，性能提升最高 3.83x，高速网络上扩展效率达 99%。

- 链接：https://arxiv.org/abs/2103.15195 | OA：open_pdf | PDF：papers/inbox/2021_MergeComp_..._52e8177931fb.pdf | 引用：N/A

### [DynaComm: Accelerating Distributed CNN Training between Edges and Clouds through Dynamic Communication Scheduling](https://arxiv.org/abs/2101.07968) (2021 IEEE JSAC)

面向边云协同训练场景，提出动态通信调度器 DynaComm。将每次传输过程动态分解为多个段，实现运行时最优的逐层通信与计算重叠。无需修改模型精度即可在所有测试场景中达到最优调度。

- 链接：https://arxiv.org/abs/2101.07968 | OA：open_pdf | PDF：papers/inbox/2021_DynaComm_..._ef0a266b4997.pdf | 引用：N/A

### [DHelix: Hiding Communication Cost in Distributed LLM Training via Micro-batch Co-execution](https://arxiv.org/abs/2411.15871) (2024)

受 DNA 双螺旋结构启发，提出微结构 DHelix 隐藏 LLM 训练中的通信开销。核心是 Strand Interleaving (SI)：将连续微批次视为两股"链"，交错排列前向和后向传播，通过动态规划搜索最优协同调度方案。在 A40/A800/H100 集群上实现 12-40% MFU 提升，且仅需不到 3% 额外显存。

- 链接：https://arxiv.org/abs/2411.15871 | OA：open_pdf | PDF：papers/inbox/2024_DHelix_..._a27be85edb8b.pdf | 引用：N/A

### [CrossPipe: Towards Optimal Pipeline Schedules for Cross-Datacenter Training](https://arxiv.org/abs/2507.00217) (2025 USENIX ATC)

面向跨数据中心 LLM 训练的 pipeline 调度框架。显式建模网络延迟和带宽限制，统一分析 pipeline parallelism 与 data parallelism 通信重叠机会。提供基于求解器的最优和近最优贪心调度算法。在内存约束下训练时间减少最高 33.6%。

- 链接：https://arxiv.org/abs/2507.00217 | OA：open_pdf | PDF：papers/inbox/2025_CrossPipe_..._6f22ba8447dc.pdf | 引用：N/A

### [Arnold: Efficient Pre-Training of LLMs via Topology-Aware Communication Alignment on More Than 9600 GPUs](https://arxiv.org/abs/2509.15940) (2025 NeurIPS)

生产级拓扑感知通信调度系统。对 LLM 预训练的通信模式进行深入特征研究，识别物理网络拓扑对训练的影响。开发调度算法将通信模式与现代数据中心物理拓扑对齐，最大通信组散布降低 1.67x。在 9600+ GPU 上训练端到端性能提升 10.6%。

- 链接：https://arxiv.org/abs/2509.15940 | OA：open_pdf | PDF：papers/inbox/2025_Arnold_..._56448e3a5d52.pdf | 引用：N/A

## 4. 一般论文清单

| # | 标题 | 年份 | Venue | 引用 | OA | 链接 |
|---|------|------|-------|------|----|------|
| 1 | ScaleAcross Explorer: Exploring Communication Optimization for Scale-Across AI Model Training | 2026 | (Meta production) | N/A | open_pdf | [arXiv](https://arxiv.org/abs/2605.24326) |
| 2 | Demystifying NCCL: An In-depth Analysis of GPU Communication Protocols and Algorithms | 2025 | -- | N/A | open_pdf | [arXiv](https://arxiv.org/abs/2507.04786) |
| 3 | RailS: Load Balancing for All-to-All Communication in Distributed MoE Training | 2025 | -- | N/A | open_pdf | [arXiv](https://arxiv.org/abs/2510.19262) |
| 4 | GADGET: Online Resource Optimization for Scheduling Ring-All-Reduce Learning Jobs | 2022 | IEEE INFOCOM [CCF-A] | N/A | open_pdf | [arXiv](https://arxiv.org/abs/2202.01158) |
| 5 | Isolated Scheduling for Distributed Training Tasks in GPU Clusters (vClos) | 2023 | -- | N/A | open_pdf | [arXiv](https://arxiv.org/abs/2308.05692) |
| 6 | EmbRace: Accelerating Sparse Communication for Distributed Training of NLP Neural Networks | 2021 | -- | N/A | open_pdf | [arXiv](https://arxiv.org/abs/2110.09132) |
| 7 | A-SRPT: Prediction-Assisted Online Distributed Deep Learning Workload Scheduling in GPU Clusters | 2025 | IEEE INFOCOM [CCF-A] | N/A | open_pdf | [arXiv](https://arxiv.org/abs/2501.05563) |

## 5. PDF 下载结果

| 状态 | 数量 |
|------|------|
| core + open_pdf (应下载) | 10 |
| 已下载 | 10 |
| 下载失败 | 0 |
| 非 OA / 需合法获取 | 0 |

所有 core 论文均来自 arXiv，均为合法 OA 下载。下载源均为 arXiv PDF 直链。

## 6. 问题与备注

1. **Semantic Scholar 不可用**：整个 session 中 S2 API 持续返回 429，无法获取引用数数据。如需精确引用数，建议后续 Coordinator 利用 Semantic Scholar batch API 批量补全。
2. **Venue 信息不完整**：arXiv API 返回的论文中部分未标注最终发表 venue（如 DHelix、RailS、Nezha 等预印本）。建议后续精读阶段核实最终发表信息。
3. **覆盖维度**：本次搜索覆盖了通信调度的三个主要维度：(a) 优先级调度 (TicTac, P3, DLCP, DynaComm)，(b) 压缩协同 (MergeComp, EmbRace)，(c) 拓扑感知调度 (Arnold, CrossPipe, vClos, RailS)。NCCL 层次的工作 (Demystifying NCCL) 提供了底层机制理解的重要补充。

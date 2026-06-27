# Query Q3 搜索报告

**查询式**：NCCL multi-job concurrent resource sharing isolation GPU communication
**平台**：arXiv API, Semantic Scholar API（S2 遇 429 限流，改为 arXiv 主搜 + WebSearch 补充）
**时间**：2026-06-27T09:43:00Z

## 1. 搜索策略

围绕核心问题「NCCL 等通信库在多任务/多租户场景下是否有资源管理和隔离机制」展开 4 路互补 arXiv 搜索 + 3 路 WebSearch 补充：

| # | 平台 | 查询 | 总结果 |
|---|------|------|--------|
| 1 | arXiv | `all:NCCL AND all:multi-tenant AND all:isolation` | 0 |
| 2 | arXiv | `all:NCCL AND all:resource AND all:sharing AND all:GPU` | 3 |
| 3 | arXiv | `all:GPU AND all:communication AND all:contention AND all:cluster` | 25 |
| 4 | arXiv | `all:distributed AND all:training AND all:communication AND all:scheduling AND all:GPU` | 82 |
| 5 | WebSearch | NCCL multi-tenant GPU cluster resource isolation | 补充 MIG/调度类 |
| 6 | WebSearch | collective communication scheduler TACCL SCCL MSCCL | 补充通信算法合成类 |
| 7 | WebSearch | MSCCL++ GPU-driven communication stack | 1 |

去重后候选集合 35 篇。按以下优先级筛选：
1. 时效性：近 6 个月标注 `[新]` 置顶
2. 相关性：与多任务通信隔离/资源管理直接相关的 > 单任务优化 > 推理/非训练场景
3. 权威性：CCF-A/B 顶会优先（NSDI、SIGCOMM、NeurIPS、MLSys）
4. 引用数（S2 API 429 限流无法获取，主要以 venue 和年份判断学术价值）

## 2. 结果总览

| 指标 | 数值 |
|------|------|
| 总检索 / 筛选后 | 110+ / 15 |
| core / general | 8 / 7 |
| CCF-A 顶会 | 3 (NSDI'23 x1, SIGCOMM'24 x1, NeurIPS'25 x1) |
| MLSys 审稿中 | 1 (Flex-MIG) |
| arXiv 预印本 | 11 |
| 近 6 个月 [新] | 4 |

## 3. 重要论文（core）

### [1] BandPilot: Towards Performance- and Contention-Aware GPU Dispatching in AI Clusters（2025，arXiv，[新]）
直接面向多租户 AI 集群的 GPU 通信调度问题，通过分层设计从稀疏 NCCL 测量中学习带宽模型，并在实时调度中预估多租户干扰下的实际通信带宽。H100 32-GPU 集群上达到 92-97% 带宽效率，较拓扑紧凑启发式提升 20-40%。
- 链接：https://arxiv.org/abs/2506.15595 | OA：open_pdf | PDF：papers/inbox/2025_BandPilot_2506.15595.pdf

### [2] VCCL: An Efficient, Reliable and Observable Collective Communication Library in Large-scale GPU Training Clusters（2025，arXiv）
生产环境 NCCL 替代方案。揭示 NCCL 在大规模 GPU 训练集群中的三大实践局限：SM 计算-通信竞争、链路故障昂贵重启代价、瞬态通信异常不可观测。提出 VCCL 移除 SM 占用 P2P 内核，引入主备 QP 机制容错，设计窗口监控器。已开源并在生产集群部署数月，训练吞吐提升最高 5.28%。
- 链接：https://arxiv.org/abs/2510.00991 | OA：open_pdf | PDF：papers/inbox/2025_VCCL_2510.00991.pdf

### [3] MultiWorld: Enabling Elastic Model Serving with MultiWorld（2024，arXiv）
直接指出 NCCL 等 CCL 在推理场景的根本局限：CCL process group 内所有 worker 共享单一故障域且无法随负载增长动态扩缩。提出 MultiWorld 实现 worker 粒度故障容忍和在线扩缩，在多数场景下吞吐损失仅 1.4-4.3%。
- 链接：https://arxiv.org/abs/2407.08980 | OA：open_pdf | PDF：papers/inbox/2024_MultiWorld_2407.08980.pdf

### [4] Flex-MIG: Enabling Distributed Execution on MIG（2025，MLSys 2026 审稿中）
软件方案替代 MIG 硬件一对一的 GPU 分配模型，实现一对多分配。关键是修改 NCCL 的 Bus ID 重复检测 bug，实现跨 MIG 实例的 Host Shared Memory 集合通信，解决多租户场景下 MIG 分区间的通信隔离与共享矛盾。
- 链接：https://arxiv.org/abs/2511.09143 | OA：open_pdf | PDF：papers/inbox/2025_FlexMIG_2511.09143.pdf

### [5] Isolated Scheduling for Distributed Training Tasks in GPU Clusters (vClos)（2023，arXiv）
系统研究 GPU 集群中分布式训练的网络争用问题——ECMP 哈希冲突导致 NCCL all-reduce 性能严重下降。提出 vClos 联合优化网络拓扑与通信调度消除争用，并设计 OCS-vClos 变体降低资源碎片化。
- 链接：https://arxiv.org/abs/2308.05692 | OA：open_pdf | PDF：papers/inbox/2023_vClos_2308.05692.pdf

### [6] Arnold: Efficient Pre-Training of LLMs via Topology-Aware Communication Alignment on More Than 9600 GPUs（2025，NeurIPS 2025 [CCF-A]）
从数据中心物理网络拓扑角度研究 LLM 预训练的通信争用：GPU 间稀疏但高容量的突发通信模式在不合理资源调度下产生严重的带宽争用。提出 Arnold 调度系统，在 9600+ GPU 上实现端到端 10.6% 性能提升。
- 链接：https://arxiv.org/abs/2509.15940 | OA：open_pdf | PDF：papers/inbox/2025_Arnold_2509.15940.pdf

### [7] Characterizing Compute-Communication Overlap in GPU-Accelerated Distributed Deep Learning（2025，arXiv，[新]）
定量研究分布式训练中计算-通信重叠对 GPU 资源争用的影响。系统性实验揭示：通信与计算重叠可导致平均 18.9%、最高 40% 的计算减速（相比无通信时），并增加功耗。对理解多任务共享 GPU 时的通信干扰有直接定量价值。
- 链接：https://arxiv.org/abs/2507.03114 | OA：open_pdf | PDF：papers/inbox/2025_ComputeCommOverlap_2507.03114.pdf

### [8] When Scaling Fails: Network and Fabric Effects on Distributed GPU Training Performance（2026，arXiv，[新]）
通过多个生产级集群的实证研究揭示分布式 GPU 训练扩展失败的根因：网络拓扑、拥塞动态、集合通信同步行为和 GPU 局部性在跨节点扩展时主导端到端性能。识别出同步放大、拓扑诱导争用和局部性驱动性能方差等重复性故障模式。
- 链接：https://arxiv.org/abs/2603.04424 | OA：open_pdf | PDF：papers/inbox/2026_WhenScalingFails_2603.04424.pdf

## 4. 一般论文清单

| # | 标题 | 年份 | Venue | 引用 | OA |
|---|------|------|-------|------|----|
| 9 | TACCL: Guiding Collective Algorithm Synthesis using Communication Sketches | 2023 | NSDI [CCF-A] | N/A (S2 429) | open_pdf |
| 10 | Rethinking ML Collective Communication as a Multi-Commodity Flow Problem (TE-CCL) | 2024 | SIGCOMM [CCF-A] | N/A (S2 429) | open_pdf |
| 11 | FiCCO: Design Space Exploration of DMA based Finer-Grain Compute Communication Overlap | 2025 | arXiv [预印本] | N/A | open_pdf |
| 12 | MFS: Multi-stage Flow Scheduling for LLM Serving | 2026 | arXiv [预印本] | N/A | open_pdf |
| 13 | MSCCL++: Rethinking GPU Communication Abstractions for Cutting-edge AI Applications | 2025 | arXiv [预印本] | N/A | open_pdf |
| 14 | PCCL: Photonic circuit-switched collective communication for distributed ML | 2025 | arXiv [预印本] | N/A | open_pdf |
| 15 | ScaleAcross Explorer: Exploring Communication Optimization for Scale-Across AI Model Training | 2026 | arXiv [预印本] | N/A | open_pdf |

**链接**：
- #9 [TACCL](https://arxiv.org/abs/2111.04867)
- #10 [TE-CCL](https://arxiv.org/abs/2305.13479)
- #11 [FiCCO](https://arxiv.org/abs/2512.10236)
- #12 [MFS](https://arxiv.org/abs/2603.17456)
- #13 [MSCCL++](https://arxiv.org/abs/2504.09014)
- #14 [PCCL](https://arxiv.org/abs/2509.15450)
- #15 [ScaleAcross Explorer](https://arxiv.org/abs/2605.24326)

## 5. PDF 下载结果

| 状态 | 数量 |
|------|------|
| 应下载（core+OA）/ 已下载 | 8 / 8 |
| 非 OA / 需合法获取 | 0 / 0 |

所有 core 论文均来自 arXiv（全部 open_pdf），8 篇全部成功下载。

## 6. 关键发现与备注

1. **NCCL 自身没有多租户隔离机制**：根据 MultiWorld 和 Flex-MIG 的分析，NCCL 的 process group 设计假设固定 GPU 集合、共享故障域，不区分不同训练任务。多任务资源共享时的隔离完全依赖上层调度系统。

2. **多租户通信争用是现实问题**：BandPilot 和 vClos 从不同角度证明，GPU 集群中的通信争用（网络拓扑、ECMP 哈希冲突、NIC 饱和）对分布式训练性能有显著影响，且现有静态启发式调度无法有效应对。

3. **通信-计算重叠引入额外资源争用**：Characterizing Compute-Communication Overlap 的定量证据表明，重叠执行本身可导致最高 40% 的计算减速，这对理解多任务共享 GPU 时的性能干扰至关重要。

4. **S2 API 429 限流**：Semantic Scholar API 无 Key 时速率限制极低，本次搜索中多次触发 429。建议配置 S2 API Key 以获取准确的引用数数据。

5. **非学术过滤**：未纳入博客、新闻稿、Reddit 等非学术来源。搜索结果全部来自 arXiv 和学术会议（NSDI、SIGCOMM、NeurIPS）。

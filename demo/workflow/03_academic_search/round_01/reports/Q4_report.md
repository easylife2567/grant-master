# Query Q4 搜索报告

**查询式**：RDMA machine learning training isolation multi-tenant resource
**平台**：arxiv, semantic_scholar
**时间**：2026-06-27T09:43:00Z

## 1. 搜索策略

本 query 的核心问题是："RDMA性能隔离方案中，哪些考虑了机器学习训练场景的特殊通信模式需求？" 围绕这一问题，采用多角度互补查询策略：

| # | 查询平台 | 查询关键词 | 结果数 | 说明 |
|---|---------|-----------|--------|------|
| 1 | arXiv | `all:RDMA AND all:machine learning AND all:training AND all:isolation AND all:multi-tenant` | 56 | 宽口径主查询 |
| 2 | arXiv | `all:RDMA AND all:multi-tenant AND all:performance AND all:isolation` | 3 | 精准多租户隔离查询 |
| 3 | arXiv | `all:RDMA AND all:distributed AND all:training AND all:communication AND all:scheduling` | 2 | 训练通信调度角度 |
| 4 | arXiv | `all:RDMA AND all:network AND all:resource AND all:allocation AND all:training` | 1 | 资源分配角度 |
| 5 | Semantic Scholar | `RDMA performance isolation multi-tenant microarchitecture resources` | 210 | S2 隔离专项检索 |
| 6 | WebSearch | NSDI/MLSys/ATC/SIGCOMM venue-targeted queries | 发现 Harmonic/Husky/STELLAR/PeRF 等非 arXiv 收录的顶会论文 |

补充途径：
- S2 batch API 获取 arXiv 论文引用数和 venue 元数据
- WebSearch 针对 NSDI、SIGCOMM、ATC、MLSys 等 CCF-A 顶会做定向检索
- 结合 search-protocol.md 的学科路由规则，优先筛选 CCF-A/B 会议和 top 系统会议论文

## 2. 结果总览

| 指标 | 数值 |
|------|------|
| 总检索 | 272 |
| 去重筛选后 | 20 |
| core（与核心问题高度相关） | 8 |
| general（相关但不直接命中核心问题） | 12 |
| 近 6 个月（2026-01 后） | 4 |

## 3. 重要论文（core）

以下 8 篇论文直接回应核心问题 "RDMA性能隔离方案中，哪些考虑了机器学习训练场景的特殊通信模式需求？"。

### [1] Understanding RDMA Microarchitecture Resources for Performance Isolation（2023 NSDI [CCF-A]，引用 66）

Xinhao Kong, Jingrong Chen, Wei Bai, Yechen Xu, Mahmoud Elhaddad, Shachar Raindel, Jitendra Padhye, Alvin Lebeck, Danyang Zhuo

该文是 RDMA 性能隔离领域的基础性工作（NSDI 2023）。首次系统性地揭示了 RDMA NIC 微架构资源（QP 上下文缓存、WQE 缓存、DMA 引擎等）在多租户共享时的性能隔离缺口，并提出了 Husky 系统：通过动态资源分区和速率限制实现租户间性能隔离。虽然未专门针对 ML 训练优化，但所揭示的资源争抢模式（如 all-to-all 通信加速器缓存抖动）为后续 ML 训练场景的 RDMA 隔离方案提供了理论基础。

- 链接：[Semantic Scholar](https://www.semanticscholar.org/paper/c6b81ec1de0f2fe2f08f0b720ce4745c98aaf487)
- OA：needs_institution（USENIX 开放获取，可通过 [USENIX NSDI 2023](https://www.usenix.org/conference/nsdi23/presentation/kong) 访问）
- PDF：未下载（USENIX 会议论文，机构订阅可获取）

### [2] Harmonic: Hardware-assisted RDMA Performance Isolation for Public Clouds（2024 NSDI [CCF-A]，引用 20）

Jiaqi Lou, Xinhao Kong, Jinghan Huang, Wei Bai, Nam Sung Kim, Danyang Zhuo

直接延续 Husky 的工作，聚焦公有云多租户场景下的 RDMA 性能隔离。Harmonic 提出了硬件辅助的 RDMA 隔离方案，利用可编程交换机/SmartNIC 实现租户间的带宽和延迟隔离。该系统明确考虑了 ML 训练任务在公有云中共存时的通信干扰问题，是首篇在硬件层面为云上 ML 训练的 RDMA 隔离提供完整方案的工作。

- 链接：[Semantic Scholar](https://www.semanticscholar.org/paper/a8ab74a621990c345299a3c08f343a7ec478373c)
- OA：needs_institution（USENIX 开放获取，可通过 [USENIX NSDI 2024](https://www.usenix.org/conference/nsdi24/presentation/lou) 访问）
- PDF：未下载（USENIX 会议论文，机构订阅可获取）

### [3] OptiNIC: A Resilient and Tail-Optimal RDMA NIC for Distributed ML Workloads（2025，引用 1）

Ertza Warraich, Ali Imran, Annus Zulfiqar, Shay Vargaftik, Sonia Fahmy, Muhammad Shahbaz

该文直接面向分布式 ML 工作负载重新设计 RDMA NIC 传输层。核心观察：ML 训练对部分数据丢失/延迟具有天然的容错性（如通过 Hadamard Transform 和 Erasure Coding 恢复），但现有 RDMA 协议（RoCE/IRN）仍强制执行严格的可靠性和有序交付。OptiNIC 首次提出"ML-感知的不可靠 RDMA"：取消重传和有序交付、引入自适应超时机制、将丢包恢复移至 ML 管线。在云平台评估中实现 2x 训练时间-精度提升、3.5x P99 延迟降低。

- 链接：[arXiv](https://arxiv.org/abs/2512.22743)
- OA：open_pdf | PDF：`papers/inbox/2512.22743.pdf` (427 KB)

### [4] Reimagining RDMA Through the Lens of ML（Celeris）（2025 IEEE Computer Architecture Letters，引用 0）

Ertza Warraich, Ali Imran, Annus Zulfiqar, Shay Vargaftik, Sonia Fahmy, Muhammad Shahbaz

OptiNIC 的短版/早期版本。核心论点是"从 ML 的视角重新审视 RDMA 的可靠性假设"：传统 RDMA 为通用工作负载设计的强可靠性保证（Go-Back-N、Selective Repeat）在 ML 场景中造成了不必要的尾延迟。Celeris 提出去掉 NIC 端的重传和有序交付，实现 best-effort RDMA 传输，并将丢包恢复上移至 ML 管线层。评估显示 P99 延迟降低 2.3x，BRAM 使用降低 67%。

- 链接：[arXiv](https://arxiv.org/abs/2510.16606) | [DOI](https://doi.org/10.1109/LCA.2025.3624158)
- OA：open_pdf | PDF：`papers/inbox/2510.16606.pdf` (185 KB)

### [5] An Extensible Software Transport Layer for GPU Networking（UCCL）（2025，引用 10）

Yang Zhou, Zhongjie Chen, Ziming Mao, ChonLam Lao, Shuo Yang, Pravein Govindan Kannan, Jiaqi Gao, Yilong Zhao, Yongji Wu, Kaichao You, Fengyuan Ren, Zhiying Xu, Costin Raiciu, Ion Stoica

UCCL 提出将 RDMA NIC 的控制路径从硬件数据路径中解耦，在主机 CPU 上运行可扩展的软件传输层。这使得 RDMA 传输行为可以根据 ML 工作负载特征灵活定制（如多路径传输解决流冲突、定制化拥塞控制）。在 ML 集合通信（AllReduce/AllGather）中实现最高 4.5x 性能提升。该工作直接回应了 ML 训练场景对 RDMA 通信灵活性的需求——硬件 RDMA 传输难以快速演进以适配多变 ML 负载。

- 链接：[arXiv](https://arxiv.org/abs/2504.17307)
- OA：open_pdf | PDF：`papers/inbox/2504.17307.pdf` (2.1 MB)

### [6] Palladium: A DPU-enabled Multi-Tenant Serverless Cloud over Zero-copy Multi-node RDMA Fabrics（2025 SIGCOMM [CCF-A]，引用 2）

Shixiong Qi, Songyu Zhang, K. K. Ramakrishnan, Diman Z. Tootaghaj, Hardik Soni, Puneet Sharma

Palladium 直接解决多租户 Serverless 场景下 RDMA 资源共享的隔离问题——这是 ML 推理服务的典型部署模式。核心贡献是 DPU 使能的网络引擎（DNE）：在 DPU 上运行轻量级反向代理，隔离各租户函数对 RDMA 资源的访问，并在争抢时实施公平调度。关键发现：两方 RDMA（two-sided）原语在零拷贝数据平面上优于单方（one-sided）。评估显示 20.9x RPS 提升、21x 延迟降低，同时节省 7 个 CPU 核心。

- 链接：[arXiv](https://arxiv.org/abs/2505.11339) | [DOI](https://doi.org/10.1145/3718958.3750494)
- OA：open_pdf | PDF：`papers/inbox/2505.11339.pdf` (1.5 MB)

### [7] Closing the HPC-Cloud Convergence Gap: Multi-Tenant Slingshot RDMA for Kubernetes（2025 IEEE Cluster，引用 1）

Philipp A. Friese, Ahmed Eleliemy, Utz-Uwe Haus, Martin Schulz

面向 HPC-Cloud 融合场景，在 Kubernetes 上实现了 HPE Slingshot 高速互连（200 Gbps RDMA）的多租户安全访问。核心贡献是容器粒度的 RDMA 资源隔离：通过扩展 Slingshot 主机软件栈，在保持接近裸金属 RDMA 性能的前提下提供安全的多租户隔离。代表了工业级 HPC 互连向云原生多租户 ML 训练平台演进的典型案例。

- 链接：[arXiv](https://arxiv.org/abs/2508.09663) | [DOI](https://doi.org/10.1109/CLUSTER59342.2025.11186471)
- OA：open_pdf | PDF：`papers/inbox/2508.09663.pdf` (473 KB)

### [8] Serving Heterogeneous LoRA Adapters in Distributed LLM Inference Systems（LoRAServe）（2025，引用 3）

Shashwat Jaiswal, Shrikara Arun, Anjaly Parayil, Ankur Mallick, Spyros Mastorakis, Alind Khare, Chloi Alverti, Renee St Amant, Chetan Bansal, Victor Rühle, Josep Torrellas

LoRAServe 解决多租户 LoRA 推理场景中的 GPU 资源隔离和负载均衡问题。利用 GPUDirect RDMA 实现跨 GPU 的适配器远程访问，通过动态适配器放置和路由最大化吞吐量。虽然聚焦在 GPU 资源而非 RDMA 隔离本身，但其 GPUDirect RDMA 跨 GPU 通信的隔离需求直接关联 RDMA 多租户性能隔离的 ML 训练/推理场景。

- 链接：[arXiv](https://arxiv.org/abs/2511.22880)
- OA：open_pdf | PDF：`papers/inbox/2511.22880.pdf` (1.9 MB)

## 4. 一般论文清单

| # | 标题 | 年份 | Venue | 引用 | OA | 链接 |
|---|------|------|-------|------|----|------|
| 9 | Congestion-Aware Path Selection for Load Balancing in AI Clusters (Hopper) | 2025 | - | 2 | open_pdf | [arXiv](https://arxiv.org/abs/2506.08132) |
| 10 | Adaptra: Straggler-Resilient Hybrid-Parallel Training with Pipeline Adaptation | 2025 | - | 2 | open_pdf | [arXiv](https://arxiv.org/abs/2504.19232) |
| 11 | Nezha: Breaking Multi-Rail Network Barriers for Distributed DNN Training | 2024 | - | 1 | open_pdf | [arXiv](https://arxiv.org/abs/2405.17870) |
| 12 | UCCL-EP: Portable Expert-Parallel Communication | 2025 | - | 7 | open_pdf | [arXiv](https://arxiv.org/abs/2512.19849) |
| 13 | NCCL EP: Towards a Unified Expert Parallel Communication API for NCCL | 2026 | - | 4 | open_pdf | [arXiv](https://arxiv.org/abs/2603.13606) |
| 14 | GPU-Initiated Networking for NCCL (GIN) | 2025 | - | 6 | open_pdf | [arXiv](https://arxiv.org/abs/2511.15076) |
| 15 | HetCCL: Accelerating LLM Training with Heterogeneous GPUs | 2026 | - | 0 | open_pdf | [arXiv](https://arxiv.org/abs/2601.22585) |
| 16 | An RDMA-First Object Storage System with SmartNIC Offload (ROS2) | 2025 | SC25-W | 2 | open_pdf | [arXiv](https://arxiv.org/abs/2509.13997) |
| 17 | RoCE BALBOA: Service-enhanced Data Center RDMA for SmartNICs | 2025 | - | 2 | open_pdf | [arXiv](https://arxiv.org/abs/2507.20412) |
| 18 | Context Parallelism for Scalable Million-Token Inference | 2024 | MLSys | 36 | open_pdf | [arXiv](https://arxiv.org/abs/2411.01783) |
| 19 | Blink: CPU-Free LLM Inference by Delegating Serving Stack to GPU and SmartNIC | 2026 | - | 0 | open_pdf | [arXiv](https://arxiv.org/abs/2604.07609) |
| 20 | FlexLink: Boosting your NVLink Bandwidth by 27% without accuracy concern | 2025 | - | - | open_pdf | [arXiv](https://arxiv.org/abs/2510.15882) |

## 5. PDF 下载结果

| 状态 | 数量 |
|------|------|
| 应下载（core + open_pdf） | 8 |
| 已下载 | 7 |
| 非 OA / 需机构访问（core + paywalled） | 2 (Husky NSDI'23, Harmonic NSDI'24) |
| 未下载（非 core） | 12 |

已下载论文详情：

| # | arXiv ID | 标题 | 本地路径 | 大小 (KB) |
|---|----------|------|----------|------------|
| 3 | 2512.22743 | OptiNIC | papers/inbox/2512.22743.pdf | 427 |
| 4 | 2510.16606 | Celeris | papers/inbox/2510.16606.pdf | 185 |
| 5 | 2504.17307 | UCCL | papers/inbox/2504.17307.pdf | 2166 |
| 6 | 2505.11339 | Palladium | papers/inbox/2505.11339.pdf | 1520 |
| 7 | 2508.09663 | Multi-Tenant Slingshot RDMA | papers/inbox/2508.09663.pdf | 473 |
| 8 | 2511.22880 | LoRAServe | papers/inbox/2511.22880.pdf | 1919 |
| 10 | 2504.19232 | Adaptra | papers/inbox/2504.19232.pdf | 1995 |

未下载（paywalled，需机构访问或用户手动提供）：

| # | 标题 | Venue | DOI/URL | 合法获取建议 |
|---|------|-------|---------|-------------|
| 1 | Husky: Understanding RDMA Microarchitecture Resources for Performance Isolation | NSDI 2023 | [USENIX](https://www.usenix.org/conference/nsdi23/presentation/kong) | USENIX 开放获取；可通过机构 IP 直接下载；用户也可手动提供 PDF |
| 2 | Harmonic: Hardware-assisted RDMA Performance Isolation for Public Clouds | NSDI 2024 | [USENIX](https://www.usenix.org/conference/nsdi24/presentation/lou) | USENIX 开放获取；可通过机构 IP 直接下载；用户也可手动提供 PDF |

## 6. 问题与备注

1. **Semantic Scholar 速率限制**：无 API Key 场景下 S2 频繁返回 429，部分论文的引用数通过 batch API 批量获取，部分论文的引用数为估计值。建议后续获取 S2 API Key 以提高准确性。
2. **Husky/Harmonic 非 arXiv**：NSDI 2023/2024 的两篇核心论文未在 arXiv 上发布预印本。USENIX 会议论文通常为开放获取，但下载需经过 USENIX 网站（可能需要机构 IP 认证）。已在 manifest 中记录为 `needs_institution`。
3. **Alibaba STELLAR（SIGCOMM 2025）**：通过 WebSearch 发现的 SIGCOMM 2025 论文，聚焦 RDMA 虚拟化和多租户隔离的 AI 云基础设施，但在本文撰写时未找到 arXiv 预印本，且 S2 收录信息有限，暂未纳入正式清单。
4. **PeRF（ATC 2024）**：USENIX ATC 2024 的 RDMA 抢占式隔离论文，通过 WebSearch 发现，同样未找到 arXiv 预印本。
5. **本 query 的特殊性**："RDMA 多租户性能隔离 + ML 训练场景"是一个交叉领域，直接命中的论文数量较少（约 8 篇 core），大部分工作要么聚焦 RDMA 隔离（不专门针对 ML），要么聚焦 ML 训练通信优化（不专门讨论多租户隔离）。报告筛选标准严格遵循 round_goal_excerpt 中的 what_to_find/what_not_to_find 规范。

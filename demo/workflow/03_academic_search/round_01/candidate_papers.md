# 候选论文优先级排序表（Round 01）

> 论文按"近期顶会优先 > 引用数降序 > venue 等级"排序。Core 论文附摘要一句。

## Core 论文（26 篇，建议优先精读）

| # | 论文 | 年份 | Venue | 等级 | 引用 | 技术路线 | PDF | 来源 Query |
|---|------|------|-------|------|------|---------|-----|-----------|
| 1 | Harmonic: Hardware-assisted RDMA Performance Isolation | 2024 | NSDI | CCF-A | 20 | 微架构感知隔离 | ✅ OA | Q1,Q4 |
| 2 | Husky: Understanding RDMA Microarchitecture Resources | 2023 | NSDI | CCF-A | 66 | 微架构瓶颈揭示 | ✅ OA | Q1,Q4 |
| 3 | PeRF: Preemption-enabled RDMA Framework | 2024 | ATC | CCF-A | — | 软件抢占隔离 | ✅ OA | Q1 |
| 4 | Palladium: DPU-enabled Multi-Tenant Serverless | 2025 | SIGCOMM | CCF-A | 2 | DPU隔离 | ✅ OA | Q1,Q4,Q5 |
| 5 | Arnold: Topology-Aware Communication Alignment | 2025 | NeurIPS | CCF-A | — | 拓扑感知调度 | ✅ OA | Q2,Q3,Q5 |
| 6 | CrossPipe: Optimal Pipeline Schedules for Cross-DC Training | 2025 | ATC | CCF-A | — | 跨DC训练调度 | ✅ OA | Q2 |
| 7 | TicTac: Accelerating Distributed DL with Comm Scheduling | 2018 | MLSys | CCF-B | — | 优先级调度 | ✅ OA | Q2 |
| 8 | P3: Priority-based Parameter Propagation | 2019 | SysML | CCF-B | — | 优先级调度 | ✅ OA | Q2 |
| 9 | DLCP: Domain-specific Communication Optimization | 2020 | — | — | — | 优先级+负载均衡 | ✅ OA | Q2 |
| 10 | DHelix: Hiding Communication Cost via Micro-batch Co-execution | 2024 | — | — | — | 通信隐藏 | ✅ OA | Q2 |
| 11 | vClos: Isolated Scheduling for Distributed Training Tasks | 2023 | — | — | — | 拓扑+通信联合 | ✅ OA | Q3,Q5 |
| 12 | BandPilot: Contention-Aware GPU Dispatching | 2025 | — | — | — | 学习带宽模型 | ✅ OA | Q3 |
| 13 | When Scaling Fails: Network Effects on GPU Training | 2026 | — | — | — | 实证故障分析 | ✅ OA | Q3,Q5 |
| 14 | Network Contention-Aware Scheduling with RL | 2023 | — | — | — | RL调度 | ✅ OA | Q5 |
| 15 | Compute-Comm Overlap Characterization | 2025 | — | — | — | 干扰定量分析 | ✅ OA | Q3 |
| 16 | OptiNIC: Tail-Optimal RDMA NIC for Distributed ML | 2025 | — | 1 | ML感知RDMA | ✅ OA | Q1,Q4,Q5 |
| 17 | Celeris: Reimagining RDMA Through the Lens of ML | 2025 | CAL | — | ML感知RDMA | ✅ OA | Q4 |
| 18 | Extensible Software Transport Layer for GPU Networking | 2025 | — | 10 | 软件传输层 | ✅ OA | Q4 |
| 19 | Palos: Fair and Flexible Flow Scheduling on RNIC | 2024 | HPCC | — | — | 数据块调度 | ❌ 机构 | Q1 |
| 20 | OSMOSIS: Enabling Multi-Tenancy in Datacenter SmartNICs | 2023 | — | — | — | SmartNIC隔离 | ✅ OA | Q1 |
| 21 | Slingshot RDMA for Kubernetes | 2025 | IEEE Cluster | — | 1 | 平台级隔离 | ✅ OA | Q1,Q4,Q5 |
| 22 | MergeComp: Compression Scheduler for Communication-Efficient Training | 2021 | — | — | — | 压缩协同调度 | ✅ OA | Q2 |
| 23 | DynaComm: Dynamic Communication Scheduling for Edge-Cloud | 2021 | JSAC | CCF-A | — | 动态调度 | ✅ OA | Q2 |
| 24 | VCCL: Efficient Collective Communication Library | 2025 | — | — | — | 通信库替代 | ✅ OA | Q3 |
| 25 | Impact of RoCE Congestion Control on DNN Training | 2022 | — | — | — | 拥塞控制评估 | ✅ OA | Q5 |
| 26 | Ethereal: Divide and Conquer Network Load Balancing | 2024 | — | — | — | 负载均衡 | ✅ OA | Q5 |

## General 论文（22 篇，视 synthesis 需求选择性阅读）

| # | 论文 | 年份 | Venue | PDF | 来源 |
|---|------|------|-------|-----|------|
| 27 | NCCL EP: Unified Expert Parallel Communication API | 2026 | arXiv | ✅ OA | Q1,Q4 |
| 28 | UCCL-EP: Portable Expert-Parallel Communication | 2025 | arXiv | ✅ OA | Q1,Q4 |
| 29 | HetCCL: Accelerating LLM Training with Heterogeneous GPUs | 2026 | arXiv | — | Q1,Q4 |
| 30 | CSA-UD: Communication-Semantic-Aware RDMA Loss Recovery | 2026 | arXiv | ✅ OA | Q1 |
| 31 | ForestColl: Throughput-Optimal Collective Communications | 2024 | arXiv | — | Q1 |
| 32 | MonkeyTree: Near-Minimal Congestion via Migration | 2026 | arXiv | — | Q1 |
| 33 | Demystifying NCCL: In-depth Analysis of GPU Communication | 2025 | arXiv | ✅ OA | Q2 |
| 34 | EmbRace: Accelerating Sparse Communication | 2023 | arXiv | ✅ OA | Q2 |
| 35 | TACCL: Guiding Collective Algorithm Synthesis | 2023 | NSDI | — | Q3 |
| 36 | TE-CCL: ML Collective Communication as Multi-Commodity Flow | 2024 | SIGCOMM | — | Q3 |
| 37 | MSCCL++: Rethinking GPU Communication Abstractions | 2025 | arXiv | — | Q3 |
| 38 | MultiWorld: Enabling Elastic Model Serving | 2024 | arXiv | ✅ OA | Q3 |
| 39 | Flex-MIG: Enabling Distributed Execution on MIG | 2025 | arXiv | ✅ OA | Q3 |
| 40 | FiCCO: Finer-Grain Compute Communication Overlap | 2025 | arXiv | — | Q3 |
| 41 | ScaleAcross Explorer: Communication Optimization | 2026 | arXiv | ✅ OA | Q3,Q5 |
| 42 | MFS: Multi-stage Flow Scheduling for LLM Serving | 2026 | arXiv | — | Q3 |
| 43 | PCCL: Photonic Circuit-Switched Collective Communication | 2025 | arXiv | — | Q3 |
| 44 | Hopper: Congestion-Aware Path Selection for AI Clusters | 2025 | arXiv | ✅ OA | Q4,Q5 |
| 45 | Adaptra: Straggler-Resilient Hybrid-Parallel Training | 2025 | arXiv | ✅ OA | Q4,Q5 |
| 46 | Resilient AI Supercomputer Networking using MRC and SRv6 | 2026 | arXiv | — | Q5 |
| 47 | Context Parallelism for Scalable Million-Token Inference | 2024 | MLSys | — | Q4 |
| 48 | Wormhole: Packet-level Simulation Acceleration | 2026 | arXiv | — | Q5 |

## 排序规则

1. **近 6 个月**（2026 年）和 **顶会**（NSDI, SIGCOMM, ATC, NeurIPS）论文自动置顶
2. 引用数降序（有引用数据的论文）
3. venue 等级：CCF-A > CCF-B > 未收录 > 预印本
4. Core > General 优先级
5. 标注清晰 PDF 下载状态

## 精读建议

**第一优先**（直接回答"已有方案覆盖了什么/不足在哪"）：
1. Husky (NSDI'23) + Harmonic (NSDI'24) → 理解 RDMA 隔离的核心瓶颈和 SOTA
2. PeRF (ATC'24) + Palos (HPCC'24) → 了解软件和硬件调度方案的差异
3. vClos (2023) + Arnold (NeurIPS'25) → 理解训练通信与网络拓扑联合优化

**第二优先**（提供"训练感知调度"的方法论和baseline）：
4. BandPilot (2025) + RL-based Scheduling (2023) → 智能调度的可行性和效果
5. When Scaling Fails (2026) + Compute-Comm Overlap (2025) → 多任务干扰的定量证据
6. OptiNIC/Celeris (2025) → ML 感知 RDMA 新范式

**补充阅读**（视 synthesis 需求）：
7. 优先级调度系列（TicTac, P3, DLCP, DynaComm）→ 理解通信调度方法论演进
8. DPU 方案（Palladium, OSMOSIS, Slingshot）→ 了解工业级隔离实现

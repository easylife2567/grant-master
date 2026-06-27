# 第 01 轮文献查找短期目标

## 1. 本轮调研依据

本轮计划基于以下文件生成：

- `workflow/01_topic/01_topic_card.md` —— 课题初始理解卡片，提供了应用场景、核心矛盾假设和技术方向概述
- `workflow/01_topic/01_literature_seed.yaml` —— 第一轮调研种子，包含 5 个搜索方向（A1-A5）及关键词
- `workflow/01_topic/01_topic_result.yaml` —— 阶段结果，background_clarity=0.50, literature_plan_readiness=0.75

## 2. 长期计划更新摘要

本轮对 `long_plan.yaml` 的更新：

- **新增核心任务** C1（RDMA多租户性能隔离方法）、C2（分布式训练通信调度与网络优化）
- **新增技术任务** T1（RDMA拥塞控制与负载均衡公平性）、T2（智能网络资源调度方法基础）、T3（智算中心多租户网络架构实践）
- **本轮标注为跳过的任务**：
  - B1（RDMA与分布式训练网络基础概念调研）：01_topic_card.md 已充分覆盖核心技术概念和应用背景，literature_plan_readiness=0.75，无需单独背景轮次
- **本轮选择 C1 + C2**：两者构成课题核心问题的两个互补维度——"网络隔离"（C1）和"训练调度"（C2），直接服务于"了解现有方法及其不足"的首轮目标；T1-T3 作为深度扩展方向，预留后续轮次

## 3. 本轮短期 goal

**系统调研 RDMA 多租户性能隔离和分布式训练通信调度的现有方法，回答"已有方案覆盖了什么、在哪里还有不足"这个核心问题，为 synthesis 提供可精读的候选论文集合。**

## 4. 本轮任务边界

### 本轮要查

1. **C1 — RDMA 多租户性能隔离方法**：软件层面（速率限制、抢占式调度、虚拟化隔离）、硬件层面（FPGA 辅助、微架构感知）、软硬协同方案的架构、隔离粒度、性能开销和对比
2. **C2 — 分布式训练通信调度与网络优化**：通信模式感知的调度、多任务训练的网络资源分配、NCCL/集体通信库的资源管理、通信压缩与网络调度的联合优化

### 本轮不查

- RDMA 基础协议和实现细节（非多租户相关）——已在 01_topic_card 中覆盖
- 训练算法的优化（如梯度压缩算法本身、模型并行策略优化）——除非直接涉及网络调度
- 通用数据中心网络调度（非 RDMA、非训练场景）——除非方法可直接迁移
- 智能调度方法的通用综述（如 RL for networking survey）——本轮聚焦于具体系统工作
- 智算中心整体架构设计（非网络聚焦）——除非包含明确的多租户网络隔离方案
- 广域长距 RDMA 传输优化——属于 T3 范围，本轮不涉及

## 5. 本轮核心问题

1. 已有 RDMA 多租户性能隔离方案主要有哪些技术路线？各自在隔离粒度、性能开销、实现复杂度上如何对比？
2. PeRF、Harmonic、Palos 等 2024 年新工作各解决了什么问题？它们的局限性在哪里？
3. 这些方案是否考虑了分布式训练场景的通信模式特征（周期性突发、all-reduce 聚合、迭代性质）？
4. 分布式训练的通信调度主要关注哪些维度（拓扑感知、压缩协同、优先级调度）？有哪些代表性系统工作？
5. 现有训练通信调度是否有考虑多任务/多租户并发场景下的网络隔离和公平性？
6. 在"网络层隔离"（C1）和"训练层调度"（C2）之间是否存在交叉地带？是否有工作尝试将两者结合？

## 6. 本轮关键词与检索式

### C1：RDMA 多租户性能隔离

**中文关键词**：RDMA 多租户、性能隔离、RDMA NIC 共享、RoCE QoS、RDMA 虚拟化、RDMA 抢占式调度、RNIC 公平性

**英文关键词**：RDMA multi-tenant, performance isolation, RDMA NIC sharing, RoCE QoS, RDMA virtualization, RDMA preemption, RNIC fairness, RDMA rate limiting, RDMA microarchitecture isolation

**英文检索式**：
- `"RDMA" AND ("multi-tenant" OR "multi-tenancy") AND ("performance isolation" OR "resource isolation")`
- `"RDMA" AND ("NIC sharing" OR "RNIC sharing") AND ("fairness" OR "isolation" OR "scheduling")`
- `"RoCE" AND ("multi-tenant" OR "cloud") AND ("QoS" OR "performance" OR "congestion")`
- `("RDMA" OR "RoCE") AND ("network virtualization" OR "SR-IOV") AND ("tenant" OR "isolation")`

### C2：分布式训练通信调度

**中文关键词**：分布式训练 通信调度、梯度同步 网络优化、NCCL 多任务、训练通信 拓扑感知、all-reduce 调度、通信压缩 网络协同

**英文关键词**：distributed training communication scheduling, gradient synchronization optimization, NCCL multi-job, collective communication scheduling, topology-aware training communication, parameter server network scheduling

**英文检索式**：
- `"distributed training" AND ("communication scheduling" OR "communication optimization") AND ("network" OR "RDMA")`
- `("all-reduce" OR "collective communication") AND ("scheduling" OR "resource allocation") AND ("multi-job" OR "multi-tenant")`
- `"NCCL" AND ("multi-job" OR "concurrent" OR "resource sharing" OR "isolation")`
- `"distributed machine learning" AND ("network-aware scheduling" OR "communication-aware") AND ("GPU cluster" OR "training cluster")`

## 7. 候选论文筛选标准

### 优先保留
- 直接涉及 RDMA 多租户性能隔离的系统论文（如 PeRF、Harmonic、Palos 等）
- 涉及分布式训练网络通信调度和资源分配的顶会论文
- 将训练语义（通信模式、迭代性质）纳入网络层决策的工作
- 在真实 RDMA 硬件或高保真仿真环境中验证的工作

### 排除
- 仅讨论 RDMA 协议改进不涉及多租户的论文
- 纯训练算法优化（梯度压缩、量化、稀疏化）不涉及网络调度的论文
- 通用云网络调度与训练无关的论文
- 纯理论分析无系统实现的论文（除非是领域核心引用）
- 研讨会短文（<6 页）除非创新性显著

### 时间范围
- 优先 2021-2026 年论文（近 5 年）
- RDMA 多租户隔离是 2023+ 新兴热点，重点覆盖 2023-2025
- 分布式训练通信调度可回溯至 2019 左右的关键工作（如 ByteScheduler、PACE 等）

### 优先会议
- 网络/系统：**NSDI、SIGCOMM、OSDI、SOSP、ATC、EuroSys**
- 高性能计算：**SC、HPDC**（分布式训练通信方面）
- 机器学习系统：**MLSys**（训练系统与通信调度交叉）

### PDF 获取
- 优先开放获取论文（USENIX 系列通常 OA、arXiv 预印本）
- 记录 paywalled 论文的合法获取建议
- 需要用户手动下载的论文标注清楚来源和被付费墙阻挡的原因

## 8. 期望第 03 阶段输出

期望 `03_academic_search` 在 `workflow/03_academic_search/round_01/` 中生成：

- `search_results.yaml` —— 结构化搜索结果（含论文元信息、来源、下载状态）
- `candidate_papers.md` —— 候选论文列表（按 C1/C2 分组，含筛选理由和初步分级）
- `download_queue.yaml` —— 可下载论文队列
- `search_summary.md` —— 本轮搜索总结（含覆盖评估、gap、建议）

## 9. 给 academic-search 的执行说明

> **本轮 academic-search 只执行当前 round_01 的 goal，不扩展到 long_plan.yaml 中的其他任务。**
>
> 请严格按照 `workflow/02_literature_plan/round_01/search_queries.yaml` 中的查询式执行搜索，重点覆盖 C1 和 C2 两个并行任务。每个任务预期找到 5-8 篇候选论文，本轮总计 10-16 篇候选。
>
> 搜索时请注意：
> - 每条查询式可能需要在对学术搜索引擎中分别执行
> - 优先使用 OpenAlex API 或学术搜索引擎，覆盖 NSDI、SIGCOMM、ATC 等顶会
> - 对核心 OA 论文确保下载或记录 arXiv 链接
> - 对 paywalled 论文记录合法获取建议，不尝试绕过付费墙
> - 完成后按 C1/C2 分组整理候选论文
>
> **不要做**：不生成文献综述结论、不精读论文、不扩展到 long_plan 中其他任务、不修改任何 plan 文件。

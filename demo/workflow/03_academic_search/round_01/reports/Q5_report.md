# Query Q5 搜索报告

**查询式**：multi-job training communication interference network contention GPU cluster
**平台**：arXiv, Semantic Scholar（S2 速率受限）
**时间**：2026-06-27T17:45:00Z

## 1. 搜索策略

**目标**：系统调研 RDMA 多租户性能隔离和分布式训练通信调度的现有方法，聚焦多任务并发训练的通信干扰定量研究、训练任务网络争抢的实证分析和解决方案。

**搜索平台**：arXiv API（主要），Semantic Scholar（辅助，遭遇 429 速率限制，3 次重试后放弃）。

**扩展查询**：
- `all:network AND all:contention AND all:GPU AND all:training`（50 条结果）
- `all:RDMA AND all:distributed AND all:training`（32 条结果）
- `all:communication AND all:scheduling AND all:distributed AND all:training AND all:GPU`（82 条结果）
- `all:multi-tenant AND all:RDMA AND all:performance AND all:isolation`（3 条结果）

**筛选标准**：
- 年份：2021-2026（与说明书一致）
- 优先 CCF-A/B 会议/期刊（NeurIPS, ICLR, IEEE Cluster, MLSys 等）
- 排除仅讨论计算资源争抢不涉及网络通信、单机多任务 GPU 共享不涉及网络的论文
- 按时效性（近 6 个月置顶）+ 引用数/venue 等级排序

**API 问题**：Semantic Scholar 返回 429（速率超限），OpenAlex 返回预算不足。两个平台的引用数据均不可用，已根据 arXiv 元数据和已知 venue 信息完成筛选。此问题不影响论文搜索和筛选本身——仅影响引用计数字段。

## 2. 结果总览

| 指标 | 数值 |
|------|------|
| 总检索 / 筛选后 | ~164 / 15 |
| core / general | 8 / 7 |
| 近 6 个月论文 | 5 |
| OA（含 arXiv PDF） | 15 |
| 已下载 PDF | 8（core 全部下载）|

## 3. 重要论文（core）

---

### [1] When Scaling Fails: Network and Fabric Effects on Distributed GPU Training Performance（2026，cs.NI）

Dinesh Gopalan, Ratul Ali

**简介**：该文对多个生产级 GPU 集群中分布式训练的扩展失效现象进行了实证研究。作者发现网络拓扑、拥塞动态、集合通信同步行为和 GPU 局部性在跨节点训练中主导端到端性能，且这些效应往往被标准 profiling 工具遗漏，被误诊为框架或模型层面的低效。论文识别了同步放大、拓扑引发争用和局部性驱动的性能方差等反复出现的故障模式，并提供了系统构建者可用于理解扩展限制的实用诊断原则。

- 链接：[arXiv](https://arxiv.org/abs/2603.04424)
- OA：open_pdf | PDF：papers/inbox/2603.04424.pdf（647 KB，10 页）
- 下载来源：arxiv

---

### [2] Network Contention-Aware Cluster Scheduling with Reinforcement Learning（2023，cs.LG / cs.DC）

Junyeol Ryu, Jeongyoon Eo

**简介**：该文针对 GPU 集群中分布式训练任务的网络争用问题，首次将集群调度形式化为强化学习问题。提出的调度策略能够捕获不同训练任务间的争用敏感度，动态调整调度决策。实验表明相比广泛使用的调度策略，该方法将平均作业完成时间降低 18.2%，尾部作业完成时间降低 20.7%，且在平均完成时间和资源利用率之间实现了良好的折衷。

- 链接：[arXiv](https://arxiv.org/abs/2310.20209)
- OA：open_pdf | PDF：papers/inbox/2310.20209.pdf（736 KB，3 页）
- 下载来源：arxiv

---

### [3] Isolated Scheduling for Distributed Training Tasks in GPU Clusters（vClos）（2023，cs.DC）

Xinchi Han, Weihao Jiang, Peirui Cao, Qinwei Yang, Yunzhuo Liu, Shuyao Qi, Shengkai Lin, Shizhen Zhao

**简介**：该文分析了多租户 GPU 集群中由 hash 冲突引起的网络争用如何增加通信开销并造成不公平。作者提出 vClos 系统，通过联合优化网络拓扑和分布式训练通信模式来消除网络争用。此外还提出 OCS-vClos 方案，在 leaf-spine 网络中引入光电路交换机以减少资源分配策略导致的网络资源碎片化。在 32 个 V100 GPU 的测试台和基于真实 trace 的大规模模拟中验证了 vClos 的优越性。

- 链接：[arXiv](https://arxiv.org/abs/2308.05692)
- OA：open_pdf | PDF：papers/inbox/2308.05692.pdf（952 KB）
- 下载来源：arxiv

---

### [4] Impact of RoCE Congestion Control Policies on Distributed Training of DNNs（2022，cs.NI）

Tarannum Khan, Saeed Rashidi, Srinivas Sridharan, Pallavi Shurpali, Aditya Akella, Tushar Krishna

**简介**：该文全面分析了多种 SOTA RoCE 拥塞控制方案在分布式训练平台上的表现。作者发现，与通用数据中心不同，分布式训练平台中的可扩展拓扑感知集合通信算法能够天然避免 incast 模式并优化流量均衡，因此此前提出的 RoCE 拥塞控制方案对训练工作负载端到端性能影响有限。该发现为设计针对训练工作负载特征优化的轻量级拥塞控制方案提供了重要动机。

- 链接：[arXiv](https://arxiv.org/abs/2207.10898)
- OA：open_pdf | PDF：papers/inbox/2207.10898.pdf（571 KB，10 页）
- 下载来源：arxiv

---

### [5] Congestion-Aware Path Selection for Load Balancing in AI Clusters（Hopper）（2025，cs.NI）

Erfan Nosrati, Majid Ghaderi

**简介**：该文提出 Hopper，一种针对 AI 集群 RDMA 流量的主机端负载均衡技术。Hopper 持续监控当前路径的拥塞状况，检测到拥塞时动态将流量切换到更不拥塞的路径，无需专用硬件或交换机修改。在 ns-3 模拟和测试台实现上的评估表明，相比现有 SOTA 主机端负载均衡技术，Hopper 将平均和 99 分位尾部流完成时间分别降低 20% 和 14%。

- 链接：[arXiv](https://arxiv.org/abs/2506.08132)
- OA：open_pdf | PDF：papers/inbox/2506.08132.pdf（1.36 MB，11 页）
- 下载来源：arxiv

---

### [6] Efficient Pre-Training of LLMs via Topology-Aware Communication Alignment on More Than 9600 GPUs（Arnold）（2025，NeurIPS 2025 [CCF-A]）

Guoliang He, Youhe Jiang, Wencong Xiao, Kaihua Jiang, Shuguang Wang, Jun Wang, Zixian Du, Zhuo Jiang, Xinlei Zhang, Binhang Yuan, Eiko Yoneki

**简介**：该文提出 Arnold 调度系统，将 LLM 预训练的复杂通信模式与数据中心物理拓扑进行有效对齐。通过深入的特性研究识别了物理网络拓扑对 LLM 预训练作业的影响，并开发了调度算法以有效对齐通信模式与网络拓扑。模拟实验表明通信组最大跨度降低 1.67x，生产环境中 9600+ GPU 训练端到端性能提升 10.6%。该文发表于 NeurIPS 2025，是面向大规模训练网络争用问题的一项权威工作。

- 链接：[arXiv](https://arxiv.org/abs/2509.15940)
- OA：open_pdf | PDF：papers/inbox/2509.15940.pdf（1.27 MB，17 页）
- 下载来源：arxiv
- Venue：[NeurIPS 2025](https://neurips.cc) [CCF-A]

---

### [7] Palladium: A DPU-enabled Multi-Tenant Serverless Cloud over Zero-copy Multi-node RDMA Fabrics（2025，cs.NI / cs.DC）

Shixiong Qi, Songyu Zhang, K. K. Ramakrishnan, Diman Z. Tootaghaj, Hardik Soni, Puneet Sharma

**简介**：该文提出 Palladium，一个以 DPU 为中心的 serverless 数据平面，利用 RDMA 和跨处理器共享内存实现零拷贝通信。核心贡献是 DPU-enabled Network Engine（DNE），能够在多租户环境下隔离 RDMA 资源、编排跨节点 RDMA 流、并在争用下强制执行公平性。通过早期 HTTP/TCP-to-RDMA 传输转换解决了协议不匹配问题。实验表明 DPU 卸载使 RPS 提升 20.9x，延迟降低 21x。

- 链接：[arXiv](https://arxiv.org/abs/2505.11339)
- OA：open_pdf | PDF：papers/inbox/2505.11339.pdf（1.52 MB，7 页）
- 下载来源：arxiv

---

### [8] Ethereal: Divide and Conquer Network Load Balancing in Large-Scale Distributed Training（2024，cs.NI / cs.DC）

Vamsi Addanki, Prateesh Goyal, Ilias Marinos, Stefan Schmid

**简介**：该文挑战了"packet spraying 是大规模分布式训练必需的"这一普遍信念，证明了单路径传输在 CLOS 拓扑中的分布式训练工作负载下几乎可以匹配理想 packet spraying 的性能。作者提出 Ethereal，一种利用集合通信模式特性的分布式负载均衡算法——在应用层主动分割流并分配路径，对现有 RDMA NIC 几乎不需要修改。相比 packet spraying 将完成时间最多降低 30%，为下一代传输协议设计提供了替代视角。

- 链接：[arXiv](https://arxiv.org/abs/2407.00550)
- OA：open_pdf | PDF：papers/inbox/2407.00550.pdf（1.55 MB）
- 下载来源：arxiv

---

## 4. 一般论文清单

| # | 标题 | 年份 | Venue | 链接 | OA |
|---|------|------|-------|------|----|
| 9 | Resilient AI Supercomputer Networking using MRC and SRv6 | 2026 | cs.NI（OpenAI/MS 生产） | [arXiv](https://arxiv.org/abs/2605.04333) | open_pdf |
| 10 | OptiNIC: A Resilient and Tail-Optimal RDMA NIC for Distributed ML Workloads | 2025 | cs.DC / cs.NI | [arXiv](https://arxiv.org/abs/2512.22743) | open_pdf |
| 11 | Adaptra: Straggler-Resilient Hybrid-Parallel Training with Pipeline Adaptation | 2025 | cs.DC | [arXiv](https://arxiv.org/abs/2504.19232) | open_pdf |
| 12 | Supercharging Packet-level Network Simulation of Large Model Training via Memoization and Fast-Forwarding (Wormhole) | 2026 | cs.NI | [arXiv](https://arxiv.org/abs/2602.10615) | open_pdf |
| 13 | Closing the HPC-Cloud Convergence Gap: Multi-Tenant Slingshot RDMA for Kubernetes | 2025 | IEEE Cluster 2025 | [arXiv](https://arxiv.org/abs/2508.09663) | open_pdf |
| 14 | ScaleAcross Explorer: Exploring Communication Optimization for Scale-Across AI Model Training | 2026 | cs.DC（Meta 生产） | [arXiv](https://arxiv.org/abs/2605.24326) | open_pdf |
| 15 | Design Space Exploration of DMA based Finer-Grain Compute Communication Overlap (FiCCO) | 2025 | cs.DC / cs.AR | [arXiv](https://arxiv.org/abs/2512.10236) | open_pdf |

## 5. PDF 下载结果

| 状态 | 数量 |
|------|------|
| 应下载（core + open_pdf） | 8 |
| 已下载 | 8 |
| 非 OA / 需合法获取 | 0 |
| 下载失败 | 0 |

全部 15 篇论文均来自 arXiv，均为开放获取。8 篇 core 论文 PDF 已成功下载至 `papers/inbox/`，7 篇 general 论文按策略不下载。

## 6. 问题与备注

1. **S2/OpenAlex API 速率限制**：Semantic Scholar 返回 429 错误（3 次重试无改善），OpenAlex 返回预算耗尽错误。本报告中的引用数据因此不可用。论文筛选和排序未受显著影响——所有结果均来自 arXiv API，venue 信息和摘要已提供充分依据。

2. **引用数据缺失建议**：建议 coordinator 在合并阶段通过有 API Key 的 S2 调用或 Google Scholar CDP 模式补全引用数据。

3. **多租户通信干扰研究现状**：搜索结果显示，直接研究"多任务并发训练通信干扰"的工作集中在调度层面（如 vClos, Arnold）和负载均衡/拥塞控制层面（如 Hopper, Ethereal）。关于多租户 RDMA 隔离的工作（Palladium, Slingshot RDMA）则更多来自系统/网络领域。两个研究方向之间的交叉是目前学术界较少直接覆盖的空白地带。

4. **实证研究稀缺**：仅有 "When Scaling Fails"（2026 年 2 月）和 "Impact of RoCE Congestion Control"（2022 年）对训练集群中的网络争用效应进行了实证量化分析。该方向仍然开放。

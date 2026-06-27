# CrossPipe: Towards Optimal Pipeline Schedules for Cross-Datacenter Training

## 1. 论文基本信息

- **标题**：CrossPipe: Towards Optimal Pipeline Schedules for Cross-Datacenter Training
- **作者**：Tiancheng Chen, Ales Kubicek, Langwen Huang, Torsten Hoefler (ETH Zurich)
- **年份**：2025
- **发表**：USENIX ATC (CCF-A)
- **引用数**：0（本地索引值）
- **类型**：core / high priority

## 2. 研究问题与动机

**核心问题**：当 LLM 训练跨越地理分布式数据中心时，如何生成最优的 pipeline schedule 来显式建模和缓解网络延迟和带宽约束？

**背景动机**：
- LLM 训练的计算需求超过单个数据中心能力，跨数据中心训练成为必然趋势
- 跨 DC 的网络延迟（10us-100ms）和有限带宽（1.4-800 Gbps）显著影响训练效率
- 现有静态 pipeline schedules（1F1B、ZBV 等）假设通信成本可忽略，直接应用于跨 DC 场景会产生"bubble strides"——延迟沿关键路径累积放大

**关键分析发现**：
- Pipeline Parallelism 是跨 DC 最具可行性的并行策略（TP/SP/EP 通信过于频繁/体积大）
- 对于 Llama 3 405B，PP 在低带宽（4 GB/s）下比 DP 快 3.05×
- 静态 schedules 的 O(n_mb) 个跨 DC 通信延迟点导致延迟放大

## 3. 核心方法

CrossPipe 包含三个层次：

1. **Pipeline Performance Model**：显式建模 latency (alpha) + bandwidth (beta) 的通信延迟，区分 Forward (F) / DGrad (D) / WGrad (W) 三个计算子块，统一分析 PP 和 DP 通信重叠。

2. **Schedule Generation Algorithms**：
   - **Optimal Schedule（约束优化）**：将 pipeline scheduling 形式化为 constraint optimization (CO) 问题，通信操作提升为与计算同等的一类 citizen。CO 求解器（Gurobi/CPLEX）生成最优 schedule。CO 比 MILP 在大规模上更可扩展（32-64 stages 可行）。
   - **Greedy Schedule**：针对动态系统参数或极大规模，将每个计算块进一步切分为 n_sub 个子块。贪心调度包含三步：stage selection → operation selection → operation scheduling。整体复杂度 O(n_mb^2 * n_sub^2 * n_PP * log(...))。CrossUDSub 与求解器方案在多数延迟条件下性能相当。

3. **Flexible Execution Engine**：两层抽象——block scheduling 与 communication arrangement 解耦。提前 Post Recv 操作以最大化 overlap；四条 GPU stream 分别处理四个通信方向；支持训练过程中的 hot-switching schedules。

## 4. 与课题相关性

**高度相关**。CrossPipe 直接贡献了"拓扑感知调度"和"训练语义纳入调度"两个关键维度：

- **调度维度**：跨数据中心 pipeline schedule optimization，综合考虑 latency、bandwidth、memory budget
- **调度依据**：网络拓扑特性（latency/bandwidth ratios）+ 计算特性（F/D/W blocks）+ 内存约束
- **调度目标**：minimize makespan（而非仅优化通信/计算 overlap）

**多任务隔离**：未涉及。CrossPipe 专注于单训练作业的跨 DC 调度，假设独占链路资源。但论文中 bandwidth occupancy model 的设计为将来多作业协调提供了基础。

## 5. 关键结论与实验

**Alps 超算实验（GH200 Grace Hopper Superchips，M8/M70 模型）**：
- 跨 DC PP 通信延迟下，CrossPipe 相比静态 schedules 减少训练时间最高 33.6%
- T_bw/T_F = 2 时差距最大（M70 模型）
- CrossWave 在低延迟下最优，CrossUD(Sub) 在高延迟下最优
- 增加 GBS 和 memory budget 可进一步减少 bubble（CrossWave 在高延迟下可达无延迟情况效率）

**仿真分析**：
- Wave schedules 低延迟下效率高，UD schedules 高延迟下更优
- Loop schedules 对延迟最敏感（每个 microbatch 6 个跨 DC 通信）
- 对于 Llama 3 405B，跨 DC PP 比跨 DC DP 在 4 GB/s 时快 3.05×
- 带宽延迟比延迟延迟更难通过增加 GBS/内存来缓解

**4-DC 扩展**：CrossPipe 在 4 DC 下仍保持优势。增加 GBS 和内存后，CrossUD 仅比同配置 2 DC 慢 22.8%。

## 6. 局限与Gap

**论文自身指出的局限**：
- 求解器方案在极大规模（64+ stages）下仍需数小时计算时间
- 仅考虑 homogeneous DC（同类 GPU），heterogeneous DC 需要 stage-specific 参数调整
- 未与 compression 技术结合（如 gradient compression 可进一步减少跨 DC 流量）
- 节点故障需要更高级别的容错机制（checkpointing），非 pipeline scheduling 层面可解决
- 网络动态变化（秒级/分钟级）需 conservative estimation 或 hot-switching
- 仅覆盖 PP + DP 的跨 DC 通信，未扩展到 TP/SP/EP 的跨 DC 场景

**潜在 gap**：
- 多训练作业共享跨 DC 链路时的带宽分配未建模
- bandwidth occupancy model 较简单（range-based），未考虑真实网络中的 congestion control 交互
- 在公共云环境（高延迟、低带宽）下的实际部署验证不足

## 7. 与其他论文关联

- **与 TicTac/P3/DLCP 的关系**：CrossPipe 聚焦于 pipeline parallelism（模型并行）的通信调度，而 TicTac/P3/DLCP 聚焦于 data parallelism（Parameter Server/All-Reduce）的通信调度。两者是不同的并行维度，互补。
- **与 DHelix 的关系**：DHelix 也是 pipeline parallelism 的通信优化（micro-batch co-execution），CrossPipe 的静态 schedule 分析直接引用了 1F1B/ZBV 等 pipeline schedule，DHelix 则通过双股 interleaving 进一步优化。
- **与 DynaComm 的关系**：DynaComm 也在 layer 级别优化通信/计算重叠，但面向 edge-cloud 的 CNN PS 架构；CrossPipe 面向跨 DC 的 LLM PP 架构。调度问题的结构不同但目标一致。

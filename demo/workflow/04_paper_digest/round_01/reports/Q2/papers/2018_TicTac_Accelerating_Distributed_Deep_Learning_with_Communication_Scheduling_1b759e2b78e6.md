# TicTac: Accelerating Distributed Deep Learning with Communication Scheduling

## 1. 论文基本信息

- **标题**：TicTac: Accelerating Distributed Deep Learning with Communication Scheduling
- **作者**：Sayed Hadi Hashemi, Sangeetha Abdu Jyothi, Roy H. Campbell (UIUC)
- **年份**：2018
- **发表**：MLSys (CCF-B)
- **引用数**：0（本地索引值）
- **类型**：core / high priority

## 2. 研究问题与动机

**核心问题**：Parameter Server (PS) 架构下，DAG 表示的深度学习计算图中，参数传输顺序是随机的，导致不同 worker 之间的迭代时间方差大、straggler 效应严重。

**动机发现**：对 ResNet-V2-50 跑 1000 次迭代，每次迭代收到的参数顺序都不同（1000 次中 1000 种唯一顺序）；VGG-16 的 1000 次中有 493 种顺序。这种随机性导致两个后果：
1. 次优的计算-通信 overlap，降低吞吐
2. 同一迭代中不同 worker 的参数到达顺序不同，导致 straggler

**定位**：早期的 layer-by-layer 系统（如 Poseidon）只能处理顺序模型。TicTac 针对现代 DAG 表示的框架（TensorFlow/PyTorch），在 PS 架构下实现通信调度。

## 3. 核心方法

TicTac 通过 critical path analysis on DAG 生成近最优的参数传输调度，包含两个 heuristic：

1. **TIC（Timing-Independent Communication scheduling）**：仅基于 DAG 拓扑结构，不考虑运算时间。使用通用 time oracle（computation=0, communication=1），通过计算 impending communication load (M+) 分配优先级。

2. **TAC（Timing-Aware Communication scheduling）**：同时利用 DAG 依赖信息和 time oracle 估计的各 op 运行时间。设计了一个 Comparator 比较两个 recv op 的相对优先级，综合考虑直接计算负载 (P) 和通信时间 (M)。

**关键概念**：
- 将调度问题形式化为最小 makespan 的 job shop scheduling（NP-Hard）
- 定义了 Scheduling Efficiency 指标：E ∈ [0,1]，1 为完美调度
- 执行端实现：在 sender 端（PS）的 gRPC 传输队列前插入优先级排序（仅 40 LOC C++）

**系统组件**：Tracing Module → Time Oracle → Ordering Wizard → Enforcement Module

## 4. 与课题相关性

课题核心问题："分布式训练的通信调度主要关注哪些维度？现有训练通信调度是否考虑多任务并发的隔离和公平性？"

**高度相关**。TicTac 是分布式训练通信调度的奠基性工作，贡献了以下维度：
- **调度维度**：DAG 关键路径上的参数传输顺序（operation-level priority）
- **调度目标**：最小化单 worker 的迭代时间，通过消除 straggler 效应间接提升多 worker 协同效率
- **多任务隔离**：未涉及。TicTac 假设单训练作业独占资源，只优化单作业内部的通信顺序

TicTac 为后续工作（P3、ByteScheduler、DLCP）奠定了"利用训练语义指导通信调度"的范式基础。

## 5. 关键结论与实验

**实验结果**：
- 吞吐提升：训练最高 19.2%，推理最高 37.7%
- Straggler 效应降低：最高 2.3 倍
- Scheduling efficiency 接近 1（TAC 调度下），即近最优
- TIC 与 TAC 性能接近（当前模型），表明仅 DAG 结构信息已接近最优
- 小模型/小规模场景下偶见 4.2% 的性能退化（调度开销超过收益）

**实验环境**：Azure GPU 云 (NC6, K80) 和高端 CPU 集群，覆盖 10 个 DNN 模型、2-16 workers、1-4 PS

**关键发现**：
- 参数传输顺序的随机性是当前系统的"无声性能杀手"
- 固定任意一种顺序即可消除大部分 straggler 效应
- 调度收益随 worker 数增加先升后降（通信占比太高时 overlap 收益递减）

## 6. 局限与Gap

**论文自身指出的局限**：
- 仅适用于 Parameter Server 架构，不适用于 All-Reduce/decentralized 聚合（如 Horovod/NCCL）
- gRPC 层会偶尔打破调度顺序（Inception 模型中 TIC 误差 0.5%，TAC 误差 0.4%）
- TAC 依赖 time oracle 的时间估计，跨平台/跨迭代运行时可能不准
- TIC 与 TAC 的差距在当前模型中很小，但在未来更复杂模型中可能增大

**潜在 gap**（从论文可推导）：
- 仅考虑单作业内通信调度，未涉及多作业共享网络资源的场景
- 调度决策是离线的（offline），不随网络拥塞状态动态调整
- 未考虑网络层的拥塞信号——time oracle 是纯端侧测量
- 仅优化 recv 顺序，未涉及 send 端的选择性传输或压缩

## 7. 与其他论文关联

- **与 P3 的关系**：P3 是 TicTac 的平行工作，都做优先级调度，但 P3 是在 layer 粒度做 priority + slicing，TicTac 在 operation 粒度做 DAG critical path 分析。两者互补。
- **与 DLCP 的关系**：DLCP 将优先级从 end-host 推到 network switch 级别（packet-level），TicTac 仍在应用层做 flow-level 调度。DLCP 的引用中明确提到 TicTac。
- **与 CrossPipe 的关系**：CrossPipe 是在 pipeline parallelism 维度做 schedule optimization，TicTac 在 PS 的 recv-op 维度。两者面对的是不同并行策略下的通信调度问题。

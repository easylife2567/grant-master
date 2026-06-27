# DynaComm: Accelerating Distributed CNN Training between Edges and Clouds through Dynamic Communication Scheduling

## 1. 论文基本信息

- **标题**：DynaComm: Accelerating Distributed CNN Training between Edges and Clouds through Dynamic Communication Scheduling
- **作者**：Shangming Cai, Dongsheng Wang, Haixia Wang, Yongqiang Lyu, Guangquan Xu, Xi Zheng, Athanasios V. Vasilakos
- **年份**：2021
- **发表**：IEEE JSAC (CCF-A)
- **引用数**：0（本地索引值）
- **类型**：core / medium priority

## 2. 研究问题与动机

**核心问题**：在边缘-云协同训练的 Parameter Server 架构下，如何实现最优的逐层通信调度，将参数/梯度传输分解为最优的段（segment）以实现最大化的通信-计算重叠？

**动机**：
- Layer-by-layer 传输策略（Poseidon）虽然能重叠通信与计算，但每次独立传输 mini-procedure 引入固定开销 Δt（函数调用 + 节点间协调），在慢速边缘网络上尤其显著
- iBatch 的贪心策略只保证局部最优，在某些情况下甚至比 vanilla layer-by-layer 还差
- 需要一种既能适应网络条件又能保证全局最优的动态调度方案

## 3. 核心方法

DynaComm 将通信调度形式化为 Zero-One Integer Programming 问题，用 DP 求解最优分解决策。

**问题形式化**：
- 对于 L 层网络，存在 L-1 个可选分解位置
- 每个分解位置决定是否在此处拆分传输（启用则引入 Δt overhead）
- 目标：选择分解位置以最小化总迭代时间

**两个 DP 算法**：

1. **Forward Scheduling（参数传输）**：
   - 状态 F[m][n]：前 m 层有 n 个启用分解位置的最小 cost
   - Bellman 方程：F[m][n] = min_{k} {max(F[k][n-1], n*Δt + Σpt) + Σfc}
   - 时间复杂度 O(L^3)，空间 O(L^2)

2. **Backward Scheduling（梯度传输）**：
   - 类似结构，状态 B[m][n] 表示最后 m 层
   - 保证最优子结构性质

**实际考量**：
- 利用 MXNet 内置 profiler 获取实时 computation/communication cost vectors
- 每 epoch 调度一次以最小化调度 overhead
- 利用 idle-event-trigger 在计算空闲时提前执行调度
- 调度算法延迟 < Δt + g_t（梯度传输时延），可完全隐藏在传输中

## 4. 与课题相关性

**直接相关**。DynaComm 在通信调度的形式化和求解方法上对本课题有重要启示：

- **调度维度**：layer 级别的分段决策（decomposition positions），决定通信与计算如何交织
- **调度方法**：将通信调度形式化为组合优化问题并用 DP 求解——这是一个可推广到其他调度维度的框架
- **与 P3/TicTac 的关系**：DynaComm 的 forward scheduling 是 P3 优先级调度的更一般化形式——DynaComm 不预设固定的优先级规则，而是通过 DP 搜索最优的传输序列分解

**多任务隔离**：未涉及。

## 5. 关键结论与实验

**实验环境**：8 台 edge 设备 + 4 PS（RTT ~10ms），MXNet 实现，VGG-19/GoogLeNet/Inception-v4/ResNet-152，ILSVRC12/CIFAR-10 数据集。

**主要结果**：
- DynaComm 在所有模型和 batch size 下均达到最优调度（相比 Sequential/LBL/iBatch）
- 迭代时间减少：VGG-19 最高 41.10%，ResNet-152 最高 41.92%
- 8-worker scaling: 7.2× speedup（vs iBatch 6.2×，LBL 5.4×）
- iBatch 在 Inception-v4 上比 LBL 还差（24.22% vs 35.25% reduction）——证明贪心策略不可靠
- 模型准确率不受影响（验证了 training/validation accuracy 一致）
- 调度 overhead 极小（11.59ms for ResNet-152 forward scheduling），远小于通信时间

**敏感性分析**：
- batch size 增加到一定程度后收益收敛（计算占比成为瓶颈）
- 带宽从 1Gbps 到 5Gbps 收益最大（通信/计算比最平衡）
- 通信调度仅在通信和计算均非严重瓶颈时有效

## 6. 局限与Gap

**论文自身指出的局限**：
- 仅适用于 layered models（CNN/MLP），不适用于 RNN 等有循环依赖的模型
- 性能收益上界受限于 computation/communication ratio——当某一方严重瓶颈时收益有限
- 调度算法的 O(L^3) 复杂度在极深网络（>1000 层）下可能成为瓶颈
- 未与大 batch size 下的 gradient accumulation 等技术联合优化
- Δt 作为常量的假设在高度动态的网络中可能不准确

**潜在 gap**：
- DP 算法的状态定义假设所有 worker 同步执行，在异步训练（ASGD）下不适用
- 边缘网络的 Δt 在实际中可能波动很大（WiFi/移动网络），需要自适应 Δt 估计
- 未考虑多作业共享 edge-cloud 带宽时的调度冲突

## 7. 与其他论文关联

- **与 P3 的关系**：P3 固定优先级 + 固定大小 slicing，DynaComm 通过 DP 搜索最优分解位置——DynaComm 提供了比 P3 更一般化、更优的解决方案，但 P3 的 slicing 概念与 DynaComm 的 segmentation 概念相通。
- **与 TicTac 的关系**：TicTac 在 PS 的 DAG recv-op 级别做优先级，DynaComm 在 layer-level 做分段决策。TicTac 的 TAC 将调度视为 job shop 问题，DynaComm 将其作为 ZOIP + DP——形式化方法不同但目标一致。
- **与 CrossPipe 的关系**：CrossPipe 的 CO/贪心 schedule generation 与 DynaComm 的 DP 求解共享"将通信调度形式化为组合优化"的方法论。

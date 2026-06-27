# PeRF: Preemption-enabled RDMA Framework

## 1. 论文基本信息

- **标题**：PeRF: Preemption-enabled RDMA Framework
- **作者**：Sugi Lee, Mingyu Choi (Acryl Inc.); Ikjun Yeom (Acryl Inc. / Sungkyunkwan Univ.); Younghoon Kim (Sungkyunkwan Univ.)
- **发表**：USENIX ATC 2024（CCF-A）
- **引用数**：0（新发表）
- **与本课题相关性**：core
- **来源 query**：Q1

## 2. 研究问题与动机

### 核心问题
现有软件 RDMA 性能隔离方案（Freeflow、Justitia）采用**预留式（reservation-based）非工作保持（non-work-conserving）** 的资源分配，需要精确估计可用网络资源。低估导致 RNIC 空闲→吞吐损失；高估导致隔离被破坏。目标：在不牺牲裸金属性能的前提下实现灵活、工作保持的性能隔离。

### 动机
- 硬件方案（SR-IOV/VL）提供严格隔离但无法适应动态需求变化
- 软件方案更灵活但不精确的资源估计导致严重性能退化（Justitia 相比硬件方案吞吐损失 ~50%）
- RNIC 的两个关键微行为：(1) 对多个 QP 采用**轮询（round-robin）调度**（无论消息大小差异化）；(2) 对所有消息以**MTU 粒度均匀调度**包——大消息垄断 RNIC 处理时间，小消息被严重饿死

### 三类应用的性能异常
- **D_App**（延迟敏感，小消息）：被大消息阻塞→尾延迟飙升
- **M_App**（消息密集型，批量小消息）：消息速率被 B_App 严重降低
- **B_App**（带宽密集型，大消息/多 QP）：垄断 RNIC，单 QP B_App 被多 QP B_App 不公平对待

## 3. 核心方法

PeRF 的核心创新是**基于抢占（preemption）的工作保持调度**。

### 3.1 RNIC 抢占机制
利用 RDMA 用户级 API 中的实验性动词实现：
- **Managed QP** + **ENABLE WR**：Managed QP 中的 WR 被缓存而非直接发送，需 ENABLE WR 触发 RNIC 获取
- **WAIT WR**：插入 QP 后暂停 RNIC 处理该 QP 的后续 WR，直到指定数量的 CQE 被 poll
- **0_WAIT WR**（wait_cqe_num=0）：立即完成并暂停当前 QP，RNIC 切换到其他 QP

### 3.2 三大调度引擎

**大消息调度引擎（LMSE）——消息级隔离**
- 将大消息 WR 分割为多个子 WR（SUB_MSG_SIZE=16KB）
- 子 WR 之间插入 0_WAIT WR（0_WAIT_UNIT=1KB → 16:1 比例）
- 为小消息创造传输机会，防止大消息垄断

**多 QP 调度引擎（MQSE）——QP 级隔离**
- 创建 Preemption Control QP（PCQ）+ Preemption Control CQ（PCC）
- PAUSE：在非活跃 QP 中插入 WAIT WR，仅保留 QNum_allow 个活跃 QP
- RESUME：通过 ENABLE WR 重新激活被暂停的 QP
- 管理多 QP 应用，防止其不公平占用更多传输机会

**提前完成引擎（ECE）**
- PeRF 的 transparent WR（sub-WR, WAIT, ENABLE）可能填满应用的 SQ
- ECE 负责轮询这些透明 WR 的 CQE，释放 SQ 空间

### 3.3 选择性隔离策略
- D_App 和 M_App_single **完全绕过**隔离流程（无开销）
- B_App 所有 WR 都走 offloading 流程
- M_App_multi：部分限制（仅管理 QP 数量）
- 混合消息大小的应用：仅大消息走隔离

### 3.4 分类器
基于平均消息大小（≥1KB→B_App）和 SQ 长度（5ms 内最高 SQ_Len >5→M_App）自动识别应用类型。

### 实现
~400 行 C++，~3000 行 C（修改 libmlx5），基于 OFED 4.9。不修改应用代码。

## 4. 与课题相关性

### 与本轮精读目标的直接关联
PeRF 提供了一个**纯软件、零硬件依赖**的 RDMA 性能隔离方案，工作保持特性使其吞吐接近硬件方案。它直接回答了：
- "已有 RDMA 多租户性能隔离方案主要有哪些技术路线"：在 Harmonic（硬件/FPGA）之外，PeRF 代表了**纯软件抢占式**路线
- 与 Harmonic 的对比：Harmonic 需要 FPGA PCIe 交换机（硬件成本），PeRF 只用标准 RDMA API（零硬件依赖）

### 对假设的验证
- H1（"多租户RDMA网络共享下存在不可忽视的相互干扰"）：PeRF 用微观分析确认了干扰的具体来源——RNIC 的公平轮询 QP 调度 + 均匀包调度无视消息大小/类型差异
- H2（"现有方案未充分考虑训练场景的通信模式特征"）：PeRF **同样不感知分布式训练**。其应用分类（B_App/M_App/D_App）基于消息大小和 QP 数量，不基于通信模式（allreduce vs PS vs all-to-all）

### 对本课题的技术启示
- PeRF 的抢占机制本质上是一种**时间分片（time-slicing）**——与分布式训练的迭代周期（计算→通信→计算）有天然衔接可能
- 大消息分割（SUB_MSG_SIZE=16KB + 0_WAIT interleave）的粒度可以针对不同训练通信模式（如梯度同步阶段）进行定制
- 分类器（消息大小+SQ长度）可扩展为感知训练阶段（梯度同步=大消息 burst，参数拉取=小消息密集）
- PeRF 的 QP 级隔离（QNum_allow 配置）可用于为训练任务保留最小 QP 数

## 5. 实验与评估

### 实验环境
- 5 台服务器集群，Intel i7-11700K 16核，32GB DRAM
- Mellanox 100Gbps 交换机（SN-2100）
- 默认 RNIC：NVIDIA ConnectX-6 100Gbps RoCEv2（另测试 CX-5 40GbE、CX-6 InfiniBand）
- 测试应用类型：B_App（1MB连续WRITE）、M_App（16B批量×32）、D_App（16B稀疏）

### 主要结果

**5.2.1 开销**
- CPU：消息级隔离 0.94%、QP级 0.56%（B_App_multi）~3.62%（M_App_multi），均 <5%
- RAM：9MB（App_single），+500KB/每额外 QP
- 性能开销：与硬件方案（Strict Policy / ETS）相当——PeRF 吞吐和延迟接近专用硬件优先级队列

**5.2.2 消息级隔离**（vs Justitia）
- B_App vs D_App：PeRF 维持 B_App 高吞吐（~94 Gbps）且 D_App 延迟极低（avg <2us），Justitia 将 B_App 吞吐降低 ~50%
- B_App vs M_App：PeRF 同时保持高吞吐 + 高消息速率，Justitia 两者均大幅下降
- 关键原因：Justitia 通过暂停 WR posting 限速→RNIC 空闲→非工作保持。PeRF 的抢占允许 RNIC 持续工作

**5.2.3 QP 级隔离**
- B_App_single vs B_App_multi（1-20 QPs）：PeRF 维持双方吞吐在 ~44 Gbps 均衡，Justitia 随 QP 增加仍保持不公
- M_App_multi vs D_App_single：PeRF 将 99th 尾延迟降低 22%，M_App_single 消息率提升 35%
- B_App_single vs M_App_multi：PeRF 维持 B_App 88 Gbps 吞吐，M_App_multi 消息率可控（通过调整 QNum_allow 1/2/4/8）

**5.2.4 拥塞网络兼容性**
- PeRF 与 DCQCN 无缝协作，网络拥塞时维持公平带宽分配 + D_App 低延迟

**5.2.5 可扩展性与通用性**
- 扩展到 1000 个 B_App（每个 8 QPs）+ 1 个 M_App：消息率保持 9.7 Mmps，B_Apps 吞吐稳定 87 Gbps 且方差极低
- 支持 SEND/READ 操作：效果与 WRITE 类似
- 支持加权策略（token bucket）

**5.3 真实应用**
- Apache Crail（分布式存储/B_App）+ HERD（KVS/M_App）+ rping（D_App）同时运行
- PeRF 比 Justitia：Crail 吞吐 +55%、HERD 消息率 +14%、rping 延迟 <1us 劣化
- 可调 0_WAIT_UNIT 参数在延迟和吞吐间权衡（PeRFS 变体延迟低于 JustitiaR）

## 6. 局限与 Gap

### 论文明确指出的局限
1. **不支持 UD 连接**：WAIT/ENABLE 动词在 Unreliable Datagram 上不可用。需开发新的隔离机制（论文提出可用 0_WR 替代 0_WAIT 做消息级隔离但未实现）
2. **不支持事件驱动消息传输**：论文认为实现较简单但将其列为 future work
3. **不支持 ATOMIC 操作**：论文提到计划应用 0_WAIT WR 隔离 ATOMIC 但未实现
4. **0_WAIT_UNIT 和 SUB_MSG_SIZE 需手动配置**：没有自适应机制，云厂商需根据策略手动调整参数权衡

### 论文未涉及但对本课题重要的 gap
1. **完全不感知分布式训练通信模式**：与 Harmonic 类似，PeRF 的应用分类（D/M/B）基于消息特征而非通信语义。训练中的 AllReduce（同步、bursty）和参数更新（异步、分散）无法区分
2. **抢占粒度固定**：SUB_MSG_SIZE=16KB 和 0_WAIT 插空比例对所有 B_App 相同，无法按训练任务的不同通信阶段动态调整
3. **跨主机协调缺失**：隔离作用域仅限单个 RNIC，不协调跨主机的训练任务通信调度
4. **未评估训练场景**：实验应用（Crail/HERD/rping）和数据中心存储/KVS，无分布式训练评估
5. **抢占机制的 CPU 开销累积**：扩展至 1000 QPs 虽 OK，但 M_App_multi 情况 CPU 达 3.62% 单核——大量训练任务的多 QP 场景可能放大开销
6. **分类器错误风险**：应用类型基于启发式规则（1KB 阈值、5 个未完成 WR），异常通信模式（如梯度压缩后的变长消息）可能误分类
7. **软件层的安全风险**：作为用户级库实现，恶意租户可绕过 PeRF 库代码直接使用原始 libmlx5——论文未讨论安全保障

## 7. 关键引用与延伸阅读

### 同类软件隔离方案
- **Justitia [40] (NSDI 2022)**：基于令牌桶的软件 RDMA 多租户隔离，PeRF 的主要比较对象
- **Freeflow [22] (NSDI 2019)**：基于软件的容器化 RDMA 网络虚拟化，使用静态令牌桶

### 硬件隔离方案
- **MasQ [19] (SIGCOMM 2020)**：SR-IOV 混合虚拟化的 RDMA 私有云方案
- **LITE [35] (SOSP 2017)**：Linux 内核间接层 + VLs

### 微架构理解
- **Husky [23] (NSDI 2023)**：PeRF 引用了 ATOMIC 异常但未实现隔离
- **Collie [24] (NSDI 2022)**：RDMA 性能异常自动检测

### 对本课题的延伸
- PeRF 的 0_WAIT 抢占与 Harmonic 的 CNP 注入形成互补：前者工作保持（无吞吐损失），后者更精确（资源感知）。**组合方案**可能同时实现微架构资源精度 + 工作保持效率
- 训练场景可利用 PeRF 的抢占比例（0_WAIT_UNIT）动态配置：梯度同步阶段提高大消息份额，参数聚合阶段提高小消息份额

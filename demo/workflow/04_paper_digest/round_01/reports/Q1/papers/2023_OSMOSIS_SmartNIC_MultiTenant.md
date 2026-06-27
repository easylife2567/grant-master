# OSMOSIS: Enabling Multi-Tenancy in Datacenter SmartNICs

## 1. 论文基本信息

- **标题**：OSMOSIS: Enabling Multi-Tenancy in Datacenter SmartNICs
- **作者**：Mikhail Khalilov et al. (ETH Zurich, SPCL/IIS)
- **发表**：USENIX ATC 2024（CCF-A），arXiv:2309.03628v3
- **引用数**：0（新发表）
- **与本课题相关性**：core
- **来源 query**：Q1

## 2. 研究问题与动机

### 核心问题
**On-path SmartNIC 的多租户资源公平复用**。传统 NIC 的数据路径行为可预测（in=out），但 SmartNIC 的 kernel 执行时间不可预测（compute-bound vs IO-bound），使传统的基于 link bandwidth 的隔离机制（SR-IOV/VF 带宽分配）失效。现有方案缺乏对 sNIC 三种资源的统一公平调度：**计算（PUs）、DMA 带宽、出口带宽**。

### 动机
- SmartNIC 已从简单包转发发展为执行复杂用户自定义 kernel（如 Allreduce 梯度聚合、KVS 缓存、存储 offload）
- 400Gbit/s 下每包时间预算（PPB）极短：<150 cycles @ 1GHz with 64B 包作为 compute-bound 负载
- 上下文切换开销不可接受：Linux ~28K cycles，RTOS ~121 cycles——均远超 PPB
- 具体干扰案例：
  - PU 争用：compute-heavy 租户（2x 计算开销）使用 round-robin 调度占用 2x 的 PU（R1）→ 不公平
  - DMA/出口争用：大 IO 请求导致小请求 HoL-blocking，延迟放大 4-15x（R2）→ 严重不公平
  - 内存垄断（R3）、调度开销（R4）、控制路径优先级（R5）、QoS API 缺失（R6）

## 3. 核心方法

### 3.1 OSMOSIS 架构
- **分层设计**：灵活的软件控制平面（host CPU 上） + 性能关键的硬件数据平面（sNIC SoC 内）
- **Flow Management Queue (FMQ)**：硬件抽象，每个 ECTX（执行上下文）对应一个 FMQ，类似进程中的硬件线程。存储：匹配规则、kernel 二进制指针、SLO 策略、BVT 计数值

### 3.2 WLBVT 调度器（Weight-Limited Borrowed Virtual Time）
- 核心 PU 调度算法：组合 WFQ（权重）+ BVT（借出虚拟时间）
- 关键特性：**不依赖预先知道每包的计算成本**（invariant to per-packet cost）——这是相比 round-robin 的核心优势
- 每个 FMQ 维护 BVT 计数器+优先级，调度器选取当前活跃 FMQ 中优先级归一化吞吐最低且未超过权重上限的
- 硬件实现：128 FMQs 在 GF 22nm 仅占 1% 的 4 簇 PsPIN 面积

### 3.3 IO 路径公平：WRR + 传输分片
- **DMA 引擎**和**出口引擎**采用 WRR 调度
- **传输分片**（fragmentation）：大 DMA 请求被拆分为小事务（512B/64B 片段），消除 HoL-blocking
- 支持软件分片（kernel 内 wrapped）和硬件分片（DMA 引擎内）

### 3.4 静态内存分配 + PMP 保护
- sNIC 内存（L1/L2）在 ECTX 创建时静态分配
- 使用 RISC-V Physical Memory Protection (PMP) 单元实现内存隔离
- 避免虚拟内存的页表开销（每内存访问至少额外 1 cycle）

### 3.5 Run-to-completion
- 每个 kernel 执行处理一个包后完成，避免上下文切换
- 超时 watchdog 机制：超时 kernel 被硬件中断终止，通知 host

### 实现
- PsPIN open-source on-path sNIC（RISC-V PU 簇，1GHz，400Gbit/s）
- 控制平面：335 LOC（C）；数据平面 HW：1216 LOC（C++ 仿真）+ SystemVerilog 综合 IP

## 4. 与课题相关性

### 与本轮精读目标的关联
OSMOSIS 从 SmartNIC 硬件层面系统性地证明了：当 on-path sNIC 执行**不可预测复杂度的用户 kernel** 时，仅靠 link bandwidth 的隔离（SR-IOV/VF）完全不够。它补充了 Husky/Harmonic 的视角——后者关注传统 RNIC 微架构资源隔离，而 OSMOSIS 面向更激进的 **on-path sNIC 可编程处理**场景。

### 与本课题假设的关系
- H1（"多租户RDMA网络共享下存在不可忽视的相互干扰"）：OSMOSIS 通过 6 个要求（R1-R6）系统性地记录了 sNIC 上的干扰形式，特别是 Allreduce 梯度聚合与 KVS 缓存的资源争用
- H2（"现有方案未充分考虑训练场景的通信模式特征"）：OSMOSIS 的设计原则（cost-per-packet invariant 调度）对训练场景具有**概念上的适配性**——感知不同集合操作（Allreduce=compute-bound; data loading=IO-bound）的计算成本差异

### 技术启示
- WLBVT 的"运行时发现 cost-per-byte"思路可启发训练通信调度：不需要预先知道 AllReduce 的梯度大小，只需运行时追踪每个任务的 PU 占用时间
- OSMOSIS 的 FMQ 抽象可直接用于隔离训练任务的不同通信流（如 AllReduce 流 vs 参数服务器流）
- 传输分片机制可应用于训练中大小梯度混合传输场景
- 限制：OSMOSIS 针对 on-path sNIC，而 Harmonic/PeRF 针对传统 RNIC；两者部署假设不同

## 5. 实验与评估

### 实验设置
- 周期精确仿真（Verilator），4 个 PsPIN 簇（32 核 @1GHz），400 Gbit/s 链路
- GF 22nm ASIC 面积综合（Synopsys Design Compiler）

### 主要结果

**6.1 硬件扩展性**
- 4 簇 PsPIN 对 512B Reduce 提供足够 PPB
- WLBVT scheduler：128 FMQs 仅占 4 簇+4MiB L2 面积的约 1%
- WRR scheduler 线性缩放

**6.3 合成基准**
- PU 公平分配（R1/R5）：WLBVT 在 compute-heavy 攻击者存在时维持完美公平（Jain's ~1.0），RR 仅约 0.5
- HoL-blocking 消除（R2/R5）：传输分片将受害小请求完成时间降低一个数量级，吞吐仅降约 2x

**6.4 数据中心工作负载**
- 独立模式开销：compute-bound 负载（Aggregate/Reduce/Histogram）不超过 ±3%；IO-bound 负载因分片开销最大达 23%（可达 332 Mpps）
- Compute 混合负载：WLBVT 平均 fairness 0.946 vs RR 0.643，受害流 FCT 降低 39%
- IO 混合负载：WLBVT 平均 fairness 0.903 vs RR 0.493，所有租户 FCT 降低最高 63%
- IO Congestor 中位数每包延迟增加高达 8x（公平代价），但整体 FCT 因并行化改善

## 6. 局限与 Gap

### 论文明确指出的局限
1. **仅实现了仿真，无真实硬件评估**：使用 Verilator 周期精确仿真而非 FPGA/ASIC 原型
2. **分片引入吞吐损失**：大 IO 传输分片增加协议握手开销（N 次传输→N 次额外握手），可通过扩展 AXI 协议解决但当前未实现
3. **不感知网络端拥塞**：假定部署在无损网络（InfiniBand/RoCEv2+PFC），FMQ 内的排队延迟不触发网络拥塞反应
4. **内存分配是静态的**：不支持虚拟内存或动态分配
5. **仅支持 run-to-completion**：长计算任务不适合

### 论文未涉及但对本课题重要的 gap
1. **面向 SmartNIC on-path 场景，非传统 RNIC**：OSMOSIS 的 deployment 需要可编程的 on-path sNIC（如 PsPIN、BlueField-3 DPA），与 Harmonic/PeRF 的目标硬件不同
2. **不感知分布式训练的同步通信模式**：虽然 Allreduce 被用作 compute-bound 示例，但调度不区分 AllReduce 和其他 compute 任务
3. **无跨节点协调**：FMQ 调度仅限单 sNIC 内的 PU/DMA/Egress 公平分配，不感知跨节点训练任务间的依赖关系（如 Ring AllReduce 的流水线特性）
4. **调度粒度是 per-packet 的**：对训练场景的更大粒度（per-iteration communication burst）可能不适用
5. **无 GPU Direct 考虑**：sNIC 与 GPU 的 DMA 路径与纯 host 内存 DMA 路径可能有不同的争用特征
6. **实验规模非常有限**：评估仅涉及 2-4 个租户、单节点，未评估大量训练任务同时运行的场景

## 7. 关键引用与延伸阅读

### sNIC / on-path NIC 相关工作
- **PsPIN [18] (ISCA 2021)**：OSMOSIS 的实现后端，RISC-V based 可编程 on-path sNIC
- **FairNIC [31] (SIGCOMM 2020)**：SmartNIC 性能隔离（但采用静态分配，非工作保持）
- **iPipe [59] (SIGCOMM 2019)**：SmartNIC 卸载分布式应用（拥塞时回退到 host CPU）
- **PANIC [58] (OSDI 2020)**：面向 FPGA-sNIC 的多租户可编程 NIC

### 相关调度工作
- **BVT [21] (SOSP 1999)**：OSMOSIS WLBVT 的策略基础
- **Shinjuku [46] (NSDI 2019)**：微秒级抢占式调度
- **Dominant Resource Fairness [30] (NSDI 2011)**：多资源公平分配理论基础

### 对本课题的延伸
- OSMOSIS 为 SmartNIC 上的训练通信加速（in-network AllReduce）提供了多租户公平性的解决方案
- 其 per-packet cost-invariant 的调度思想可直接应用于训练任务的 per-iteration 通信调度

# Harmonic: Hardware-assisted RDMA Performance Isolation for Public Clouds

## 1. 论文基本信息

- **标题**：Harmonic: Hardware-assisted RDMA Performance Isolation for Public Clouds
- **作者**：Jiaqi Lou (UIUC), Xinhao Kong (Duke), Jinghan Huang (UIUC), Wei Bai (Microsoft/NVIDIA), Nam Sung Kim (UIUC), Danyang Zhuo (Duke)
- **发表**：NSDI 2024（CCF-A）
- **引用数**：20
- **与本课题相关性**：core
- **来源 query**：Q1

## 2. 研究问题与动机

### 核心问题
RDMA 正在被引入公有云以加速租户工作负载，但存在一个关键的缺失组件——**性能隔离（performance isolation）**。已有 RDMA 性能隔离方案（如SR-IOV、单独硬件队列、Justitia等）仅关注传统网络带宽（BPS），忽略了 RNIC 微架构资源（on-NIC cache、处理单元PU、PCIe带宽），因此在微架构资源争用时提供不足的隔离。

### 具体挑战
1. **（C1）准确测量每个租户的 RNIC 资源使用量**：RDMA 流量绕过内核（kernel bypass），无法在系统软件层面拦截监控；商用RNIC只暴露聚合统计数据，无法区分租户。
2. **（C2）找到合适的速率限制执行入口**：系统软件不适合作速率限制点（数据路径绕过CPU）；商用RNIC不提供逐租户、逐操作类型的速率限制能力；丢弃数据包会严重损害RDMA性能。

### 动机证据
- 恶意或有缺陷的租户可以通过大量ATOMIC操作耗尽NIC处理单元，严重降低其他租户的RDMA性能。
- Husky [29] 已证明现有所有RDMA性能隔离方案均不充分。

## 3. 核心方法

Harmonic 采用**硬件/软件协同设计**，包含三个关键创新：

### 3.1 RDMA 性能抽象扩展
在传统 BPS 之外，引入 RDMA 特定架构资源：QP数量、CQ数量、MR数量和大小、数据动词RPS（DRPS）、控制动词RPS（CRPS）。ATOMIC操作被归一化为3个单位的SEND操作。微架构资源（缓存、处理单元）不直接暴露给用户但被系统监控。

### 3.2 可编程智能 PCIe 交换机（PIPS）
- **部署位置**：FPGA 实现的 PCIe 交换机，插入 RNIC 与主机之间
- **核心原理**：所有 RDMA 流量经过 PCIe 总线；RDMA 将所有对象（payload、QP、CQ 等）pin 在主机 DRAM 中，物理地址到租户/对象的映射不变。PIPS 解析 TLP 头部中的物理地址字段，与映射表匹配置别租户和操作类型
- **实现**：AMD/Xilinx Versal VCK190 FPGA，4K 行 Verilog RTL。含5个模块：内核驱动(M1)、PCIe交换机(M2)、主机-PIPS通信接口(M3)、映射管理器(M4)、TLP分析器(M5)。映射管理采用双层哈希结构（L1直接映射+L2链表槽池）
- **优势**：在PCIe层面捕获所有RDMA行为（包括cache miss导致的额外DMA），不依赖应用层库的配合

### 3.3 RDMA 友好的速率限制器
- **核心洞察**：复用商用RNIC已有的 DCQCN 拥塞控制速率限制器。Harmonic daemon 主动伪造并注入 CNP（Congestion Notification Packet）来限制目标租户的发送速率
- **控制逻辑**：Harmonic daemon 定期轮询 PIPS 采集的统计信息，计算每租户 NIC BPS、PCIe BPS、DRPS 和 cache miss频率。当租户超量使用时，发送CNP进行限速。cache 资源通过监控全局cache争用和租户优先级来决定限速
- **DRF（Dominant Resource Fairness）**：PCIe带宽按主导资源公平模型分配
- **控制动词限速**：在修改后的内核驱动中实现（mlx5_ib.ko），通过sleep延迟而非返回错误来透明限速

### 部署模型
适用容器化云（租户共享内核驱动）和准虚拟化VM场景。需要云提供商控制内核驱动以拦截控制动词。当前不适用纯SR-IOV虚拟化方案。

## 4. 与课题相关性

### 与本轮精读目标的直接关联
本课题关注的"RDMA多租户性能隔离"正是 Harmonic 的核心研究问题。Harmonic 直接回答了：
- **已有方案覆盖了什么**：传统BPS隔离（SR-IOV、HW TC）和软件层调度（Justitia）能处理带宽争用，但对微架构资源争用（cache、处理单元、PCIe带宽）无效
- **不足在哪里**：Harmonic 证明了忽视 RNIC 微架构资源会导致隔离失败——这一发现通过 Husky 测试套件提供了系统性的实证证据

### 对假设的验证
- H1（"多租户RDMA网络共享下存在不可忽视的相互干扰"）获得**强支撑**：Harmonic 的评估明确展示了 ATOMIC 攻击可使其他租户吞吐下降 38%，cache 攻击和 PCIe 攻击同样造成严重性能损失
- H2（"现有方案未充分考虑训练场景的通信模式特征"）：Harmonic 关注的是通用 RDMA 多租户隔离，**未专门针对分布式训练**。其归一化处理（ATOMIC=3×SEND）假设固定比例，不考虑训练中的迭代通信模式（如AllReduce的同步性、突发性）

### 与本课题的技术衔接点
- Harmonic 的 PIPS 监控方案提供了逐租户资源使用的精确测量能力——这是感知训练通信特征的**前提基础设施**
- Harmonic 的 CNP 注入速率控制是一种粗粒度机制——可以改进为**感知通信模式**（如梯度同步阶段需要更高带宽保证）
- Harmonic 的归一化成本模型（ATOMIC=3×SEND）基于微基准离线评测，**不适合动态变化的训练通信模式**

## 5. 实验与评估

### 实验设置
- 硬件：两台服务器，各配一块 NVIDIA ConnectX-6 Dx 25Gbps RNIC。FPGA PCIe交换机支持 PCIe Gen4×8（最高128Gbps），但因25Gbps网卡限制，实际评估在25Gbps下进行
- 测试工具：Husky 测试套件（RDMA性能隔离专用）、Redis over RDMA（真实应用）
- 基线比较：SR-IOV、单独硬件流量类别（HW TC）、Justitia
- 隔离目标：两租户各保证12.5 Gbps + 15M DRPS

### 主要结果
1. **测量精度**：PIPS 可准确区分不同类型RDMA操作（WRITE/ATOMIC/READ）并测量每类请求速率（误差极小）
2. **速率控制精度**：CNP-based速率限制器可精确控制租户的 BPS 和 DRPS，响应时间 <1ms
3. **Husky 测试**：Harmonic 是**唯一通过全部 Husky 测试**的方案（容忍度 α=20%）。在所有资源争用场景下均保护受害租户达成保证
4. **处理能力争用**：HW TC/SR-IOV 失败（ATOMIC BPS低但消耗处理能力高），Justitia 更差（未区分动词成本）。Harmonic 考虑 ATOMIC 高成本并适当限速
5. **Cache 争用**：SR-IOV、HW TC、Justitia 均无法满足双方保证。Harmonic 检测 cache 争用，按公平份额分配降低的可用 BPS
6. **PCIe 争用**：Harmonic 通过 PIPS 硬件监控准确追踪每租户PCIe带宽消耗并分配
7. **Redis 应用**：Harmonic 在 cache/PCIe 争用下比 SR-IOV+HW TC 组合**高出 1.3x~1.4x** 应用吞吐
8. **开销**：延迟增加 <2us（主要来自FPGA实现而非监控功能），带宽/吞吐下降可忽略。PCIe带宽额外消耗 <0.31%。CPU 占用 33.5%单核

### 可扩展性论证
作者在 §7 详细论证了向 100/200 Gbps 扩展的可行性：TLP分析器在250MHz下平均7周期查一条映射，可并行扩展；映射管理器可扩展为多重哈希；ASIC实现将大幅优化。

## 6. 局限与 Gap

### 论文明确指出的局限
1. **FPGA 原型限制**：当前仅支持 25 Gbps，受 FPGA PCIe PHY 物理层限制（8-lane edge connector）。高速度部署需ASIC集成或未来RNIC内嵌（论文披露正集成到某领先企业的下一代RNIC）
2. **CNP 伪造的局限性**：软件发送 CNP 并非最优方案。瞬时网络拥塞可能影响速率限制精度。论文指出可编程拥塞控制（PCC）是更好的未来方向
3. **成本模型粗糙**：动词归一化基于离线的固定比例（WRITE=1, READ=1.1, ATOMIC=3），静态且不感知工作负载特性
4. **Cache 资源管理简单**：仅在全局 cache 争用严重时按优先级限速，不设置逐租户 cache miss 阈值。论文承认有更优策略但未深入

### 论文未涉及但对本课题重要的 gap
1. **不感知分布式训练通信模式**：Harmonic 是通用 RDMA 多租户隔离方案，完全没有考虑分布式训练场景的通信特征（如 AllReduce 的同步性、Ring/Gather 阶段的 bursty 流量、迭代间的空闲期）
2. **无时间维度隔离**：Harmonic 提供的是空间维度的资源公平分配，不感知租户通信在时间上的突发/空闲模式。分布式训练恰好利用这种模式（每迭代周期：计算→通信→计算）
3. **跨主机协调缺失**：Harmonic 只在本主机内隔离 RNIC 资源，不协调跨主机的端到端通信调度
4. **未评估多租户训练叠加场景**：实验仅涉及两租户（受害+攻击），未评估多个训练任务同时运行时的交互效应
5. **CNP 注入可能干扰训练同步通信**：训练任务对延迟敏感，CNP 的速率调制可能破坏梯度同步的时效性

## 7. 关键引用与延伸阅读

### 直接前置工作
- **[29] Kong et al., Husky (NSDI 2023)**：RDMA微架构资源隔离理解的开创性工作，提出了Husky测试套件。Harmonic 使用Husky作为评估工具
- **[62] Zhang et al., Justitia (NSDI 2022)**：基于软件的多租户RDMA隔离方案，要求修改应用库，Harmonic 无需此要求
- **[27] Kalia et al. (USENIX ATC 2016)**：高性能RDMA系统设计指南，揭示RNIC微架构资源的性能影响

### 相关 RDMA 虚拟化工作
- **[49] Pfefferle et al., HyV (VEE 2015)**：Hybrid I/O虚拟化框架
- **[22] He et al., MasQ (SIGCOMM 2020)**：RDMA用于虚拟私有云
- **[28] Kim et al., FreeFlow (NSDI 2019)**：基于软件的容器化RDMA网络虚拟化

### 与本课题相关的后续探索方向
- 分布式训练通信调度：如何将 Harmonic 的逐租户资源监控与训练任务的迭代通信模式相结合
- 端到端隔离：Harmonic 只解决 RNIC 端隔离，需结合网络端隔离（如交换机处的带宽保证）
- 可编程拥塞控制（PCC）：论文 §7 提到可用 PCC 替代 CNP 注入实现更精确的速率控制

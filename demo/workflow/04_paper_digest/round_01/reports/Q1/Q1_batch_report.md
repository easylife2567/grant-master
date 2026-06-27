# Batch Q1 精读报告

**论文数**：6 篇（core 6，general 0）
**来源 query**：Q1 — "系统调研 RDMA 多租户性能隔离和分布式训练通信调度的现有方法"

## 1. 各论文摘要

### [Harmonic: Hardware-assisted RDMA Performance Isolation for Public Clouds]（2024 NSDI，引用 20）[core]

- **研究问题**：如何实现感知 RNIC 微架构资源（cache、PU、PCIe）的 RDMA 多租户性能隔离？
- **核心方法**：FPGA 可编程智能 PCIe 交换机（PIPS）监控每租户 RDMA 资源使用 + 复用 DCQCN 拥塞通知包（CNP）限速。硬件/软件协同设计。
- **关键结论**：唯一通过 Husky 测试套件的方案。相比 SR-IOV+HW TC，在 cache/PCIe 争用下为 Redis 提升 1.3-1.4x 吞吐。正集成到某领先企业下一代 RNIC。
- **与课题相关性**：直接相关。提供了硬件辅助微架构感知隔离的完整原型，但不感知训练通信模式。
- **潜在 gap**：静态 verb 成本归一化（ATOMIC=3x SEND），不感知训练迭代通信特征。
- → [详细报告](papers/2024_Harmonic_RDMA_Isolation.md)

### [Husky: Understanding RDMA Microarchitecture Resources for Performance Isolation]（2023 NSDI，引用 66）[core]

- **研究问题**：RNIC 微架构资源（cache、PU、PCIe）如何被各类 RDMA 操作消耗？现有隔离方案能否提供充分隔离？
- **核心方法**：在 4 种 100Gbps RNIC 上系统实验，揭示 4 大发现，构建 RDMA 操作-资源消耗模型，开发 Husky 测试套件（52 攻击 + 20 受害工作负载）。
- **关键结论**：所有现有方案（SR-IOV、HW TC、Justitia 及组合）均无法通过 Husky 测试。仅需 1Gbps 精心构造攻击即可破坏 50Gbps 隔离保证。
- **与课题相关性**：本课题的基础评估工具和问题定义来源。allreduce 受害负载直接相关训练场景。
- **潜在 gap**：仅做问题揭示无解决方案；allreduce 作为稳态负载未考虑训练迭代模式。
- → [详细报告](papers/2023_Husky_RDMA_Microarchitecture.md)

### [PeRF: Preemption-enabled RDMA Framework]（2024 USENIX ATC，引用 0）[core]

- **研究问题**：如何在不牺牲裸金属性能的前提下，用纯软件实现 RDMA 工作保持（work-conserving）的性能隔离？
- **核心方法**：利用 RDMA Managed QP + WAIT/ENABLE 动词实现 RNIC 抢占机制，通过消息分割 + 0_WAIT 插入实现消息级隔离，PAUSE/RESUME 实现 QP 级隔离。
- **关键结论**：相比 Justitia 吞吐提升 ~2.04x。CPU 开销 <5%，性能接近硬件方案。支持 SEND/READ/WRITE。
- **与课题相关性**：纯软件、工作保持的隔离方案，与 Harmonic 形成互补。抢占机制（时间分片）与训练迭代模式有天然衔接可能。
- **潜在 gap**：不感知训练通信模式，分类器基于消息大小而非通信语义。不支持 UD/ATOMIC。
- → [详细报告](papers/2024_PeRF_Preemption_RDMA.md)

### [Palladium: A DPU-enabled Multi-Tenant Serverless Cloud]（2025 SIGCOMM，引用 2）[core]

- **研究问题**：如何在 serverless 多租户场景中利用 DPU 实现 RDMA 零拷贝数据平面的高性能 + 多租户隔离？
- **核心方法**：DPU Network Engine (DNE) 作为 off-path 反向代理隔离 RDMA QP；cross-processor 共享内存消除 DPU-Host 数据搬移；DWRR 调度实现租户间公平带宽分配。
- **关键结论**：吞吐提升 20.9x（vs NightCore），延迟降低 21x。2 个弱 DPU 核相当于节省 7 个 CPU 核。
- **与课题相关性**：提供了 DPU offload 架构下的 RDMA 多租户隔离方案。DWRR 权重调度可启发训练场景按任务优先级分配带宽。
- **潜在 gap**：完全针对 serverless 场景，DWRR 权重静态不变，不感知训练任务迭代周期。
- → [详细报告](papers/2505.11339.md)

### [OSMOSIS: Enabling Multi-Tenancy in Datacenter SmartNICs]（2024 USENIX ATC，引用 0）[core]

- **研究问题**：on-path SmartNIC 上执行不可预测复杂度的用户 kernel 时，如何公平分配 PU、DMA、Egress 三种资源？
- **核心方法**：WLBVT 调度器（BVT+WFQ 混合）公平分配 PU 时间；WRR + 传输分片消除 DMA/Egress 的 HoL 阻塞。RISC-V PsPIN 实现。
- **关键结论**：PU 公平性（Jain's 0.946 vs RR 0.643），受害流 FCT 降低 39%（compute）至 63%（IO）。硬件面积开销仅 ~1%。
- **与课题相关性**：从 SmartNIC on-path 执行角度补充了 RNIC 微架构隔离的另一个维度——计算资源公平性。Allreduce 被作为 compute-bound 负载示例。
- **潜在 gap**：面向 on-path sNIC 而非传统 RNIC；不感知跨节点训练任务协同调度。
- → [详细报告](papers/2023_OSMOSIS_SmartNIC_MultiTenant.md)

### [Slingshot: Multi-Tenant RDMA for Kubernetes]（2025 IEEE Cluster，引用 1）[core]

- **研究问题**：如何将 HPE Slingshot 200Gbps RDMA 网络接入 Kubernetes 容器化多租户环境？
- **核心方法**：netns-based 认证替代 UID/GID；CNI 插件管理 CXI Service 生命周期；VNI Service 全局管理虚拟网络（VNI）分配/回收。
- **关键结论**：通信性能开销 <1%，作业准入开销 <3.5%。实现容器粒度的安全 RDMA 通信域隔离。
- **与课题相关性**：提供通信域安全隔离（谁可通信），而非性能隔离（占用多少资源）。与本批次其他论文互补。
- **潜在 gap**：不提供性能隔离；不感知工作负载通信模式。
- → [详细报告](papers/2508.09663.md)

## 2. 批次内跨论文发现

### 2.1 共同主题/技术趋势

1. **微架构资源感知是 RDMA 多租户隔离的核心挑战**：从 Husky（问题揭示）→ Harmonic（硬件方案）→ OSMOSIS（SmartNIC 计算资源），一致表明仅靠带宽隔离（BPS）已完全不够。RNIC cache、PU、PCIe 带宽的争用可被极少量攻击流量利用。

2. **三层隔离体系正在形成**：
   - **通信域隔离**（Slingshot VNI）：谁可以跟谁通信
   - **性能资源隔离**（Harmonic/PeRF/OSMOSIS/Palladium）：如何公平共享 RNIC 资源
   - **工作负载感知调度**（本批次所有论文均缺失）：如何根据应用特征优化隔离策略

3. **工作保持（work-conserving）是软件隔离方案的关键需求**：PeRF 通过抢占实现工作保持，Palladium 通过 DWRR + off-path DPU 保持，OSMOSIS 通过 WLBVT 保持。静态预留（Justitia、FairNIC）会导致 RNIC 资源利用率大幅下降。

4. **硬件辅助 vs 纯软件的路线分歧**：Harmonic（FPGA PIPS）和 OSMOSIS（硬件 WLBVT scheduler）代表 HW/SW co-design 路线；PeRF 代表纯软件、零硬件依赖路线。前者更精确但部署成本高，后者更灵活但有控制粒度限制。

### 2.2 互相印证

- **Husky ↔ Harmonic**：Husky 定义了 RDMA 性能隔离的问题空间和评估基准；Harmonic 是第一个完全通过 Husky 测试的方案。两者形成"问题定义-解决方案"的完整链路。
- **Husky ↔ OSMOSIS**：Husky 揭示 RNIC 微架构资源争用；OSMOSIS 独立证明了 SmartNIC 上类似的多资源公平分配需求（PU、DMA、Egress），从不同硬件维度验证了同一核心论点。
- **PeRF ↔ Palladium**：两者都强调工作保持的重要性——PeRF（抢占式）、Palladium（DWRR + off-path），从不同技术路线达成一致：非工作保持的软件隔离性能损失过大。

### 2.3 互相矛盾或争议

- **双向 RDMA vs 单向 RDMA**：Palladium 明确论证双向 RDMA 更适合 serverless 零拷贝数据面（比单向+分布式锁快 2.3x）。PeRF 主要基于 WRITE（单向）但通过 RPC 包装支持 READ。Harmonic 通过 CNP 限速主要控制数据发送方（单向）。三种方案对 RDMA 原语选择有不同倾向。
- **硬件侵入度权衡**：Harmonic 需 FPGA（PCIe 交换机改装），OSMOSIS 需 ASIC 面积（1% overhead），Palladium 依赖 DPU（NVIDIA BlueField），PeRF 零硬件依赖。不存在共识——路径选择取决于部署场景的硬件约束。

### 2.4 本批次覆盖的空白

**已覆盖**：
- RNIC 微架构资源争用的全貌（Husky）
- 硬件辅助微架构感知隔离（Harmonic）
- 纯软件抢占式性能隔离（PeRF）
- DPU offload 架构下的隔离（Palladium）
- SmartNIC 计算+IO 资源公平分配（OSMOSIS）
- HPC 网络到云原生的通信域隔离（Slingshot）

**未覆盖但对课题至关重要**：
1. **感知分布式训练通信模式的调度/隔离方案**——所有 6 篇论文均将应用当作稳态流量，不区分计算迭代和通信阶段
2. **跨节点协同的端到端通信调度**——所有方案作用域均限于单个 RNIC/sNIC，缺乏全局调度视角
3. **GPU Direct RDMA 场景下的性能隔离**——GPU 与 RNIC 直连的 PCIe 拓扑带来新的资源争用类型
4. **>2 个训练任务的竞争效应**——实验均为 2-3 个租户，大量并发训练任务的交互未评估
5. **训练通信原语（AllReduce/AllGather/ReduceScatter）对各类 RNIC 资源的不同敏感性**
6. **动态工作负载（burst/空闲交替）下的自适应隔离策略**

## 3. 假设验证汇总

| 假设 | 支持论文 | 否定/弱化论文 | 判断 |
|------|---------|-------------|------|
| H1：多租户RDMA网络共享场景下，不同训练任务的通信性能确实存在不可忽视的相互干扰 | Husky（52种攻击只需极低带宽即破坏隔离）、Harmonic（ATOMIC攻击降38%、cache攻击降50%）、OSMOSIS（不同cost-per-byte导致RR调度严重不公平） | Slingshot（VNI隔离后无通信域交叉，但未评估共享NIC的性能干扰） | **strongly supported** |
| H2：现有的RDMA性能隔离方案（如PeRF、Harmonic）未充分考虑训练场景的通信模式特征 | Husky、Harmonic、PeRF、Palladium、OSMOSIS、Slingshot——均未将训练迭代的burst/空闲模式纳入设计 | — | **strongly supported**（6/6论文均无训练感知） |
| H3：感知训练任务通信特征的调度策略相比纯网络层调度可取得更好的隔离效果 | —（本批次无论文实现此策略） | — | **not yet evaluated**（需本课题后续工作验证） |

## 4. 问题与备注

1. **Harmonic 与 PeRF 的互补性值得深入探讨**：Harmonic 解决了"监控精度"问题（PIPS 逐租户逐操作监控），PeRF 解决了"控制效率"问题（工作保持抢占）。两者结合（PIPS 监控 + 抢占式控制）可能实现比各自单独更优的隔离效果。

2. **OSMOSIS 的 WLBVT 调度思想可启发训练场景的 per-iteration 调度**：不依赖预先知道梯度大小，只追踪每任务的累计 PU 时间进行公平分配。可直接应用于多个训练任务共享同一 RNIC 的场景。

3. **Palladium 的 DNE 架构为训练任务 DPU offload 调度提供了参考蓝图**：将训练通信调度从 GPU 服务器 CPU 卸载到 DPU，利用 off-path 模式保持 RDMA 直通性能。

4. **Slingshot 的 VNI + Traffic Class 双维度隔离是训练场景的理想抽象**：VNI 实现安全隔离（任务间不可见），Traffic Class 实现性能优先级（梯度同步高优、数据加载低优）。但目前 Slingshot Traffic Class 的 QoS 配置未被论文实现。

5. **Husky 测试套件应成为本课题方案评估的标准工具**：任何训练感知的调度方案，首先应通过 Husky 的基本隔离测试（确保不弱于 Harmonic），然后扩展到训练场景特定的通信模式评估。

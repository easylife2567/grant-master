# Understanding RDMA Microarchitecture Resources for Performance Isolation

## 1. 论文基本信息

- **标题**：Understanding RDMA Microarchitecture Resources for Performance Isolation
- **作者**：Xinhao Kong, Jingrong Chen (Duke); Wei Bai, Mahmoud Elhaddad, Shachar Raindel, Jitendra Padhye (Microsoft); Yechen Xu (SJTU); Alvin R. Lebeck, Danyang Zhuo (Duke)
- **发表**：NSDI 2023（CCF-A）
- **引用数**：66
- **与本课题相关性**：core
- **来源 query**：Q1

## 2. 研究问题与动机

### 核心问题
RDMA 正在被云厂商引入多租户公有云，但 **RNIC 微架构资源（on-NIC cache、处理单元PU、PCIe带宽）** 的存在使得现有的 RDMA 性能隔离方案（SR-IOV、独立硬件流量类、Justitia）**全部失效**。本文系统性地揭示了这一问题的全貌。

### 动机
- 云基础设施已超4000亿美元规模，多租户资源共享下的性能隔离是核心需求
- RDMA 在大型云厂商内部（如 Azure storage）已大规模部署，但未向公有云租户开放——性能隔离缺失是主要障碍
- 论文用一个简洁的动机性实验开篇：SR-IOV+HW TC 配置下，1Gbps 精心构造的攻击流量即可将 50Gbps 保证的受害租户压至 2Gbps

### 核心贡献
首次系统性研究了**所有类型 RDMA 操作**（控制动词、数据动词、异常/错误处理）对 RNIC 微架构资源的消耗，提出了 RDMA 操作-资源消耗模型，并构建了 Husky 测试套件。

## 3. 核心方法

### 3.1 方法学
论文在4种100Gbps RNIC上进行了系统实验：NVIDIA ConnectX-5、ConnectX-6 Dx、Chelsio T62100-LP-CR（iWARP）、Intel E810（RoCE/iWARP）。使用 victim-attacker 模型设计实验，并利用 NVIDIA Neo-Host 管理工具获取硬件计数器（cache miss 等）。

### 3.2 四大关键发现

**发现 #1：控制动词可导致过度 cache miss 并严重降低性能**
- MR 注册操作导致 MTT cache miss 从 17.2% 增至 49.1%，带宽从 96.6 Gbps 降至 48 Gbps
- 攻击者无需消耗网络带宽——仅发出 deregistration 操作即可
- 根因（NVIDIA 确认）：RNIC 内部 QoS 调度策略将 deregistration 操作赋予最高优先级
- 控制动词比数据动词更容易引发 cache 干扰，因为其语义性操作（如 invalidate 整个 cache）影响更广

**发现 #2：不同数据动词的性能干扰取决于动词复杂度**
- ATOMIC 操作（CAS、FAA）的请求速率远低于 SEND/WRITE（约 5 Mrps vs 90+ Mrps），但消耗更多 PU 资源
- 不同动词组合的干扰行为差异显著：CAS 攻击可使 READ 受害从 60 Mrps 降至 3 Mrps
- 跨 RNIC 验证：Chelsio（iWARP）上也观察到类似但具体模式不同的结果

**发现 #3：错误处理可停顿 RNIC 处理单元并挂起所有应用**
- RNR（Receive Not Ready）错误可完全停顿 RNIC PU
- 仅 4 Kbps 的 SEND 流量配合 RNR 错误即可使受害 97 Gbps WRITE 降至几乎为零
- 有趣的是，不同连接类型的 RNR 行为不同：RC RNR 可被 SR-IOV 隔离，UD/UC RNR 不能
- Chelsio NIC 不受 RNR 影响（推测因其 iWARP over TCP 设计）

**发现 #4：PCIe 带宽仅在特定请求大小范围内成为瓶颈**
- 小消息（<28B）嵌入 WQE 中，不增加 PCIe 开销
- 中等消息（29-256B）PCIe 开销比例最大（可达网络带宽的 1.5x）
- 大消息被 RNIC 网络带宽先限流
- 29B 消息理论上需 148.8 Gbps PCIe 才能饱和 100 Gbps 网络

**其他发现**：不同数据动词争用不同 RNIC cache 类型；大规模对象跨广度访问是 cache miss 的关键触发条件，而非仅对象数量

### 3.3 RDMA 操作-资源消耗模型
将 4 大发现和其他发现综合为一个关系模型，描述各类 RDMA 操作与微架构资源的对应关系（定性模型）。该模型指导 Husky 测试套件设计。

### 3.4 Husky 测试套件
- 覆盖 4 类资源：NIC 带宽、PCIe 带宽、NIC PU、NIC cache
- 包含 **52 种攻击工作负载**和 **20 种受害工作负载**（含真实应用：allreduce、eRPC-based Masstree KVS）
- 扩展了 Collie 的流量引擎以支持控制动词和错误处理
- 隔离违规定义：受害性能 < (1-α)*min(Ba, Bg)，α=25%

## 4. 与课题相关性

### 与本轮精读目标的直接关联
Husky 是本课题涉及的所有 RDMA 多租户隔离工作的**基础验证工具**。它系统性地证明了：
- 已有方案覆盖了**传统网络带宽隔离**（BPS），但完全忽略了**微架构资源隔离**
- 所有现有方案（SR-IOV、HW TC、Justitia、及其组合）均无法通过 Husky 测试

### 对假设的验证
- H1（"多租户RDMA网络共享下存在不可忽视的相互干扰"）获得**最强的实证支撑**：Husky 用 52 种攻击工作负载证明了从 4 个维度（BW/PU/Cache/PCIe）都能破坏性能隔离，且仅需极少的攻击带宽
- H2（"现有方案未充分考虑训练场景通信模式特征"）：Husky 已包含 allreduce 作为受害应用（分布式训练的核心通信原语），评估显示 allreduce 对 cache 攻击和 PU 攻击**极度敏感**（cache 攻击下 SR-IOV+HW TC 保护后仍下降近 50%）。但 Husky **仅将 allreduce 作为测试对象，未提出训练场景专用的隔离策略**

### 对本课题的核心启示
- **Takeaway #3** 对本课题至关重要："分布式应用需要每个服务器的性能隔离"——训练任务中任何一个 worker 的 RNIC 性能受损即可拖慢整个迭代
- 论文明确提出了未来隔离方案的设计原则：**硬件支持是必要的**（类似 Intel RDT 的 cache 分区），以及需要**一层间接层**（硬件或微内核软件）
- Husky 为评估未来训练感知的调度方案提供了现成的测试基础

## 5. 实验与评估

### 实验环境
- 4 台服务器，每台 NVIDIA ConnectX-5 100Gbps RNIC，PCIe 3.0 x16
- Ubuntu 20.04, kernel 5.11, MLNX_OFED 5.4, firmware 16.31.1014
- 同样在 ConnectX-6 上复现

### 主要结果

**5.1 现有方案系统性评估**（Table 3）
| 隔离方案   | NIC BW | PU(Error RC) | PU(Error UD/UC) | PU(Data) | Cache(Ctrl) | Cache(Data) | PCIe |
|-----------|--------|-------------|-----------------|----------|-------------|-------------|------|
| SR-IOV    | OK     | OK          | FAIL            | FAIL     | FAIL        | FAIL        | FAIL |
| HW TC     | OK     | FAIL        | FAIL            | FAIL     | FAIL        | FAIL        | FAIL |
| SR-IOV+HW TC | OK  | OK          | FAIL            | FAIL     | FAIL        | FAIL        | FAIL |
| Justitia  | OK     | FAIL        | FAIL            | OK       | FAIL        | FAIL        | FAIL |
| Justitia+HW TC | OK | FAIL        | FAIL            | OK       | FAIL        | FAIL        | FAIL |

**5.2 真实应用评估**

**Allreduce（OSU Benchmark，MPI over RDMA）**：
- 8 workers 跨 4 台服务器，1MB buffer
- SR-IOV+HW TC 下：BW 攻击 OK | PCIe 攻击 -27.3% | Cache 攻击 -50% | PU 攻击 **完全停顿**
- Justitia+HW TC：性能先降 38.5%（软件层开销），然后各类攻击使其进一步下降

**eRPC-based Masstree（Key-Value Store）**：
- Cache 攻击影响较小（UD transport 对 cache 敏感度低）
- PCIe 攻击影响更显著（小消息开销大）
- PU 攻击完全停顿整个系统——即使非 colocated client 也受影响

### 关键 Takeaway
1. 针对微架构资源使破坏隔离变得**轻而易举**（仅需 7 Gbps Cache 攻击，<0.5 Gbps PU 攻击）
2. 不同应用对各类资源争用的敏感度差异很大
3. 分布式应用需在**每台服务器**上都确保隔离

## 6. 局限与 Gap

### 论文明确指出的局限
1. **模型是定性的**：无法理解 RNIC 内部的确切资源消耗量（如 cache 确切大小、一个操作消耗多少 cache），因为 RNIC 内部对研究者和云厂商都是黑盒
2. **仅评估了 NVIDIA 的隔离方案**：因技术限制未在 Chelsio 和 Intel NIC 上启用硬件隔离机制
3. **SR-IOV 实现细节不公开**：不同 VF 间的隔离机制和调度策略属于厂商机密

### 论文未涉及但对本课题重要的 gap
1. **仅做问题揭示，未提出解决方案**：Husky 证明现有方案失败，给出了设计指南，但本身不提供可行的隔离方案
2. **未考虑训练任务的时间模式**：allreduce 作为稳态应用评估，未考察训练中计算-通信交替的时间模式（burst 后在空闲期可被其他租户利用）
3. **实验规模有限**：仅 2-4 台服务器、2 个租户（victim+attacker），未评估 >2 个训练任务同时运行的交互效应
4. **未考虑不同通信集合操作的差异**：仅测试了 allreduce，未评估 allgather、reduce-scatter、broadcast 等不同集合操作对各类资源争用的敏感性差异
5. **测试套件的静态性**：Husky 的工作负载是预定义的，不支持动态调整攻击模式以自适应目标方案
6. **无时间窗口分析**：所有评估关注稳态性能，未分析攻击的瞬时效果和恢复时间

## 7. 关键引用与延伸阅读

### 直接方法论基础
- **Collie [33] (NSDI 2022)**：RDMA 性能异常的系统搜索，Husky 扩展了其流量引擎
- **Kalia et al. [29] (USENIX ATC 2016)**：RNIC 微架构行为的早期系统化理解
- **ScaleRPC [5] (EuroSys 2019)**：揭示了 QP context cache miss 的 scalability 问题

### 被评估的隔离方案
- **Justitia [71] (NSDI 2022)**：软件层 RDMA 多租户隔离
- **PicNIC [34] (SIGCOMM 2019)**：可预测虚拟化 NIC（非 RDMA 专用）

### 测试所用真实应用
- **MVAPICH [55]**：RDMA-based MPI 实现，用于 allreduce 测试
- **eRPC [27] (NSDI 2019)**：高效 RPC 框架，用于 Masstree 测试

### 对本课题的延伸意义
- Husky 的 allreduce 工作负载可直接用于评估训练感知调度方案的隔离效果
- 未来可扩展 Husky 以包含更多训练通信模式（Ring-AllReduce、Parameter Server 通信等）
- GPU Direct RDMA（GPUDirect）场景下的 RNIC 资源争用是 Husky 未覆盖的空白

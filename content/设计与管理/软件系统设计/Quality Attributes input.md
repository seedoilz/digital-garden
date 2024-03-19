---
aliases: 
date created: 四月 18日 2023, 10:14:23 上午
date modified: 三月 5日 2024, 4:07:11 下午
title: Quality Attributes input
tags: [input]
---

## 需求
### 功能性需求
定义系统必须做什么，并且强调系统如何提供价值给涉众。

### 质量需求
1. 系统应在功能性需求之上提供的整个系统的合乎需求的特性。
2. 质量属性是由软件系统的业务目标所决定的。
3. 分为两类
	1. 执行过程中可观察 (外部): 系统满足其行为要求的程度如何? 例如[[性能]]、[[安全性]]、[[可用性]]等
	2. 执行期间不可观察 (内部): 系统的维护、集成或测试有多容易? 例如[[可修改性]]、可移植性、可重用性和[[可测试性]]等。

## 质量属性
### 分类
Adaptability 适应性、Extensibility 可扩展性、Availability 可用性、Modularity 模块化、Configurability 灵活性、Portability 可移植性、Flexibility 灵活性、Reusability 可重用性、Interoperability 互操作性、Testability 可测试性、 Performance 性能、Auditability 可审核性、Reliability 可靠性、Maintainability 可维护性、Responsiveness 响应性、Manageability 可管理性、Recoverability 可恢复性、Sustainability 可持续性、Scalability 可扩展性、Supportability 可支持性、 Stability 稳定性、Usability 可用性、Security 安全性。

### 质量属性方案
质量属性方案用于定义所需的质量属性，是具有一定结构的简单句子。方案的主要类别为通用方案与具体方案。

#### 通用方案
与系统无关的方案，用于指导质量属性要求的规范。

#### 具体方案
具体方案是系统特定方案，用于指导特定系统的质量属性要求的规范，是通用方案的实例。

### 方案模版：表与 Stimulus-Response 图
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230424180209.png)
>刺激 (Stimulus): 到达系统时需要考虑的条件。
>刺激源 (Source of Stimulus):产生刺激的实体 (人，系统或任何其他触发)，可能是输入、信息等，对当前的状态的一个变化。
>响应 (Response): 刺激到来后工件开展的行为。
>响应度量 (Response Measure): 对刺激的响应以某种方法进行测量，以便可以测试需求 (比如多长时间系统有反馈)
>环境 (Environment): 发生刺激时系统的状况，例如系统正常运行、系统过载、系统受到攻击、系统网络出现故障等。
>工件 (Artifact):完成需求的整个系统或者系统的一部分 (软件制品)。

## [[决策]] (Tactics)
### 定义
Tactics 是影响质量属性相应控制的设计决策，比如冗余。
**体系结构 即是决策的集合。**

### 决策与模式的区别
1. 决策比模式更简单，仅有单一的结构或机制来应对单一的架构驱动
2. 决策是创建架构设计的重要组成块。
3. 模式通常将多个设计决策组合到一个包中。
4. 模式和策略共同构成了软件设计师的主要工具。
5. 大多数模式包含集中不同的策略，这些策略可能有共同的目的或者经常被选择来实现不同的质量属性。

### 7 种设计决策
1. 职责分配: 将大的职责进行拆分。
2. 协调模型: 各部分之间的沟通与交互。
3. 数据模型: 数据格式、存储方式 (缓存等)。
4. 资源管理: CPU、网络、内存、时间等资源。
5. 架构元素映射: 架构元素如何映射到具体的软件实现。
6. 绑定时间决策:设置时间点 (这个时间点前系统还是可以变化的，但之后不可以)，比如选择安装环境需要在一个时间点前确认，同样的还有技术是否添加、编译时间、初始化时间、运行时绑定，但运行时弹性最大 (尽可能延迟绑定时间)
7. 技术选择: 之前的部分已经确定后，我们可以选择的技术栈比较局限。

#### 约束
**约束**是具有零自由度的设计决策，是已经做出的预先指定的设计决策，需要通过接受设计决策并将其与其他受影响的设计决策进行协调，可以满足约束条件。

## [[可用性]] Availability
### 定义
可用性是应用程序的关键要求，以所需的可用时间来度量。

### Outage、Failure、Fault 和 Error:
1. Outage: 系统不可用的情况，比如 Scheduled Downtime。
2. Fault: Failure 的原因。
3. Error: Fault 发生与 Failure 之间的中间状态。
4. Failure: 系统状态的可观察特征，系统无法交付该系统期望的服务。
>影响因素:发现故障的时间、纠正故障的时间、重启应用的时间。
> MTBF(Mean Time Between Failure)  
>MTTR(Mean Time To Repair)  
>可用性: $$\frac{MTBF}{MTBF + MTTR}$$

### 可用性通用方案 Availability General Scenario
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/11.png)

### 可用性策略
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/13.png)

## [[互操作性]] Interoperability 
### 定义
互操作性是指两个或多个系统可以在特定的上下文中通过接口有效交换有意义的信息的程度，包括语法可操作性 (交换数据的能力)和语义可操作性 (能够正确解释数据)。互操作性需要确定谁，什么以及在什么情况下 (上下文)。
>影响因素:发现(发现服务位置、身份和接口)、处理响应(返回、转发、广播)。

### 互操作性的通用方案 Interoperability General Scenario
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/16.png)

### 互操作性的策略 Interoperability Tactics
1. 输入:信息交换请求
2. 定位:发现服务，通过已知目录服务来找到服务。
3. 管理界面:编排(Orchestrate)、定制界面
4. 输出:请求被正确处理

## [[可修改性]] Modifiability
### 定义
可修改性涉及到更改以及进行更改所需花费的时间或金钱，包括这种可变更性影响其他功能或质量属性的程度。
>影响因素:变更是什么?变更可能性?何时谁进行变更?变更成本?

### 可修改性的通用方案 Modifiability General Scenario
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/21.png)

### 可修改性的策略 Modifiability Tactics
1. 输入:出现变更拆分
2. 模块:将包含大量功能的模块拆分。
3. 增加语义一致性:将用途不同的功能进行拆分。
4. 减少耦合:封装(减少更改弥散的可能)、使用中介打破依赖、重构(两个模块受到相同变更的影响)。
5. 延迟绑定:尽可能将绑定时间延迟(设计时、开发时、测试时、发布时)
6. 输出:在一定时间和预算内实现变更。

## [[性能]] Performance
### 定义
性能与时间有关，和系统满足时序要求的能力有关(单位时间能做多少事情)。
>影响要素:处理时间(正在相应)和阻塞时间(无法相应)。

### 性能的通用方案 Performance General Scenario
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/26.png)

### 性能策略
1. 输入: 事件到达
2. 控制资源需求: 管理采样频率、限制事件响应 (排队)、事件优先级排序、减少过载、限制处理时间、提高资源利用率。
3. 管理资源: 增加资源、引入并发、计算资源多备份、数据资源多备份、限制队列长度、调度资源。
4. 输出: 在一定时间限制内响应生成。

## [[安全性]] Security
### 定义
安全性衡量系统保护数据和信息免遭未授权应用的能力，同时仍提供对授权人员和系统的访问权限。
>影响要素 (CIA):保密性 (Confidentiality 防止未授权访问)、完整性 (Integrity 防止未授权操做)、可用性 (Availability 系统可供合法使用)。

### 安全性的通用方案 Security General Scenario
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/31.png)

### 安全性策略
1. 输入:攻击
2. 检查攻击:发现入侵 (通过流量或签名)、检测服务拒绝、检查消息完整性 (通过校验或哈希)、检查消息延迟
3. 防御攻击:验证 (Identity)、授权 (Authorize)、认证 (Authenticate)请求者、限制资源访问、限制暴露、加密数据、分离实体、修改默认设置。
4. 响应攻击: 撤销对敏感资源访问、锁定电脑、告知请求者。
5. 从攻击中恢复: 对系统的追踪、快照存储。
6. 输出: 系统检测、防御、响应或恢复。

## [[可测试性]] Testability
### 定义
可测试性是指可以使软件通过(通常是基于执行)测试来证明其故障的难易程度。

### 可测试性的通用方案 Testability General Scenario
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/37.png)

### 可测试性策略
1. 输入: 执行测试。
2. 控制和观察系统状态: 专用界面、记录或回放故障、本地化状态存储、沙盒 (保证实验消除后果) 
3. 限制复杂度:限制结构复杂性 (减少依赖关系、继承深度、多态和动态调用)、限制行为不确定性。
4. 输出: 检测到 Faults。

## [[易用性]] Usability
### 定义
易用性与用户完成所需任务的难易程度以及系统提供的用户支持的类型相关。
>影响因素: 学习系统功能、有效使用系统、最小化错误影响、使系统适应用户需求、增强信息和满意度。

### 易用性的通用方案 Usability General Scenario
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/42.png)

### 易用性的策略 Usability Tactics
1. 输入: 用户请求。
2. 支持用户操作: 取消、撤销、暂停恢复、将对象成组操作
3. 支持系统操作:维护任务模型 (确定上下文对任务建模)、维护用户模型 (预期的知识)、维护系统模型 (预期的系统行为)。
4. 输出: 用户被给出正确的反馈和帮助。

## [[质量属性具体方案对比]]
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230424205133.png)

## 架构攸关的需求 [[ASR]]
### 定义
Architecturally Significant Requirements
架构攸关需求是对体系结构产生深远影响的需求 (影响了关键体系结构设计决策)。

### 如何系统地识别 ASR 和其他因素
1. **从需求文档中收集 [[ASR]]**: 可以使用“MoSCoW”样式或用户故事来收集需求(不过难以收集质量需求)，但其中大部分的内容都不会影响体系结构，对架构师有用的部分甚至没有出现在需求文档中。
2. 通过采访涉众来收集 ASR:可以使用质量属性工作坊 (QAW)
	1. QAW 演示和介绍
	2. 业务任务介绍
	3. 架构计划介绍
	4. 架构驱动程序的识别
	5. 场景集思广益: 每一个利益相关者表达一个场景
	6. 方案合并 (合并类似方案)
	7. 方案优先级 (通过投票)
	8. 方案细化
3. 通过了解业务目标来收集 ASR
4. 通过质量属性实体树 (Utility)来管理 ASR: 使用方案量化描述需求后，逐渐对质量需求进行分解细化，直到含有量化指标为止。
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec13/48.png)
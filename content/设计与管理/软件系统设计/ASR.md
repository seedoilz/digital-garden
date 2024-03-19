---
aliases: [Architecturally Significant Requirements, 架构攸关的需求]
date created: 四月 24日 2023, 9:00:16 晚上
date modified: 三月 5日 2024, 4:07:11 下午
title: ASR
tags: [system-architecture/theory]
---

### 定义
Architecturally Significant Requirements
架构攸关需求是对体系结构产生深远影响的需求 (影响了关键体系结构设计决策)。

### 如何系统地识别 ASR 和其他因素
1. **从需求文档中收集 ASR**: 可以使用“MoSCoW”样式或用户故事来收集需求 (不过难以收集质量需求)，但其中大部分的内容都不会影响体系结构，对架构师有用的部分甚至没有出现在需求文档中。
2. 通过采访涉众来收集 ASR: 可以使用质量属性工作坊 (QAW)
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
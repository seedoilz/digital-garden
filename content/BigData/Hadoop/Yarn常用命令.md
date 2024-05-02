---
aliases: 
title: Yarn常用命令
date created: 2024-04-02 10:04:00
date modified: 2024-04-02 10:04:83
tags: [code/big-data]
---
### 查看任务
#### 列出所有Application
```shell
yarn application -list
```
#### 根据Application状态过滤
```shell
yarn application -list -appStates FINISHED
```
#### Kill掉Application
```shell
yarn application -kill <ApplicationId>
```
### 查看日志
#### 查看Application日志
```shell
yarn logs -applicationId <ApplicationId>
```
#### 查询Container日志
```shell
yarn logs -applicationId <ApplicationId> -containerId <ContainerId>
```
### 查看容器
#### 列出所有Container
```shell
yarn container -list <ApplicationAttemptId>
```
#### 打印Container状态
```shell
yarn container -status <ContainerId>
```
### 查看节点状态
```shell
yarn node -list -all
```
### 查看队列
```shell
yarn queue -status <QueueName>
```

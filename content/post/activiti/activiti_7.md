---
title: "activiti 7 简介"
date: 2019-02-18T09:50:10+08:00
categories: ["activiti"]
---

# activiti 7
## 1.[Activiti Cloud介绍](https://activiti.gitbook.io/activiti-7-developers-guide/overview)
Activiti Cloud是第一个Cloud Native BPM框架，旨在为云环境中的BPM实施提供可扩展且透明的解决方案。

The BPM discipline was created to provide a better understanding of how organisations do their work and how this work can be improved in an iterative fashion

> BPM:Business Process Management。商业流程管理。旨在帮助组织更容易理解他们所做的工作以及如何以迭代方式改进这项工作

通常BPM套件部署在一个中央服务器，采用BPM套件常会遇到的痛点：

* BPM Suites are built on top of a technology stack that the IT Department(Software) inside the organization doesn’t otherwise need to know
* BPM Suites often don’t integrate nicely with other ecosystems
* Users need to learn a whole new toolset to do the work
* The department in charge of the infrastructure where the BPM Suite will run doesn’t know about its requirements. Same applies for DBAs that need to understand how the BPM Suite works to tune their Databases.
* User Interfaces provided by BPM Suites are often not flexible enough. Many BPM implementations end up needing multiple re-implementations of their User Interfaces
* BPM Suite adoptions are usually pushed from the business and not backed up by the internal software development teams inside the organization and this friction cause delays and problems during implementation.

* BPM套件构建在技术堆栈之上，组织内的IT部门（软件）无需另外知道
* BPM套件通常不能很好地与其他生态系统集成
* 用户需要学习一个全新的工具集来完成工作
* 负责BPM Suite运行的基础架构的部门不了解其要求。 同样适用于需要了解BPM Suite如何调整其数据库的DBA。
* BPM套件提供的用户界面通常不够灵活。 许多BPM实现最终需要多次重新实现其用户界面
* BPM Suite采用通常是从业务推出而不是由组织内部的内部软件开发团队备份，这种摩擦会导致实施过程中的延迟和问题。

大多数这些痛点都是因为BPM Suites强加了一整套技术，迫使组织适应它们。

Activiti Cloud试图将Activiti Process Engine剥离到最低限度并尽可能保持单一焦点。

The Process Runtime shouldn’t worry about:
* Where the process definitions are stored
* Dealing with process versions and changes
* Identity Management
* Single Sign-On
* Job executions
* Timer mechanisms
* Providing System to System integration mechanisms
* Sending Emails
* Storing History / Audit information and providing a way to query this information
* Performance of the clients consuming data generated by the engine
* Push notifications about the changes in state of the system

* 存储过程定义的位置
* 处理流程版本和更改
* 身份管理
* 单点登录
* 工作执行
* 定时器机制
* 为系统集成机制提供系统
* 发送电子邮件
* 存储历史/审计信息并提供查询此信息的方法
* 消费者使用引擎生成的数据的性能
* 推送有关系统状态变化的通知
---
title: DC synthesis Q&A
date: 2024-02-16 23:44:10
categories:
- DC synthesis
tags:
- DC
- synthesis
---

- **Q1：计算RC延时的两种方式**
  - WLM mode线性负载模型：根据fanout计算RC延时
  - Topo mode拓扑模式：会进行粗略的布局，和post-layout（布局布线之后） 的结果更加接近
- **Q2：DC和DC Ultra的区别**
  - DC 调用的综合命令是compile
  - DC Ultra调用的综合命令是compile_ultra
    - 需要DC Ultra以及DesignWare Foundation的license
    - 相比于compile，会提供额外的优化算法，例如`只有compile_ultra可以调用topo mode`进行优化

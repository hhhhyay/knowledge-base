---
title: GLM5-FP8 超节点对比风冷节点
tags:
  - AI
  - LLM
  - Benchmark
model:
  - GLM5-FP8
framework: []
issue_type: []
source: feishu
feishu_token: IECTwDoxBiZ7GTkVAjychFOPn0b
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/IECTwDoxBiZ7GTkVAjychFOPn0b
last_synced_at: 2026-06-07T17:07:53.393Z
---

# GLM5-FP8 超节点对比风冷节点

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/IECTwDoxBiZ7GTkVAjychFOPn0b)

- decode

### bs1

超节点，单次decode step时间为94ms左右，算子纯耗时83ms左右，两次step中间存在8ms左右空泡：

<image token="S1j3bAuO0oLMocxQ0EncwxcDn3g" width="3818" height="1758" align="center"/>

风冷，单次decode step时间为98ms左右，算子纯耗时94ms左右，两次step中间存在2ms左右空泡：

<image token="HiA7bVwWtorvY0x3CUqcmOl6nbg" width="2516" height="1756" align="center"/>

#### 存在问题：

1.  mtp draft extend for decode阶段少抓了算子，

风冷节点

<image token="J0fGblyzkoFzvgxpWFbcMat6nGc" width="3498" height="1450" align="left"/>

超节点

<image token="MSWjbWhc2olbYlxDXnhcHzMBnAe" width="3322" height="846" align="center"/>

1. 从上图中也可看出，风冷节点在mtp draft extend for decode阶段deepep耗时正常，但是超节点deepep dispatch耗时异常
1. 超节点gloo allgather耗时异常：

<image token="GWYrbVMypodm8qxhHrec2z9zncg" width="614" height="190" align="center"/>

风冷只有6ms左右

<image token="US3obGVZ9oj43sx9FAGcvTXwnEb" width="528" height="130" align="center"/>

### bs8

超节点，单次decode step时间为170ms左右：

<image token="ByzNbjqgIonJ34x90rwcek0DnYe" width="2369" height="1653" align="center"/>

风冷节点，单次decode step时间为190ms左右：

<image token="J9WnbhJV7otCVrxWcAbcqDWTnLf" width="2394" height="1737" align="center"/>

与bs1的对比，发现超节点和风冷节点的单步 decode 都接近翻倍：

超节点：94ms -> 170ms，增加约 76ms

<sheet token="JODfsHQFzhsjHktKcx4cT9Brnmg_3gQYS2"/>

风冷：98ms -> 190ms，增加约 92ms

<sheet token="JODfsHQFzhsjHktKcx4cT9Brnmg_JCMQWw"/>

### 5.26 bs8

算子tunning

<image token="QsN0bEii2oNfAvxBKFncQoVAn5d" width="1686" height="926" align="center"/>

<sheet token="JODfsHQFzhsjHktKcx4cT9Brnmg_Xc9eOj"/>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM5-FP8|GLM5-FP8]]（model）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

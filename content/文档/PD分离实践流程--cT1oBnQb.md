---
title: PD分离实践流程
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: AUiWwNFKkiuznhkCcWVcT1oBnQb
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/AUiWwNFKkiuznhkCcWVcT1oBnQb
last_synced_at: 2026-06-07T17:07:53.393Z
---

# PD分离实践流程

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/AUiWwNFKkiuznhkCcWVcT1oBnQb)

因为指标不同，要求不同PD分离测试没有标准的最佳实践，我们需要根据需求按照一定的步骤，去逐步测出最佳实践。以下是具体步骤。

本实践按照蚂蚁项目Ling1T模型输入4k输出1.5k，TTFT 3s，TPOT 100ms为例，求系统最大吞吐，pd配比及并行方案。

# 一、单测P节点吞吐。

## 1、并行方案选择：

P节点可选并行方案，根据不同指标，需求特点做选择:

### PP并行，特点吞吐高、延迟高。

吞吐高的原因：通信量小。

延迟高的原因：多个流水线造成prefill时候多个请求并行，并行会造成多个请求同时prefill推理，总吞吐虽然有所提高，但是对于单个请求，TTFT延迟会高一些。

### TP并行，特点吞吐低、延迟低。

吞吐低的原因：卡间通信量比较大。

延迟低的原因：每个请求串行执行，同一时刻多张卡同时推理一个请求，TTFT延迟会低一些。

### attentionDP+moeEP高吞吐模式，特点吞吐高、延迟高，适用节点间。原因和PP并行类似。

### attentionDP+moeTP，特点吞吐高、延迟高，适用节点内。

蚂蚁项目中，3s延迟要求相对宽松，通过测试对比选择PP并行方案，PP4TP2。

## 2、测出prefill最大吞吐：

通过chunk-size、max-running-request、--pp-max-micro-batch-size，限定一次prefill推理多少token，token数越大、吞吐越高，但是对应的延迟越高。

启动pd分离服务，因为这一步主要关注prefill的吞吐，decode并行方案随意。（需要用pd分离来测，ifb测的不准确）

请求数用256，请求率用inf，来测试出prefill的最大吞吐，避免decode对prefill的吞吐有影响，decode输出长度设置成1。

测试结果会输出QPS。可以作为Prefill的最大吞吐。

蚂蚁项目中，按照这种方式，ling-1t Prefill单节点吞吐约等于 3.4QPS。

# 二、计算D节点的最大吞吐：

## 1、并行方案选择：

### PP并行，特点跨节点降低显存占用，显存充足时不做优先选择。

### TP并行，特点吞吐低、延迟低。

### attentionDP+moeEP/TP低延迟模式，吞吐上限高，延迟下限高。

<image token="TXYabaHSYoq6f4xQgONctRCVn4e" width="1083" height="539" align="center"/>

蚂蚁项目中TPOT延迟要求100ms，因此选择EP16DP16并行方式。（decode的时候batch与吞吐线性相关）

ling1t模型没有使用MTP，因此延迟较高，如果延迟50ms要求，可以使用MTP也可以考虑大EP方案。

## 2、计算出decode最大吞吐：

启动PD分离服务，p和d节点的吞吐相互影响符合木桶效应，因为要获取D节点的最大吞吐，不能使得P成为系统瓶颈，这时P可以打开radix-cache（prefix-cache），用generated-shared-prefix数据集控制radix-cache命中率，消除P可能成为瓶颈的木桶效应。用相对较高的请求率下发请求，逐步增大decode的batch。

根据decode服务上的日志，判断满足TPOT的最大batch-size。

判断方法：例如tpot要求100ms。那么对于单个请求每秒钟生成token数量就是>= 1s/100ms，10tokens/s。因为随着batch的增加，平均到单个请求的吞吐会减小，所以需要关注服务端日志，看正好满足running_request * 10 tokens/s = token throughput的 batch大小。

<image token="IAaJbGmpwoRAt3x0k98cr13xnUd" width="1656" height="856" align="center"/>

确定好batch-size以后，decode的最大QPS就是 throughtput / output len。

蚂蚁项目 ling-1t的QPS就是  280token/s/rank *16 ranks / output len = 280* 16 / 1536 = 2.91 qps

# 三、P D配比

P和D的QPS都确定以后，就可以计算PD配比了。

目标是：P实例数 *P的QPS = D实例数* D的QPS。

因为无法做到特别理想，大致相等即可。 现在是P（pp4tp2） QPS 3.4 ，D（ep16dp16） QPS 2.91，大致相等。

采用1P（单节点）1D（两节点）的方式。

# 四、跑数据

按照配比和对应的并行方式启动服务后。

就可以跑bench_serving了。 按照木桶原理，QPS选择P或者D中较低的那个作为请求率，可略微上浮探寻最大请求率。

注意事项：

跑PD分离的时候，decode的并发数并不是一下子上来的，而是需要P节点一个请求一个请求打上来的。因此分为三个阶段：

1、在初始阶段decode节点有一个batch逐渐增加的过程。

2、平稳阶段，P节点仍然按照请求率往D节点输出请求，D节点按照请求率完成请求，整体batch大小不变。

3、结束阶段，P节点处理完所有的请求，D节点的batch逐步下降。

只有第二阶段的时候系统处于满载状态。满载状态的QPS=上述结算的理想QPS=请求率。

第1、3阶段，系统处于非满载阶段。实际QPS < 理想QPS。

因此想要测试结果的QPS尽可能高，要提高第2阶段的持续时长。

设置较高的num_promt可以是的第2阶段的占比更高，测试结果趋近于理想QPS。

蚂蚁项目采用num_promt=10000的请求数，最终测试QPS = 2.85 趋近于理想QPS 2.91，tpot p99正好100ms 满足客户SLA要求的同时测出最大吞吐。

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/并发|并发]]（metric）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/SLA|SLA]]（metric）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）

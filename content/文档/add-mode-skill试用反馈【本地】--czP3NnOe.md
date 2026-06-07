---
title: add-mode-skill试用反馈【本地】
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Channel-FP
  - deepseek-r1-channel-int8
  - deepseek-r1.md
  - GLM-5-Channel-INT4-w4a8
  - Llama-3.1-70B-Instruct
  - Qwen3-235B-A22B-Instruct-2507
  - Qwen3-4B-Thinking-2507
framework: []
issue_type: []
source: feishu
feishu_token: W49hwIOMViKzR6kiQOIczP3NnOe
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/W49hwIOMViKzR6kiQOIczP3NnOe
last_synced_at: 2026-06-07T17:07:53.393Z
---

# add-mode-skill试用反馈【本地】

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/W49hwIOMViKzR6kiQOIczP3NnOe)

# 整体体验：

配置好后应用较为方便

# 问题反馈

## skill的问题1让填写模型链接，咱们自己量化的模型存在未上传的现象，导致没办法输入准确的地址（现有1种解决方案就是输入一个错的，后续让大模型自己预测一个，然后人工手动删除）
```plaintext
1.模型卡：模型在 ModelScope 或 HuggingFace 上的完整路径或 URL。
   例如：hygon/GLM-5-Channel-INT4-w4a8、LLM-Research/Meta-Llama-3.1-70B-Instruct
```

<image token="VIG0bgTaToArkExGtpxcTM7unqc" width="1283" height="601" align="center"/>

<image token="BFevbxwN7otKe0xNqBecLwHynvT" width="1365" height="830" align="center"/>

## 现有最佳实践的模型硬件顺序大都是先写bw1100、bw1000、K100-AI，建议第四个问题的skill可以调整下顺序，不然会存在，大模型生成的顺序是先K100-AI的现象（此问题）

<image token="KESgbw0AvoI4UMxyQBwcDPIFnrb" width="1229" height="801" align="center"/>

<image token="Jj3BbD1jMopC9hxG5pmc7qYEnWf" width="1083" height="732" align="center"/>

## 之前汇报的可能一个模型会生成一个单独的md文件的现象，本次尝试的三个模型未出现此现象，只有一个本来有的模型** Qwen3-235B-A22B-Instruct-2507，在修改原本内容的基础上，又生成了一个md文件**        

          DeepSeek-R1-Channel-FP、<text underline="true">deepseek-r1-channel-int8都写在了deepseek-r1.md</text>
          Qwen3-4B-Thinking-2507写在了Qwen3.md

## 上传脚本容易出现网络断开现象（此方法暂时没成功过）

<image token="JhljbP060o5z2Kxf1ydcbsFXn7c" width="1289" height="346" align="center"/>

<image token="UvHPb0Q0yoZQgDxkL97cbXsBnEe" width="1136" height="153" align="center"/>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/DeepSeek-R1-Channel-FP|DeepSeek-R1-Channel-FP]]（model）
- [[飞书知识图谱/实体/deepseek-r1-channel-int8|deepseek-r1-channel-int8]]（model）
- [[飞书知识图谱/实体/deepseek-r1.md|deepseek-r1.md]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-Channel-INT4-w4a8|GLM-5-Channel-INT4-w4a8]]（model）
- [[飞书知识图谱/实体/Llama-3.1-70B-Instruct|Llama-3.1-70B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen3-235B-A22B-Instruct-2507|Qwen3-235B-A22B-Instruct-2507]]（model）
- [[飞书知识图谱/实体/Qwen3-4B-Thinking-2507|Qwen3-4B-Thinking-2507]]（model）
- [[飞书知识图谱/实体/TP|TP]]（concept）

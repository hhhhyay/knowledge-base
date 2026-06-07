---
title: vllm0.15.1测试报告
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework:
  - OpenCompass
  - vLLM
issue_type: []
source: feishu
feishu_token: Lgeyw7snqiUtNqkgZs2c1SCbnnd
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/Lgeyw7snqiUtNqkgZs2c1SCbnnd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# vllm0.15.1测试报告

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/Lgeyw7snqiUtNqkgZs2c1SCbnnd)

# 测试概要

## 测试目的

针对vllm0.15.1进行发版相关测试，用于评估该软件版本及质量是否达标。

## 前置条件

        配置以下环境
   表1 硬件信息

<lark-table rows="5" cols="5" header-row="true" column-widths="138,138,214,292,219">

  <lark-tr>
    <lark-td>
      序号
    </lark-td>
    <lark-td>
      硬件类别
    </lark-td>
    <lark-td>
      型号
    </lark-td>
    <lark-td>
      配置
    </lark-td>
    <lark-td>
      驱动&vbios
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      加速卡型号
    </lark-td>
    <lark-td>
      详细指标
    </lark-td>
    <lark-td>
      机头
    </lark-td>
    <lark-td>
      ip
    </lark-td>
    <lark-td>
      驱动版本
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW1100（nmz）
    </lark-td>
    <lark-td>
      GC频率：1350Mhz
      HBM频率：875Mhz
      CU数：64/72
      功耗：800W
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8468V
      INTEL(R) XEON(R) PLATINUM 8558P
    </lark-td>
    <lark-td>
      10.16.1.7（Intel(R) Xeon(R) Platinum 8468V）
      10.16.1.4（INTEL(R) XEON(R) PLATINUM 8558P）
    </lark-td>
    <lark-td>
      6.3.28-V1.3.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW1000
    </lark-td>
    <lark-td>
      GC频率：1500Mhz
      HBM频: 1800Mhz
      CU数：80
      功耗：800W
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8468V
      INTEL(R) XEON(R) PLATINUM 8558P
    </lark-td>
    <lark-td>
      10.16.1.1 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.1.42 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.1.43 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.1.44 (INTEL(R) XEON(R) PLATINUM 8558P)
    </lark-td>
    <lark-td>
      6.3.28-V1.3.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      K100_AI
    </lark-td>
    <lark-td>
      GC频率：1400Mhz
      HBM频率：875Mhz
      CU数：120
      功耗：400w
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      10.16.1.81 Intel(R) Xeon(R) Platinum 8458P
      10.16.1.83 Intel(R) Xeon(R) Platinum 8458P
      10.16.1.84 Intel(R) Xeon(R) Platinum 8458P
      10.16.1.85 Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      6.3.28-V1.3.0b
    </lark-td>
  </lark-tr>
</lark-table>

表2 软件信息

<lark-table rows="5" cols="4" header-row="true" column-widths="57,111,155,845">

  <lark-tr>
    <lark-td>
      序号
    </lark-td>
    <lark-td>
      软件类别
    </lark-td>
    <lark-td>
      版本
    </lark-td>
    <lark-td>
      备注
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      1
    </lark-td>
    <lark-td>
      操作系统
    </lark-td>
    <lark-td>
      ubuntu22.04
    </lark-td>
    <lark-td>
      实际覆盖物理机见表1
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      2
    </lark-td>
    <lark-td>
      驱动版本
    </lark-td>
    <lark-td>
      6.3.28-V1.3.0b
    </lark-td>
    <lark-td>
      VBIOS version:6.312.002400Q.984889
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      3
    </lark-td>
    <lark-td>
      运行时
    </lark-td>
    <lark-td>
      DTK-26.04-rc4
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      10.16.1.152:5000/jenkins/model_test_env/vllm:0.15.1-ubuntu22.04-dtk26.04-py3.10-20260515-1239
    </lark-td>
  </lark-tr>
</lark-table>

## 测试要求

表3 详细测试要求

<lark-table rows="4" cols="3" header-row="true" column-widths="72,117,796">

  <lark-tr>
    <lark-td>
      序号
    </lark-td>
    <lark-td>
      类别
    </lark-td>
    <lark-td>
      要求
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      1
    </lark-td>
    <lark-td>
      qwen系列模型
    </lark-td>
    <lark-td>
      （1）推理框架：vllm0.15.1
      （2）测试精度：bf16、int8
      （3）测试工具：opencompass（0.4.2）
      （4）数据集：gsm8k(134f0e)、math-500(11c4b5)、human-eval(dcae0e)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      2
    </lark-td>
    <lark-td>
      deepseek系列模型
    </lark-td>
    <lark-td>
      （1）推理框架：vllm0.15.1
      （2）测试精度：bf16、fp8、int8、int4
      （3）测试工具：opencompass（0.4.2）
      （4）数据集：gsm8k(134f0e)、math-500(11c4b5)、human-eval(dcae0e)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      3
    </lark-td>
    <lark-td>
      其他模型（glm、kimi、minimax）
    </lark-td>
    <lark-td>
      （1）推理框架：vllm0.15.1
      （2）测试精度：bf16、fp8、int8、int4
      （3）测试工具：opencompass（0.4.2）
      （4）数据集：gsm8k(134f0e)、math-500(11c4b5)、human-eval(dcae0e)
    </lark-td>
  </lark-tr>
</lark-table>

# 测试执行

<lark-table rows="4" cols="2" header-row="true" column-widths="155,801">

  <lark-tr>
    <lark-td>
      模型
    </lark-td>
    <lark-td>
      测试命令
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      qwen系列模型
    </lark-td>
    <lark-td>
      1、参考最佳实践进行测试【 [https://github.com/HYGON-AI/dcu-inference-cookbook](https%3A%2F%2Fgithub.com%2FHYGON-AI%2Fdcu-inference-cookbook) 】
      2、无最佳实践的其余模型参考之前的测试文档【[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]]】
      详见：[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cpCcgnOc|VLLM测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      deepseek系列模型
    </lark-td>
    <lark-td>
      1、参考最佳实践进行测试【 [https://github.com/HYGON-AI/dcu-inference-cookbook](https%3A%2F%2Fgithub.com%2FHYGON-AI%2Fdcu-inference-cookbook) 】
      2、无最佳实践的其余模型参考之前的测试文档【[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]]】
      详见：[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cpCcgnOc|VLLM测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      其他模型（glm、kimi、minimax）
    </lark-td>
    <lark-td>
      1、参考最佳实践进行测试【 [https://github.com/HYGON-AI/dcu-inference-cookbook](https%3A%2F%2Fgithub.com%2FHYGON-AI%2Fdcu-inference-cookbook) 】
      2、无最佳实践的其余模型参考之前的测试文档【[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]]】
      详见：[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cpCcgnOc|VLLM测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
</lark-table>

精度展示详表：[[飞书知识图谱/资源/20260515镜像精度测试（vllm）--crTMznvd|20260515镜像精度测试（vllm）]]

# 测试总结

测试模型共计53个，其中deepseek系列模型10个，qwen系列模型22个，GLM系列模型

以下分别为NMZ、BMZ、KME平台模型验证结果，<text color="red">标红</text>部分为异常模型。

遗留问题详见：

<!-- Unsupported block type: 53 -->

## 关联资源

- [[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/KwaLdxOLmoFnysxWuMqcJOhOnug)
- [[飞书知识图谱/文档/VLLM测试指导（QTeam）--cpCcgnOc|VLLM测试指导（QTeam）]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/M42vwS4kQiqfKGkHAgacpCcgnOc)
- [[飞书知识图谱/资源/20260515镜像精度测试（vllm）--crTMznvd|20260515镜像精度测试（vllm）]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/sheets/S8uwsHdfnhYHXDtdIPucrTMznvd)

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/OpenCompass|OpenCompass]]（framework）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

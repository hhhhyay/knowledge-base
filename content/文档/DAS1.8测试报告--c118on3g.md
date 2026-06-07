---
title: DAS1.8测试报告
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1
  - DeepSeek-R1-Channel-INT8
  - DeepSeek-R1-Distill-Llama-70B
  - DeepSeek-R1-Distill-Llama-70B-main
  - DeepSeek-V3-W4A8-V2-tx
  - DeepSeek-V3.2-bf16
  - DeepSeek-V3.2-Channel-INT8
  - llama2-13b
  - Llama2-13b
  - llama2-7b
  - Llama2-7b
  - qwen1.5-14b
  - Qwen1.5-14b
  - qwen1.5-32b
  - Qwen1.5-32b
  - qwen2.5-VL-72B
  - Qwen2.5-VL-72B-Instruct
  - Qwen2.5-VL-72B-Instruct-quantized.w8a8
  - Qwen3-235B-A22B
  - Qwen3-30B
  - Qwen3-30B-A3B
  - Qwen3-32B
  - Qwen3-32B-fp16
  - Qwen3-8B
  - qwen3-next
  - qwen3-VL-32B
  - Qwen3-VL-32B-Instruct
framework:
  - Megatron-LM
  - OpenCompass
  - PyTorch
  - SGLang
  - vLLM
issue_type: []
source: feishu
feishu_token: W6UzwQJ7siGONwk2XHdc118on3g
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/W6UzwQJ7siGONwk2XHdc118on3g
last_synced_at: 2026-06-07T17:07:53.393Z
---

# DAS1.8测试报告

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/W6UzwQJ7siGONwk2XHdc118on3g)

# 单测篇 <mention-user id="ou_4839a2ca5dcee752abf76b1cadfd22ae"/>

## 测试概要

### 测试目的

针对DAS1.8+DTK26.04进行DAS框架组件单元测试，评估本次DAS框架组件是否满足发版要求。

### 前置条件

  配置以下环境
        配置以下环境
   表1 硬件信息

<lark-table rows="9" cols="5" header-row="true" column-widths="138,138,214,292,219">

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
      CU数：64
      功耗：800W
    </lark-td>
    <lark-td>
      INTEL(R) XEON(R) PLATINUM 8558P
    </lark-td>
    <lark-td>
      10.16.6.62（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.63（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.66（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.67（INTEL(R) XEON(R) PLATINUM 8558P）
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW1101（nmz）
    </lark-td>
    <lark-td>
      GC频率：1300Mhz
      HBM频: 875Mhz
      CU数：72
      功耗：800W
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      10.16.6.20 Intel(R) Xeon(R) Platinum 8468V
      10.16.6.22 Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      10.16.6.11 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.23 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.61 (INTEL(R) XEON(R) PLATINUM 8558P)
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW100
    </lark-td>
    <lark-td>
      GC频率：1220Mhz
      HBM频率：1800Mhz
      CU数：64
      功耗：400w
    </lark-td>
    <lark-td>
      Hygon C86 7490
    </lark-td>
    <lark-td>
      10.16.5.4 Hygon C86 7490
      10.16.5.5 Hygon C86 7490
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW150
    </lark-td>
    <lark-td>
      GC频率：1400Mhz
      HBM频率：1800Mhz
      CU数：80
      功耗：600w
    </lark-td>
    <lark-td>
      Hygon C86 7490
    </lark-td>
    <lark-td>
      10.16.5.4 Hygon C86 7490
      10.16.5.5 Hygon C86 7490
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW10
    </lark-td>
    <lark-td>
      GC频率：760Mhz
      HBM频率：1800Mhz
      CU数：48
      功耗：150w
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      10.16.4.7 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.11 Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      10.16.4.5 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.12 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.13 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.14 Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
</lark-table>

表2 软件信息

<lark-table rows="6" cols="4" header-row="true" column-widths="57,111,155,845">

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
      6.3.29-V1.4.0b
    </lark-td>
    <lark-td>
      VBIOS version: 6.314.0034001.990874
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
      DTK26.04-rc1
    </lark-td>
    <lark-td>
      终版实际应该是rc1版
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      软件栈
    </lark-td>
    <lark-td>
      DAS1.8
    </lark-td>
    <lark-td>
      以镜像为准
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      [http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.9.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fvllm0.9.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fvllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/tensorflow2.18.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Ftensorflow2.18.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/tensorflow2.13.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Ftensorflow2.13.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/sglang0.5.7-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fsglang0.5.7-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/pytorch%3A2.7.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fpytorch%253A2.7.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/pytorch2.9.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fpytorch2.9.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/pytorch2.5.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fpytorch2.5.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/migraphx5.1.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fmigraphx5.1.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
    </lark-td>
  </lark-tr>
</lark-table>

### 测试要求

表3 测试框架组件

<sheet token="Nd9Csh0xchDvEOtT17Cciey1ni6_R88W2e"/>

基于上述框架组件信息，基于DTK-26.04进行各个框架组件的构建测试、单元测试和部分性能测试。

## 测试执行

单测情况展示详表：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]

# 模型篇（推理-vllm）<mention-user id="ou_94bfd752f8da359c2fa5287fd6c59d62"/>

## 测试概要

### 测试目的

针对DAS1.8+DTK26.04进行发版相关测试，用于评估该软件版本集质量是否达标。

### 前置条件

        配置以下环境
   表1 硬件信息

<lark-table rows="9" cols="5" header-row="true" column-widths="138,138,214,292,219">

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
      CU数：64
      功耗：800W
    </lark-td>
    <lark-td>
      INTEL(R) XEON(R) PLATINUM 8558P
    </lark-td>
    <lark-td>
      10.16.6.62（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.63（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.66（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.67（INTEL(R) XEON(R) PLATINUM 8558P）
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW1101（nmz）
    </lark-td>
    <lark-td>
      GC频率：1300Mhz
      HBM频: 875Mhz
      CU数：72
      功耗：800W
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      10.16.6.20 Intel(R) Xeon(R) Platinum 8468V
      10.16.6.22 Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      10.16.6.11 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.23 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.61 (INTEL(R) XEON(R) PLATINUM 8558P)
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW100
    </lark-td>
    <lark-td>
      GC频率：1220Mhz
      HBM频率：1800Mhz
      CU数：64
      功耗：400w
    </lark-td>
    <lark-td>
      Hygon C86 7490
    </lark-td>
    <lark-td>
      10.16.5.4 Hygon C86 7490
      10.16.5.5 Hygon C86 7490
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW150
    </lark-td>
    <lark-td>
      GC频率：1400Mhz
      HBM频率：1800Mhz
      CU数：80
      功耗：600w
    </lark-td>
    <lark-td>
      Hygon C86 7490
    </lark-td>
    <lark-td>
      10.16.5.4 Hygon C86 7490
      10.16.5.5 Hygon C86 7490
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW10
    </lark-td>
    <lark-td>
      GC频率：760Mhz
      HBM频率：1800Mhz
      CU数：48
      功耗：150w
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      10.16.4.7 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.11 Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      10.16.4.5 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.12 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.13 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.14 Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
</lark-table>

表2 软件信息

<lark-table rows="6" cols="4" header-row="true" column-widths="57,111,155,845">

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
      6.3.29-V1.4.0b
    </lark-td>
    <lark-td>
      VBIOS version:6.314.0034001.990874
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
      DTK26.04-rc1
    </lark-td>
    <lark-td>
      终版实际应该是rc1版
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      软件栈
    </lark-td>
    <lark-td>
      DAS1.8
    </lark-td>
    <lark-td>
      以镜像为准
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      /public/opendas/ArchivedFile/docker-images/das18-20260305/migraphx5.1.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
      /public/opendas/ArchivedFile/docker-images/das18-20260305/pytorch2.5.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
      /public/opendas/ArchivedFile/docker-images/das18-20260305/pytorch:2.7.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
      /public/opendas/ArchivedFile/docker-images/das18-20260305/pytorch2.9.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz
      /public/opendas/ArchivedFile/docker-images/das18-20260305/tensorflow2.13.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz
      /public/opendas/ArchivedFile/docker-images/das18-20260305/tensorflow2.18.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260304.tar.gz
      /public/opendas/ArchivedFile/docker-images/das18-20260305/vllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
      /public/opendas/ArchivedFile/docker-images/das18-20260305/vllm0.9.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
    </lark-td>
  </lark-tr>
</lark-table>

### 测试要求

表3 详细测试要求

<lark-table rows="10" cols="3" header-row="true" column-widths="72,117,953">

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
      Qwen3-8B
    </lark-td>
    <lark-td>
      （1）P0：nopc-nocp；P1：开pc
              P0：BW11001、BW1100、BW1000、BW150、K100_AI、H20（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：fp16、int8
      （4）测试参数：
      server：model-len 40960   v1+cudagraph+zero+nopc+nocp（vllm092需要zero,vllm0.11不需要zero）
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      2
    </lark-td>
    <lark-td>
      Qwen3-30B-A3B
    </lark-td>
    <lark-td>
      （1）P0：nopc-nocp；P1：开pc
              P0：BW11001、BW1100、BW1000、BW150、K100_AI、H20（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：fp16、int8
      （4）测试参数：
      server：model-len 40960   v1+cudagraph+zero+nopc+nocp（vllm092需要zero,vllm0.11不需要zero）
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      3
    </lark-td>
    <lark-td>
      Qwen3-235B-A22B
    </lark-td>
    <lark-td>
      （1）P0：nopc-nocp；P1：开pc
              P0：BW11001、BW1100、BW1000、BW150、K100_AI、H20（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：fp16、int8
      （4）测试参数：
      server：model-len 40960   v1+cudagraph+zero+nopc+nocp（vllm092需要zero,vllm0.11不需要zero）
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      Qwen3-32B
    </lark-td>
    <lark-td>
      （1）P0：nopc-nocp；P1：开pc
              P0：BW11001、BW1100、BW1000、BW150、K100_AI、H20（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：fp16、int8
      （4）测试参数：
      server：model-len 40960   v1+cudagraph+zero+nopc+nocp（vllm092需要zero,vllm0.11不需要zero）
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      DeepSeek-R1-Distill-Llama-70B
    </lark-td>
    <lark-td>
      （1）P0：nopc-nocp；P1：开pc
              P0：BW11001、BW1100、BW1000、BW150、K100_AI、H20（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：fp16、int8
      （4）测试参数：
      server：model-len 40960   v1+cudagraph+zero+nopc+nocp（vllm092需要zero,vllm0.11不需要zero）
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      6
    </lark-td>
    <lark-td>
      DeepSeek-R1
    </lark-td>
    <lark-td>
      （1）P0：nopc-nocp；P1：开pc
              P0：BW1000、BW150、K100_AI、H20（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：bf16、int8、w4a8
      （4）测试参数：
      server：看情况
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      7
    </lark-td>
    <lark-td>
      qwen2.5-VL-72B
    </lark-td>
    <lark-td>
      （1）P0：nopc-nocp；P1：开pc
              P0：BW11001、BW1100、BW1000、BW150、K100_AI、H20（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：fp16
      （4）测试参数：
      server：model-len 40960   v1+cudagraph+zero+nopc+nocp（vllm092需要zero,vllm0.11不需要zero）
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      8
    </lark-td>
    <lark-td>
      stable-diffusion
    </lark-td>
    <lark-td>
      （1）开compile
      （2）测试发版镜像：
      （3）测试精度：fp16
      （4）测试参数：
      width*height(512*512)(1024*1024)(2048*2048)
      steps(10,20,30,40,50)
      batch_size:(1,2)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      9
    </lark-td>
    <lark-td>
      resnet50
    </lark-td>
    <lark-td>
      （1）测试发版镜像：
      （2）测试精度：fp16、int8
      （3）测试参数：
      batch_size:(1,2,4,8,16,32,48,64,128,256,512,1024)
      (4)环境变量设置：
      fp16：export MIGRAPHX_ENABLE_FP16_FUSION=1
               export MIGRAPHX_ENABLE_MIOPEN_CONCAT=1
      int8： export MIGRAPHX_ENABLE_INT8_FUSION=1
    </lark-td>
  </lark-tr>
</lark-table>

## 测试执行

### 大语言问答模型

<lark-table rows="23" cols="3" column-widths="120,100,604">

  <lark-tr>
    <lark-td>
      模型
    </lark-td>
    <lark-td>
      精度
    </lark-td>
    <lark-td>
      测试命令
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      Qwen3-8B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve $model_path --host 0.0.0.0 --port $port \
          --dtype float16  \
          --tensor-parallel-size $tp \
          --max-num-seqs 1024 \
          --distributed-executor-backend=mp \
          --disable-cascade-attn \
          --no-enable-prefix-caching \
          --no-enable-chunked-prefill \
          --kv-cache-dtype fp8_e4m3 \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      vllm serve $model_path --host 0.0.0.0 --port $port \
          --dtype float16  \
          --tensor-parallel-size $tp \
          --max-num-seqs 1024 \
          --distributed-executor-backend=mp \
          --disable-cascade-attn \
          --no-enable-prefix-caching \
          --no-enable-chunked-prefill \
          --kv-cache-dtype fp8_e4m3 \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      Qwen3-30B-A3B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve $model_path --host 0.0.0.0 --port $port \
          --dtype float16  \
          --tensor-parallel-size $tp \
          --max-num-seqs 1024 \
          --distributed-executor-backend=mp \
          --disable-cascade-attn \
          --no-enable-prefix-caching \
          --no-enable-chunked-prefill \
          --kv-cache-dtype fp8_e4m3 \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      vllm serve $model_path --host 0.0.0.0 --port $port \
          --dtype bfloat16 \
          --tensor-parallel-size $tp \
          --max-num-seqs 1024 \
          --distributed-executor-backend=mp \
          --disable-cascade-attn \
          --no-enable-prefix-caching \
          --no-enable-chunked-prefill \
          --kv-cache-dtype fp8_e4m3 \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      Qwen3-235B-A22B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
       --dtype float16 \
       --host 0.0.0.0 \
       --port ${port} \
       --trust-remote-code \
       -tp 8 \
       --max-model-len 40960 \
       --gpu-memory-utilization $gpu_memory \
      --distributed-executor-backend mp \
       --no-enable-prefix-caching \
       --no-enable-chunked-prefill \
       --disable-cascade-attn \
       --kv-cache-dtype fp8_e4m3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
       --dtype bfloat16 \
       --host 0.0.0.0 \
       --port ${port} \
       --trust-remote-code \
       -tp 8 \
       --max-model-len 40960 \
       --gpu-memory-utilization $gpu_memory \
      --distributed-executor-backend mp \
       --no-enable-prefix-caching \
       --no-enable-chunked-prefill \
       --disable-cascade-attn \
       --kv-cache-dtype fp8_e4m3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      Qwen3-32B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve $model_path \
              --host 0.0.0.0 \
              --port $port   \
              --dtype  float16 \
              --tensor-parallel-size $tp   \
              --trust-remote-code  \
              --distributed-executor-backend=mp \
              --disable-cascade-attn \
              --no-enable-prefix-caching \
              --no-enable-chunked-prefill \
               --max-num-seqs 1024 \
               --kv-cache-dtype fp8_e4m3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      vllm serve $model_path \
              --host 0.0.0.0 \
              --port $port   \
              --dtype  float16 \
              --tensor-parallel-size $tp   \
              --trust-remote-code  \
              --distributed-executor-backend=mp \
              --disable-cascade-attn \
              --max-num-seqs 1024 \
              --no-enable-prefix-caching \
              --no-enable-chunked-prefill \
              --kv-cache-dtype fp8_e4m3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      DeepSeek-R1-Distill-Llama-70B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
       --dtype ${data_type} \
       --host 0.0.0.0 \
       --port ${port} \
       --trust-remote-code \
       --max_model_len 40960 \
       --max-num-seqs 1024 \
       -tp ${tp} \
       --kv-cache-dtype fp8_e4m3  \
       --distributed-executor-backend mp \
       --gpu-memory-utilization $gpu_memory \
       --no-enable-prefix-caching \
       --no-enable-chunked-prefill \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
       --dtype ${data_type} \
       --host 0.0.0.0 \
       --port ${port} \
       --trust-remote-code \
       --max_model_len 40960 \
       --max-num-seqs 1024 \
       -tp ${tp} \
       --kv-cache-dtype fp8_e4m3  \
       --distributed-executor-backend mp \
       --gpu-memory-utilization $gpu_memory \
       --no-enable-prefix-caching \
       --no-enable-chunked-prefill \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="3">
      DeepSeek-R1
    </lark-td>
    <lark-td>
      bf16
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \  
      --trust-remote-code \  
      --distributed-executor-backend ray \  
      --dtype $data_type \  
      --tensor-parallel-size $tp \  
      --gpu-memory-utilization $gpu_memory \  
      --host [0.0.0.0](http%3A%2F%2F0.0.0.0) \ 
       --port $port \  
      --kv-cache-dtype fp8_e4m3  \
      --max-model-len 40960 \  
      --max-seq-len-to-capture 40960 \  
      --max-num-batched-tokens 40960 \  
      --disable-log-requests \  
      --block-size 64 \  
      --speculative_config '{"num_speculative_tokens": 3}'  \ 
       --enable-chunked-prefill \  
      --no-enable-prefix-caching \  
      --disable-cascade-attn
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      （bw1000）
      vllm serve  $model_path \
      --trust-remote-code \
      --distributed-executor-backend ray \
      --dtype bfloat16 \
      --max-model-len 40960 \
      -q slimquant_marlin \
      --max-num-batched-tokens 40960  \
      --max-seq-len-to-capture 40960  \
      -tp 16 \
      --gpu-memory-utilization 0.90 \
      --max-num-seqs 1024 \
      --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 3}' \
      --disable-log-requests \
      --block-size 64 \
      --no-enable-chunked-prefill \
      --no-enable-prefix-caching  \
      --kv-cache-dtype fp8_e5m2 \
      --enforce-eager
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      w4a8
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
       --trust-remote-code  --dtype bfloat16 --max-model-len 40960 -q slimquant_w4a8_marlin  --max-seq-len-to-capture 40960 -tp ${tp} --gpu-memory-utilization 0.9 --max-num-seqs 256 --max-num-batched-tokens 40960 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 3}' --block-size 64 --no-enable-chunked-prefill  --no-enable-prefix-caching --disable-log-requests --kv-cache-dtype fp8_e4m3  --disable-cascade-attn
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      qwen2.5-VL-72B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve $model_path \
              --host 0.0.0.0 \
              --port $port   \
              --dtype  float16 \
              --tensor-parallel-size $tp   \
              --trust-remote-code  \
              --distributed-executor-backend=mp \
              --disable-cascade-attn \
              --max-num-seqs 1024   \
              --max_model_len 40960 \
              --no-enable-prefix-caching  \
              --no-enable-chunked-prefill  \
              --limit-mm-per-prompt '{"image": 255,"video": 1}' \
              --kv-cache-dtype fp8_e4m3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      qwen3-VL-32B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      服务：
      vllm serve $model_path \
              --host 0.0.0.0 \
              --port $port   \
              --dtype float16 \
              --tensor-parallel-size $tp   \
              --trust-remote-code  \
             --disable-cascade-attn \
              --max-num-seqs 1024   \
              --max_model_len 40960 \
              --max-seq-len-to-capture 40960    \
              --distributed-executor-backend=mp \
              --no-enable-prefix-caching  \
              --no-enable-chunked-prefill  \
              --limit-mm-per-prompt '{"image": 255，"video": 1}' 
      客户端：
      vllm bench serve \
              --host [0.0.0.0](http%3A%2F%2F0.0.0.0) \
              --backend openai-chat \
              --endpoint /v1/chat/completions \
              --model ${model_path} \
              --random-input-len $prompt_tokens \
              --random-output-len $completion_tokens \
              --port $port \
              --metric-percentiles 95,99 \
              --num-prompts $batch \
              --dataset-name random-mm \
              --max-concurrency $batch \
              --ignore-eos \
              --random-mm-base-items-per-request 1 \
              --random-mm-bucket-config " {(256, 256, 1): 0.5, (720, 1280, 1): 0.5, (720, 1280, 16): 0.0}"  \
              --random-mm-limit-mm-per-prompt '{"image": 255, "video": 1}' \
              --random-mm-num-mm-items-range-ratio 0.0 \
              --seed 42 \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      DeepSeek-V3-W4A8-V2-tx
    </lark-td>
    <lark-td>
      W4A8
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
       --port ${port} \
       --trust-remote-code \
       --dtype bfloat16 \
       --max-model-len 40960 \
       --max-num-seq 512 \
       --max-seq-len-to-capture 40960 \
       -tp ${tp} \
       --gpu-memory-utilization 0.9 \
       --max-num-batched-tokens 40960 \
       --no-enable-prefix-caching \
       --no-enable-chunked-prefill \
       --disable-cascade-attn \
       --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 3}' \
       -q slimquant_w4a8_marlin \
       --kv-cache-dtype fp8_e4m3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      DeepSeek-V3.2-bf16
    </lark-td>
    <lark-td>
      bf16
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
       --trust-remote-code \
       --distributed-executor-backend ray \
       --dtype bfloat16 \
       --tensor-parallel-size 16 \
       --gpu-memory-utilization 0.9 \
       --host 0.0.0.0 \
       --port $port \
       --max-model-len 40960 \
       --max-num-batched-tokens 40960 \
       --disable-log-requests \
       --block-size 64 \
       --speculative_config '{"num_speculative_tokens": 3}'  \
       --enable-chunked-prefill \
       --no-enable-prefix-caching \
       --disable-cascade-attn \
       --kv-cache-dtype fp8_e4m3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      DeepSeek-V3.2-Channel-INT8
    </lark-td>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      vllm serve /model/vllm-w8a8-models/DeepSeek-R1-Channel-INT8 \
        --trust-remote-code \
        --host 0.0.0.0 \
        --port 9196 \
        --distributed-executor-backend ray \
        --dtype bfloat16 \
        --max-model-len 32768 \
        --max-seq-len-to-capture 32768 \
        -tp 16 \
        --gpu-memory-utilization 0.95\
        --max-num-seqs 1024 \
        --disable-log-requests \
        --block-size 64 \
        --no-enable-chunked-prefill \
        --max-num-batched-tokens 32768 \
        --no-enable-prefix-caching \
        --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 3}' \
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      Qwen2.5-VL-72B-Instruct
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve $model_path \
              --host 0.0.0.0 \
              --port $port   \
              --dtype  float16 \
              --tensor-parallel-size $tp   \
              --trust-remote-code  \
              --distributed-executor-backend=mp \
              --disable-cascade-attn \
              --max-num-seqs 1024   \
              --max_model_len 40960 \
              --no-enable-prefix-caching  \
              --no-enable-chunked-prefill  \
              --limit-mm-per-prompt '{"image": 255,"video": 1}' \
              --kv-cache-dtype fp8_e4m3 \
              --gpu-memory-utilization 0.9
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      Qwen2.5-VL-72B-Instruct-quantized.w8a8
    </lark-td>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
          --host 0.0.0.0 --port $port \
          --dtype bfloat16 --tensor-parallel-size $tp \
          --trust-remote-code \
          --no-enable-prefix-caching \
          --no-enable-chunked-prefill \
          --max-num-seqs 1024 \
          --disable-cascade-attn --kv-cache-dtype fp8_e4m3 \
          --max-model-len 40960 \
          --limit-mm-per-prompt '{"image": 255, "video": 1}'
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      Qwen3-VL-32B-Instruct
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      vllm serve ${model_path} \
          --gpu-memory-utilization 0.85 \
          --host 0.0.0.0 --port $port --dtype float16 --tensor-parallel-size $tp \
          --trust-remote-code \
          --no-enable-prefix-caching --no-enable-chunked-prefill \
          --disable-cascade-attn \
          --max-model-len 40960 \
          --limit-mm-per-prompt '{"image": 1, "video": 1}'
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
</lark-table>

### 多模态模型

<lark-table rows="3" cols="3" column-widths="120,100,624">

  <lark-tr>
    <lark-td>
      模型
    </lark-td>
    <lark-td>
      精度
    </lark-td>
    <lark-td>
      测试命令
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      Stable Diffusion XL
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      pip install pycountry diffusers==0.33
      HIP_VISIBLE_DEVICES=2 PYTORCH_MIOPEN_SUGGEST_NHWC=1 python run_sdxl.py
      测试脚本见：
      <view type="2">
        <file token="UzOtbCIEPoQkEuxLp15cXl5ynxh" name="run_sdxl.py"/>
      </view>
      NV上torch2.5镜像使用的是：nvcr.io/nvidia/pytorch:24.10-py3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      WAN2.1-14B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      H20:
      pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
      pip install xfuser -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
      DCU:
      pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
      pip install pytorch_fid  lpips scikit-image pycountry xfuser -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
    </lark-td>
  </lark-tr>
</lark-table>

性能展示详表：[[飞书知识图谱/资源/DAS1.8发版模型测试过程数据——数据展示平台--c7dfznIf|DAS1.8发版模型测试过程数据——数据展示平台]]

### 传统模型

<lark-table rows="5" cols="3" column-widths="179,100,576">

  <lark-tr>
    <lark-td>
      模型
    </lark-td>
    <lark-td>
      精度
    </lark-td>
    <lark-td>
      测试命令
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      resnet50
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      /opt/dtk/bin/migraphx-driver perf "$model_path" --fp16
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      /opt/dtk/bin/migraphx-driver perf "$model_path" --int8
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      PaddleOCR-V5-Det
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      migraphx-driver perf --onnx  "$model_path" --fp16
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      PaddleOCR-V5-Rec
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      migraphx-driver perf --onnx  "$model_path"  --fp16
    </lark-td>
  </lark-tr>
</lark-table>

性能展示详表：[[飞书知识图谱/资源/DAS1.8发版模型测试过程数据——数据展示平台--c7dfznIf|DAS1.8发版模型测试过程数据——数据展示平台]]

## 测试总结

- 本次das1.8发版vlm推理模型性能测试，大语言问答模型在满足sla要求(ttft<3s，tpot小于50ms；16k、32k输入ttft<30s)情况下，共计测试27个模型；文生图模型1个；图生视频11个模型；传统模型1个。
- 本次das1.8发版与das1.7的性能对比，见：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]；详细测试数据见：[[飞书知识图谱/资源/DAS1.8发版模型测试过程数据——数据展示平台--c7dfznIf|DAS1.8发版模型测试过程数据——数据展示平台]]
- SLA要求：首字≤3s；生成≤50ms；(其中输入16~32k场景下 首字≤30s)时，Qwen3-8B（tp：1，dtype：fp16）和Qwen3-30B（tp：4(BW10)/2(BMZ)/1(NMZ)，dtype：fp16）不同卡上的每卡可支持路数对比如下图

<image token="Q8aAbabieo4TtLxmFGycBJIFnPe" width="1267" height="771" align="center"/>

- 端到端模型总体性能：
   （1）BW10<K100_AI<BW150<BW1000<H20<BW1100<BW1101。BW10/H20：14.44%；K100_AI/H20：19.10%；BW150/H20：70.19%；BW1000/H20：87.13%；BW1100/H20：106.05%；BW1101/H20：114.82%；
   （2）BW1101总体性能优于BW1100: BW1101/BW1100：104.6%
   （3）NMZ相较于BMZ大size有优势: 在16k/1024，32k/1024大输入场景下NMZ相较于BMZ可支持路数，可提升一倍左右

<image token="LWsSbBhdLoHODxx988XceEh5nmb" width="1112" height="642" align="center"/>

# 模型篇（推理-sglang）<mention-user id="ou_073081244c5b4a58102ebe5ff6a0f012"/>

## 测试概要

### 测试目的

针对DAS1.8+DTK26.04进行发版相关测试，用于评估该软件版本及质量是否达标。

### 前置条件

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
      CU数：64
      功耗：800W
    </lark-td>
    <lark-td>
      INTEL(R) XEON(R) PLATINUM 8558P
    </lark-td>
    <lark-td>
      10.16.6.62（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.63（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.66（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.67（INTEL(R) XEON(R) PLATINUM 8558P）
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW1101（nmz）
    </lark-td>
    <lark-td>
      GC频率：1300Mhz
      HBM频: 875Mhz
      CU数：72
      功耗：800W
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      10.16.6.20 Intel(R) Xeon(R) Platinum 8468V
      10.16.6.22 Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      10.16.6.11 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.23 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.61 (INTEL(R) XEON(R) PLATINUM 8558P)
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
</lark-table>

表2 软件信息

<lark-table rows="6" cols="4" header-row="true" column-widths="172,172,155,643">

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
      实际覆盖物理机见：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]
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
      6.3.29-V1.4.0b
    </lark-td>
    <lark-td>
      VBIOS version: 6.314.0034001.990874
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
      DTK26.04-rc1
    </lark-td>
    <lark-td>
      终版实际应该是rc1版
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      软件栈
    </lark-td>
    <lark-td>
      DAS1.8
    </lark-td>
    <lark-td>
      以镜像为准
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      /public/opendas/ArchivedFile/docker-images/das18-20260305/sglang0.5.7-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
    </lark-td>
  </lark-tr>
</lark-table>

### 测试要求

表3 详细测试要求

<lark-table rows="3" cols="3" header-row="true" column-widths="72,117,953">

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
      Qwen3-32B
    </lark-td>
    <lark-td>
      （1）P0：--disable-radix-cache、--chunked-prefill-size -1；P1：开radix-cache
              P0：BW1000、BW1100、BW1101（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：fp16
      （4）测试参数：
      server：model-len 40960   --disable-radix-cache、--chunked-prefill-size -1
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      2
    </lark-td>
    <lark-td>
      DeepSeek-R1
    </lark-td>
    <lark-td>
      （1）P0：--disable-radix-cache、--chunked-prefill-size -1；P1：开radix-cache
              P0：BW1000、BW1100、BW1101（全并行测试）
      （2）测试发版镜像：
      （3）测试精度：int8
      （4）测试参数：
      server：model-len 40960   --disable-radix-cache、--chunked-prefill-size -1
      client：(512,512)(1024,1024)(4096,1024)(16384,1024)(32768,1024)(4096,4096)(512,4096)
    </lark-td>
  </lark-tr>
</lark-table>

## 测试执行

<lark-table rows="3" cols="3" column-widths="120,100,933">

  <lark-tr>
    <lark-td>
      模型
    </lark-td>
    <lark-td>
      精度
    </lark-td>
    <lark-td>
      测试命令
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      Qwen3-32B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      服务端：
      python3 -m sglang.launch_server --model-path $model_path --numa-node 0 0 0 0 1 1 1 1 --disable-radix-cache --chunked-prefill-size -1 --max-running-requests 512 --context-length 40960  --cuda-graph-max-bs 512    --kv-cache-dtype fp8_e4m3  --host $host_ip  --port $port --trust-remote-code    --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank   --dtype bfloat16  --tp-size 4 --pp-size 1  --mem-fraction-static 0.9 --attention-backend fa3 
      客户端：
              python -m sglang.bench_serving --backend sglang-oai-chat \
              --model ${model_path} \
              --port $port --host 0.0.0.0 \
              --dataset-name random \
              --random-range-ratio 1 \
              --dataset-path ShareGPT_V3_unfiltered_cleaned_split.json \
              --random-input-len $prompt_tokens \
              --random-output-len $completion_tokens \
              --num-prompts $batch --max-concurrency $batch
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      DeepSeek-R1
    </lark-td>
    <lark-td>
      int8
    </lark-td>
    <lark-td>
      服务端：
      python3 -m sglang.launch_server --model-path $model_path --numa-node 0 0 0 0 1 1 1 1 --disable-radix-cache --chunked-prefill-size -1 --max-running-requests 512 --context-length 40960 --speculative-algorithm EAGLE**--speculative-num-steps 3**--speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 512   
       --quantization slimquant_marlin --kv-cache-dtype fp8_e4m3  --host $host_ip  --port $port --trust-remote-code    --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank   --dtype bfloat16  --tp-size 8 --pp-size 1  --mem-fraction-static 0.9 --attention-backend dcu_mla 
      客户端：
              python -m sglang.bench_serving --backend sglang-oai \
              --model ${model_path} \
              --port $port --host 0.0.0.0 \
              --dataset-name random \
              --random-range-ratio 1 \
              --dataset-path ShareGPT_V3_unfiltered_cleaned_split.json \
              --random-input-len $prompt_tokens \
              --random-output-len $completion_tokens \
              --num-prompts $batch --max-concurrency $batch
    </lark-td>
  </lark-tr>
</lark-table>

过程数据表（需要按照数据库要求测试和填写）：

性能展示详表：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]

## 测试总结

- 本次das1.8为sglang的第一个版本，大语言问答模型在满足sla要求(ttft<3s，tpot小于50ms；16k、32k输入ttft<30s)情况下，共测试2个模型：Qwen3-32B-fp16、DeepSeek-R1-Channel-INT8；详细测试数据见：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1测试过程数据【sglang】--cz1KinDh|DAS1.8+DTK26.04-RC1测试过程数据【sglang】]]

# 模型篇（训练） <mention-user id="ou_6dee2267a57fdeb41d36ed3c41183a27"/>

### 测试概要

对发版镜像DAS1.8+DTK26.04在预训练方面的性能进行验证，具体的预训练模型为：llama2-7b，llama2-13b，qwen1.5-14b，qwen1.5-32b，resnet50；性能衡量指标为：TFLOPS。通过测试单机和双机的性能验证此次发版镜像的软件集性能。

### 测试目的

针对DAS1.8+DTK26.04进行发版相关测试，用于评估该软件版本集质量是否达标。

### 前置条件

表1硬件信息 {align="center"}

<sheet token="Nd9Csh0xchDvEOtT17Cciey1ni6_0tYd0C"/>

表2 软件信息 {align="center"}

<lark-table rows="6" cols="4" header-row="true" column-widths="102,115,141,806">

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
      Ubuntu22.04
    </lark-td>
    <lark-td>
      物理机不限
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
      6.3.29-V1.4.0b
    </lark-td>
    <lark-td>
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
      dtk-26.04
    </lark-td>
    <lark-td>
      终版实际应该是rc1版
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      软件栈
    </lark-td>
    <lark-td>
      DAS1.8
    </lark-td>
    <lark-td>
      以镜像为准
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
      DAS1.8+DTK-26.04
    </lark-td>
    <lark-td>
      http://112.11.119.99:18000/docker-images/das18-20260305/pytorch2.5.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
      http://112.11.119.99:18000/docker-images/das18-20260305/pytorch%3A2.7.1-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
    </lark-td>
  </lark-tr>
</lark-table>

### 测试要求

<lark-table rows="6" cols="3" header-row="true" column-widths="72,117,975">

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
      Llama2-7b
    </lark-td>
    <lark-td>
      （1）训练框架：dcu_megatron_v0.15.0
      （2）测试精度：bf16
      （3）测试参数：legacy+core：1*8卡（tp1pp2dp4）和2*8卡(tp1pp2dp8) bs64，256，1024性能
      （4）性能指标：TFLOPS
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      2
    </lark-td>
    <lark-td>
      Llama2-13b
    </lark-td>
    <lark-td>
      （1）训练框架：dcu_megatron_v0.15.0
      （2）测试精度：bf16
      （3）测试参数：legacy+core模式下，1*8卡（tp2pp2dp2）和2*8卡 (tp2pp2dp4)bs64，256，1024性能
      （4）性能指标：TFLOPS
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      3
    </lark-td>
    <lark-td>
      Qwen1.5-14b
    </lark-td>
    <lark-td>
      （1）训练框架：dcu_megatron_v0.15.0
      （2）测试精度：bf16
      （3）测试参数：legacy+core模式下，1*8卡（tp2pp4dp1）和2*8卡 (tp2pp2dp4) bs64，256，1024性能
      （4）性能指标：TFLOPS
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      Qwen1.5-32b
    </lark-td>
    <lark-td>
      （1）训练框架：dcu_megatron_v0.15.0
      （2）测试精度：bf16
      （3）测试参数：legacy+core模式下，2*8卡 (tp4pp4dp1) bs64，256，1024性能
      （4）性能指标：TFLOPS
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      Resnet50
    </lark-td>
    <lark-td>
      （1）测试精度：fp16-AMP
      （2）测试参数：bs=64，单卡，输入尺寸224*224
      （3）性能指标：simple/s
    </lark-td>
  </lark-tr>
</lark-table>

## 测试执行

<lark-table rows="10" cols="3" column-widths="100,120,944">

  <lark-tr>
    <lark-td>
    </lark-td>
    <lark-td>
      **模型**
    </lark-td>
    <lark-td>
      **测试命令**
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      1 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td rowspan="2">
      Llama2-7b
    </lark-td>
    <lark-td rowspan="2">
      mpirun -np ${GPUS}  --hostfile ${HOSTFILE} \
                          --allow-run-as-root \
                          --bind-to none \
                          --mca plm_rsh_no_tree_spawn 1 \
                          --prtemca plm_ssh_args "-p 12233" \
                          bash -c "
                          source ${DTK_ENV} && \
                          source ${NCCL_ENV} && \
                          ./train_llama2_7b_$((${GPUS} / 8))nodes.sh \
                          ${HOST} \
                          ${PORT} \
                          --data_path=$DATA_PATH \
                          --tokenizer_path=$TOKENIZER_MODEL_PATH \
                          --checkpoint_path=$CHECKPOINT_PATH \
                          --launch_with_binding=${LAUNCH_WITH_BINDING} \
                          --profiling=$profiling"
    </lark-td>
  </lark-tr>
  <lark-tr>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      2 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td rowspan="2">
      Llama2-13b
    </lark-td>
    <lark-td rowspan="2">
      mpirun -np ${GPUS}  --hostfile ${HOSTFILE} \
                          --allow-run-as-root \
                          --bind-to none \
                          --mca plm_rsh_no_tree_spawn 1 \
                          --prtemca plm_ssh_args "-p 12233" \
                          bash -c "
                          source ${DTK_ENV} && \
                          source ${NCCL_ENV} && \
                          ./train_llama2_13b_$((${GPUS} / 8))nodes.sh \
                          ${HOST} \
                          ${PORT} \
                          --data_path=$DATA_PATH \
                          --tokenizer_path=$TOKENIZER_MODEL_PATH \
                          --checkpoint_path=$CHECKPOINT_PATH \
                          --launch_with_binding=${LAUNCH_WITH_BINDING} \
                          --profiling=$profiling"
    </lark-td>
  </lark-tr>
  <lark-tr>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      3 {align="center"}
    </lark-td>
    <lark-td rowspan="2">
      Qwen1.5-14b
    </lark-td>
    <lark-td rowspan="2">
      mpirun -np ${GPUS}  --hostfile ${HOSTFILE} \
                          --allow-run-as-root \
                          --bind-to none \
                          --mca plm_rsh_no_tree_spawn 1 \
                          --prtemca plm_ssh_args "-p 12233" \
                          bash -c "
                          source ${DTK_ENV} && \
                          source ${NCCL_ENV} && \
                        ./train_qwen1.5_14B_$((${GPUS} / 8))nodes.sh \
                          ${HOST} \
                          ${PORT} \
                          --data_path=$DATA_PATH \
                          --tokenizer_path=$TOKENIZER_MODEL_PATH \
                          --checkpoint_path=$CHECKPOINT_PATH \
                          --launch_with_binding=${LAUNCH_WITH_BINDING} \
                          --profiling=$profiling"
    </lark-td>
  </lark-tr>
  <lark-tr>
  </lark-tr>
  <lark-tr>
    <lark-td rowspan="2">
      4 {align="center"}
    </lark-td>
    <lark-td rowspan="2">
      Qwen1.5-32b
    </lark-td>
    <lark-td rowspan="2">
      mpirun -np ${GPUS}  --hostfile ${HOSTFILE} \
                          --allow-run-as-root \
                          --bind-to none \
                          --mca plm_rsh_no_tree_spawn 1 \
                          --prtemca plm_ssh_args "-p 12233" \
                          bash -c "
                          source ${DTK_ENV} && \
                          source ${NCCL_ENV} && \
                         ./train_qwen1.5_32B_$((${GPUS} / 8))nodes.sh \
                          ${HOST} \
                          ${PORT} \
                          --data_path=$DATA_PATH \
                          --tokenizer_path=$TOKENIZER_MODEL_PATH \
                          --checkpoint_path=$CHECKPOINT_PATH \
                          --launch_with_binding=${LAUNCH_WITH_BINDING} \
                          --profiling=$profiling"
    </lark-td>
  </lark-tr>
  <lark-tr>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      Resnet50
    </lark-td>
    <lark-td>
      #!/bin/bash
      set -e
      export HIP_VISIBLE_DEVICES=0
      export HSA_FORCE_FINE_GRAIN_PCIE=1
      save_path=/workspace/resnet50_checkpoint_path
      mkdir -p "$save_path"
      BATCH=64
      ARCH=resnet50
      EPOCHS=90
      DATASET=/workspace/resnet50/dataset/
      python3 ../train.py \
          --batch-size $BATCH \
          --arch $ARCH \
          --epochs $EPOCHS \
          --amp --opt-level O1 --loss-scale=dynamic \
          --save-path $save_path \
          $DATASET
    </lark-td>
  </lark-tr>
</lark-table>

过程数据表（需要按照数据库要求测试和填写）：

性能展示详表：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]

## 测试总结

**竞品性能对比：**

- Llama2-7b

**  ** K100AI性能平均达到H20的 <text color="green">47.5%</text>；

   BW150性能平均达到H20的<text color="red"> 172.6%</text>；
   BW1000性能平均达到H20的 <text color="red">186.7%</text>；
   BW1100性能平均达到H20的 <text color="red">156.97%</text>；
   BW1101性能平均达到H20的 <text color="red">159.2%</text>。
- Llama2-13b

** **  K100AI性能平均达到H20的 <text color="green">51.2%</text>；

   BW150性能平均达到H20的<text color="red"> 143.5% </text>；
   BW1000性能平均达到H20的<text color="red"> 169.5%</text>；
   BW1100性能平均达到H20的 <text color="red">143.6%</text>；
   BW1101性能平均达到H20的<text color="red"> 150.2%</text>。

<image token="XhEJbejPmoqMhtxtRkvcwTVunFg" width="2244" height="1593" align="left"/>

- Qwen1.5-14b  
   K100AI性能平均达到H20的 <text color="green">48.9%</text>；
   BW150性能平均达到H20的 <text color="red">137.3%</text>；
   BW1000平均达到H20的 <text color="red">159.3%</text>；
   BW1100性能平均达到H20的 <text color="red">135.7%</text>；
   BW1101性能平均达到H20的 <text color="red">145.1%</text>。
- Qwen1.5-32b
  K100AI性能平均达到H20的 <text color="green">50.8%</text>；
  BW150性能平均达到H20的 <text color="red">143.4%</text>；
  BW1000平均达到H20的 <text color="red">156.8%</text>；
  BW1100性能平均达到H20的 <text color="red">138.1%</text>；
     BW1101性能平均达到H20的 <text color="red">140.2%</text>。

<image token="RaQ2bLrk2oHBcCxbk4IcYGzcngf" width="2182" height="1532" align="left"/>

**DAS1.8对比DAS1.7**

- Llama2-7b

DAS1.8性能最高达到DAS1.7的<text color="red">106.2%</text>；最低为DAS1.7的<text color="red">100.4%</text>；平均达到DAS1.7的<text color="red">102.9%</text>。

- Llama2-13b  

DAS1.8性能最高达到DAS1.7的<text color="red">101.1%</text>；最低为DAS1.7的<text color="green">92.3%</text>； 平均达到DAS1.7的<text color="green">96.1%</text> 。

- Qwen1.5-14b  

DAS1.8性能最高达到DAS1.7的<text color="red">101.21%</text>；最低为DAS1.7的<text color="green">93.89%</text><text color="red">；</text>平均达到DAS1.7的<text color="green">96.75%</text>。

- Qwen1.5-32b

 DAS1.8性能最高达到DAS1.7的<text color="red">98.9%</text>；  最低为DAS1.7的<text color="green">90.9%</text>；平均达到DAS1.7的<text color="green">95.2%</text>。

<image token="DnGTbLvPioeYUFxeUYQc0l1xnur" width="1933" height="1429" align="left"/>

# 精度篇 <mention-user id="ou_82b41c8fe95352c76681db657d66ade6"/>

## 测试概要

### 测试目的

对发版DAS1.8的vllm0.9.2/vllm0.11.0版本镜像在各个DCU加速卡上的模型功能进行验证。

### 前置条件

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
      CU数：64
      功耗：800W
    </lark-td>
    <lark-td>
      INTEL(R) XEON(R) PLATINUM 8558P
    </lark-td>
    <lark-td>
      10.16.6.62（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.63（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.66（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.67（INTEL(R) XEON(R) PLATINUM 8558P）
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      Hygon C86 7490
    </lark-td>
    <lark-td>
      10.16.6.11 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.23 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.61 (INTEL(R) XEON(R) PLATINUM 8558P)
      node027/022/74/78(Hygon C86 7490)
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      10.16.4.5 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.12 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.13 Intel(R) Xeon(R) Platinum 8458P
      10.16.4.14 Intel(R) Xeon(R) Platinum 8458P
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
</lark-table>

表2 软件信息

<lark-table rows="6" cols="4" header-row="true" column-widths="102,115,141,627">

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
      Ubuntu22.04
    </lark-td>
    <lark-td>
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
      6.3.29-V1.4.0b
    </lark-td>
    <lark-td>
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
      dtk-26.04
    </lark-td>
    <lark-td>
      终版实际应该是rc1版
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      软件栈
    </lark-td>
    <lark-td>
      DAS1.8
    </lark-td>
    <lark-td>
      以镜像为准
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
      DAS1.8+DTK-26.04
    </lark-td>
    <lark-td>
      http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
      http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.9.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz
    </lark-td>
  </lark-tr>
</lark-table>

### 测试要求

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
      （1）推理框架：vllm0.11.0
      （2）测试精度：bf16、int8
      （3）测试工具：opencompass（0.4.2）
      （4）数据集：gsm8k(134f0e)、math-500(11c4b5)、human-eval(dcae0e)
      （5）测试参数：
      NMZ：kv-cache-dtype fp8_e4m3、no_kvcache；
      BMZ：kv-cache-dtype fp8_e5m2、no_kvcache；
      K100AI：no_kvcache；
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
      （1）推理框架：vllm0.9.2
      （2）测试精度：fp8、bf16、int8、int4
      （4）数据集：gsm8k(134f0e)、math-500(11c4b5)、human-eval(dcae0e)
      （5）测试参数：
      NMZ：kv-cache-dtype fp8_e4m3、no_kvcache；
      BMZ：kv-cache-dtype fp8_e5m2、no_kvcache；
      K100AI：no_kvcache；
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
      （1）推理框架：vllm0.11.0
      （2）测试精度：fp8、bf16
      （4）数据集：gsm8k(134f0e)、math-500(11c4b5)、human-eval(dcae0e)
      （5）测试参数：
      NMZ：kv-cache-dtype fp8_e5m2、no_kvcache；
      BMZ：kv-cache-dtype fp8_e5m2、no_kvcache；
      K100AI：no_kvcache；
    </lark-td>
  </lark-tr>
</lark-table>

## 测试执行

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
      （1）v1+cuda模式
      （2）其他参数默认，额外加上**--disable-cascade-attn**
      详见：[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      deepseek系列模型
    </lark-td>
    <lark-td>
      （1）v1+fullcuda模式（fp8原生模型），v1+cuda模式（bf16以及量化蒸馏模型）
      （2）其他参数默认，额外加上**--disable-cascade-attn（fullcuda模式除外）**
      详见：[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      其他模型（glm、kimi、minimax）
    </lark-td>
    <lark-td>
      （1）v1+cuda模式
      （2）其他参数默认，额外加上**--disable-cascade-attn**
      详见：[[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
</lark-table>

精度展示详表：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]

## 测试总结

DAS1.8精度测试，测试模型共计41个，其中deepseek系列模型12个，qwen系列模型24个，其他模型5个，所涉及模型均在NMZ、BMZ、K100AI平台上对gam8k、math500、humaneval三个数据集上进行验证。其中，在NMZ、BMZ平台分别测试了开启kcache fp8和不开kcache fp8的精度数据。

以下分别为NMZ、BMZ、KME平台模型验证结果，<text color="red">标红</text>部分为异常模型。

<image token="BHeUbiDVnoi9jcxPI9QcHURAnTf" width="1618" height="338"/>

<image token="Hq8HbuGgxo5L28xCGgFcmAUXn8t" width="1618" height="338"/>

<image token="N06TbcU0ZoMyJxxi1FdcMW32nf6" width="1618" height="302"/>

遗留问题详见：

[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]

# 精度篇（sglang）<mention-user id="ou_073081244c5b4a58102ebe5ff6a0f012"/>

## 测试概要

### 测试目的

对发版DAS1.8的vllm0.9.2/vllm0.11.0版本镜像在各个DCU加速卡上的模型功能进行验证。

### 前置条件

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
      CU数：64
      功耗：800W
    </lark-td>
    <lark-td>
      INTEL(R) XEON(R) PLATINUM 8558P
    </lark-td>
    <lark-td>
      10.16.6.62（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.63（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.66（INTEL(R) XEON(R) PLATINUM 8558P）
      10.16.6.67（INTEL(R) XEON(R) PLATINUM 8558P）
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      BW1101（nmz）
    </lark-td>
    <lark-td>
      GC频率：1300Mhz
      HBM频: 875Mhz
      CU数：72
      功耗：800W
    </lark-td>
    <lark-td>
      Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      10.16.6.20 Intel(R) Xeon(R) Platinum 8468V
      10.16.6.22 Intel(R) Xeon(R) Platinum 8468V
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
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
      10.16.6.11 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.23 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.61 (INTEL(R) XEON(R) PLATINUM 8558P)
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
</lark-table>

表2 软件信息

<lark-table rows="6" cols="4" header-row="true" column-widths="172,172,155,643">

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
      实际覆盖物理机见：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]
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
      6.3.29-V1.4.0b
    </lark-td>
    <lark-td>
      VBIOS version: 6.314.0034001.990874
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
      DTK26.04-rc1
    </lark-td>
    <lark-td>
      终版实际应该是rc1版
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      软件栈
    </lark-td>
    <lark-td>
      DAS1.8
    </lark-td>
    <lark-td>
      以镜像为准
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      [http://112.11.119.99:18000/docker-images/das18-20260305/sglang0.5.7-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fsglang0.5.7-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
    </lark-td>
  </lark-tr>
</lark-table>

### 测试要求

<lark-table rows="5" cols="3" header-row="true" column-widths="72,140,796">

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
      （1）推理框架：sglang0.5.7
      （2）测试精度：bf16、int8
      （3）测试工具：sglang脚本benchamrak、opencompass（0.4.2）
      （4）数据集：gsm8k、mmlu、ceval、human-eval(dcae0e)
      （5）测试参数：
      NMZ：kv-cache-dtype fp8_e4m3、attention_backend fa3
      BMZ：kv-cache-dtype fp8_e5m2、attention_backend fa3
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
      （1）推理框架：sglang0.5.7
      （2）测试精度：p8、bf16、int8、int4
      （3）测试工具：sglang脚本benchamrak、opencompass（0.4.2）
      （4）数据集：gsm8k、mmlu、ceval、human-eval(dcae0e)
      （5）测试参数：
      NMZ：kv-cache-dtype fp8_e4m3、attention_backend dcu_mla
      BMZ：kv-cache-dtype fp8_e5m2、attention_backend dcu_mla
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      3
    </lark-td>
    <lark-td>
      多模态模型
    </lark-td>
    <lark-td>
      （1）推理框架：sglang0.5.7
      （2）测试精度：bf16、int8
      （3）测试工具：sglang脚本benchamrak、opencompass（0.4.2）
      （4）数据集：gsm8k、mmlu、ceval、human-eval(dcae0e)
      （5）测试参数：
      NMZ：kv-cache-dtype fp8_e4m3、attention_backend fa3、--mm-attention-backend fa3
      BMZ：kv-cache-dtype fp8_e5m2、attention_backend fa3、--mm-attention-backend fa3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      qwen3-next模型
    </lark-td>
    <lark-td>
      （1）推理框架：sglang0.5.7
      （2）测试精度：bf16、int8
      （3）测试工具：sglang脚本benchamrak、opencompass（0.4.2）
      （4）数据集：gsm8k、mmlu、ceval、human-eval(dcae0e)
      （5）测试参数：
      NMZ：kv-cache-dtype fp8_e4m3、attention_backend fa3、--mamba-scheduler-strategy extra_buffer
      BMZ：kv-cache-dtype fp8_e5m2、attention_backend fa3、--mamba-scheduler-strategy extra_buffer
    </lark-td>
  </lark-tr>
</lark-table>

## 测试执行

<lark-table rows="5" cols="2" header-row="true" column-widths="155,604">

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
      （1）--radix-cache、attention_backend fa3
      详见：[[飞书知识图谱/文档/Sglang测试指导（QTeam）--chGjinEd|Sglang测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      deepseek系列模型
    </lark-td>
    <lark-td>
      （1）--radix-cache、attention_backend dcu_mla
      详见：[[飞书知识图谱/文档/Sglang测试指导（QTeam）--chGjinEd|Sglang测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      多模态模型
    </lark-td>
    <lark-td>
      （1）--radix-cache、attention_backend fa3、--mm-attention-backend fa3
      详见：[[飞书知识图谱/文档/Sglang测试指导（QTeam）--chGjinEd|Sglang测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      qwen3-next模型
    </lark-td>
    <lark-td>
      （1）attention_backend fa3、--mamba-scheduler-strategy extra_buffer
      详见：[[飞书知识图谱/文档/Sglang测试指导（QTeam）--chGjinEd|Sglang测试指导（QTeam）]]
    </lark-td>
  </lark-tr>
</lark-table>

精度展示详表：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]

## 测试总结

DAS1.8精度测试，测试模型共计20个，其中deepseek系列模型7个，qwen系列模型7个，多模态模型5个，qwen3-next模型1个，所涉及模型均在NMZ、BMZ平台上对gsm8k、ceval、mmlu、humaneval三个数据集上进行验证。其中，在NMZ、BMZ平台分别测试了开启kcache fp8的精度数据。遗留问题详见：[[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]]

# 长稳测试篇 <mention-user id="ou_d0522c743979d8d557275c2dd4496289"/>

## 测试概要

### 测试目的

针对DAS1.8+DTK26.04进行发版相关测试，主要目的评估该软件镜像版本在长时间压测条件下的稳定性表现，测试方式为长上下文和qa多轮对话压测

### 前置条件

        配置以下环境
   表1 硬件信息

<lark-table rows="3" cols="5" header-row="true" column-widths="138,138,214,292,219">

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
      10.16.6.11 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.23 (Intel(R) Xeon(R) Platinum 8468V)
      10.16.6.61 (INTEL(R) XEON(R) PLATINUM 8558P)
    </lark-td>
    <lark-td>
      6.3.29-V1.4.0b
    </lark-td>
  </lark-tr>
</lark-table>

表2 软件信息

<lark-table rows="6" cols="4" header-row="true" column-widths="57,111,155,845">

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
      6.3.29-V1.4.0b
    </lark-td>
    <lark-td>
      VBIOS version: 6.314.0034001.990874
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
      DTK26.04-rc1
    </lark-td>
    <lark-td>
      终版实际应该是rc1版
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4
    </lark-td>
    <lark-td>
      软件栈
    </lark-td>
    <lark-td>
      DAS1.8
    </lark-td>
    <lark-td>
      以镜像为准
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5
    </lark-td>
    <lark-td>
      镜像
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
      [http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.9.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fvllm0.9.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
      [http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz](http%3A%2F%2F112.11.119.99%3A18000%2Fdocker-images%2Fdas18-20260305%2Fvllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz)
    </lark-td>
  </lark-tr>
</lark-table>

### 测试要求

表3 详细测试要求

<lark-table rows="3" cols="4" header-row="true" column-widths="72,117,492,953">

  <lark-tr>
    <lark-td>
      序号
    </lark-td>
    <lark-td>
      类别
    </lark-td>
    <lark-td>
      长上下文测试要求
    </lark-td>
    <lark-td>
      多轮对话测试要求
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      1
    </lark-td>
    <lark-td>
      DeepSeek-R1-Distill-Llama-70B
    </lark-td>
    <lark-td>
      （1）输入文本最大要达到大于64k
      （2）测试显卡：bw1000
      （3）镜像是<text color="purple" underline="true">http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz</text>
      server：默认模型最大上下文长度
    </lark-td>
    <lark-td>
      （1）测试显卡：bw1000
      （2）镜像是<text color="purple" underline="true">http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz</text>
      server：默认模型最大上下文长度
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      2
    </lark-td>
    <lark-td>
      Qwen3-32B
    </lark-td>
    <lark-td>
      （1）输入文本最大要达到40k以下
      （2）测试显卡：bw1000
      （3）镜像是**http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.9.2-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz**
      server:默认模型最大上下文长度
    </lark-td>
    <lark-td>
      （1）测试显卡：bw1000
      （2）镜像是<text color="purple" underline="true">http://112.11.119.99:18000/docker-images/das18-20260305/vllm0.11.0-ubuntu22.04-dtk26.04-rc1-py3.10-20260305.tar.gz</text>
      server：默认模型最大上下文长度
    </lark-td>
  </lark-tr>
</lark-table>

## 测试执行

<lark-table rows="4" cols="6" column-widths="120,100,393,631,753,1187">

  <lark-tr>
    <lark-td rowspan="2">
      模型 {align="center"}
    </lark-td>
    <lark-td rowspan="2">
      精度 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td colspan="2">
      长上下文压测 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td colspan="2">
      qa多轮对话压测 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      环境变量 {align="center"}
    </lark-td>
    <lark-td>
      测试命令 {align="center"}
    </lark-td>
    <lark-td>
      环境变量 {align="center"}
    </lark-td>
    <lark-td>
      测试命令 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      DeepSeek-R1-Distill-Llama-70B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      镜像默认 {align="center"}
    </lark-td>
    <lark-td>
      **服务端：**
      ```bash
      export HIP_VISIBLE_DEVICES=4,5,6,7
      model_path=/models/DeepSeek-R1-Distill-Llama-70B
      model=${model_path##*/}
      time=$(date "+%m%d-%H:%M")
      data_type="float16"
      tp=4
      port=8200
      if [ ! -f ${logpath} ]; then
          mkdir ${logpath} -p
      fi
      vllm serve ${model_path} \
          --host 0.0.0.0 --port $port \
          --dtype float16 --tensor-parallel-size $tp \
          --gpu-memory-utilization 0.9 \
          --trust-remote-code \
          --max-num-seqs 1024 --distributed-executor-backend=mp --no-enable-prefix-caching  --disable-cascade-attn
      ```
      **locust脚本**
      ```python
      import json
      import random
      import numpy as np
      import os
      import requests
      from locust import HttpUser, task, between
      from time import time
      from typing import Any, AsyncGenerator, Collection, Dict, List, Optional, Tuple
      from transformers import PreTrainedTokenizerBase
      try:
          from vllm.transformers_utils.tokenizer import get_tokenizer
      except ImportError:
          from backend_request_func import get_tokenizer
      requests.packages.urllib3.disable_warnings()
      data_folder = '/mnt/das18_test/longtext_test/wiki_text'
      # 遍历文件夹中所有 txt 文件
      txt_files = sorted([f for f in os.listdir(data_folder) if f.endswith('.txt')])
      class QwenModel(HttpUser):
          proxy = {}
          def on_start(self):
              pass
          def on_stop(self):
              pass
          @task
          def ask_and_reply(self):
              url = "http://0.0.0.0:8200/v1/chat/completions"
              headers = {"Content-Type": "application/json"}
              for txt_file in txt_files:
                  file_path = os.path.join(data_folder, txt_file)
                  # 读取文本内容
                  with open(file_path, 'r', encoding='utf-8') as f:
                      long_text = f.read()
                  # 构建请求 payload
                  payload = {
                      "model": "/models/DeepSeek-R1-Distill-Llama-70B",  # 替换为实际模型名称
                      "messages": [{"role": "user", "content": long_text}],
                      "max_tokens": 1000,
                      "temperature": 0.6,
                  }
                  # 发送请求
                  response = requests.post(url, headers=headers, data=json.dumps(payload))
                  # 处理响应
                  if response.status_code == 200:
                      result = response.json()
                      print(f"文件 {txt_file} 响应内容:\n", result['choices'][0]['message']['content'], "\n")
                  else:
                      print(f"文件 {txt_file} 请求失败 (状态码 {response.status_code}): {response.text}")
                      
      ```
      **客户端命令**
      ```plaintext
      locust -f dsr1_llama70b_locust.py --host http://0.0.0.0:8200 --headless -u 10 -r 0.5 --run-time 840h       
      ```
    </lark-td>
    <lark-td>
      镜像默认 {align="center"}
    </lark-td>
    <lark-td>
      **服务端：**
      ```bash
      export HIP_VISIBLE_DEVICES=0,1,2,3
      model_path=/models/DeepSeek-R1-Distill-Llama-70B
      model=${model_path##*/}
      time=$(date "+%m%d-%H:%M")
      data_type="float16"
      tp=4
      # logpath="./server/nmz/DeepSeek-R1-Distill-Llama-70B-main"
      port=8100
      if [ ! -f ${logpath} ]; then
          mkdir ${logpath} -p
      fi
      vllm serve ${model_path} \
          --host 0.0.0.0 --port $port \
          --dtype float16 --tensor-parallel-size $tp \
          --gpu-memory-utilization 0.9 \
          --trust-remote-code \
          --max-num-seqs 1024 --distributed-executor-backend=mp --no-enable-prefix-caching  --disable-cascade-attn 
      ```
      客户端：
      ```sql
      model_path=/models/DeepSeek-R1-Distill-Llama-70B
      python3 multi-round-qa.py \
      --num-users 10 \
      --num-rounds 5 \
      --qps 0.3 \
      --shared-system-prompt 1000 \
      --user-history-prompt 2000 \
      --answer-len 100 \
      --model ${model_path} \
      --base-url http://localhost:8100/v1 
      ```
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      Qwen3-32B
    </lark-td>
    <lark-td>
      fp16
    </lark-td>
    <lark-td>
      export HIP_VISIBLE_DEVICES=4,5,6,7
      export HSA_FORCE_FINE_GRAIN_PCIE=1
      export NCCL_MAX_NCHANNELS=16
      export NCCL_MIN_NCHANNELS=16
      export NCCL_P2P_LEVEL=SYS
      export NCCL_LAUNCH_MODE=GROUP
      export ALLREDUCE_STREAM_WITH_COMPUTE=1
      export VLLM_RPC_TIMEOUT=1800000
      export VLLM_NUMA_BIND=1
      export VLLM_RANK0_NUMA=7
      export VLLM_RANK1_NUMA=5
      export VLLM_RANK2_NUMA=5
      export VLLM_RANK3_NUMA=4
      export VLLM_SPEC_DECODE_EAGER=1
      export VLLM_ZERO_OVERHEAD=1 # zero
      export VLLM_USE_PD_SPLIT=1
      export VLLM_TORCH_PROFILER_DIR=/mnt/torchprof
      export VLLM_USE_FUSED_RMS_ROPE=1
      export VLLM_USE_OPT_RESHAPE_AND_CACHE=1
    </lark-td>
    <lark-td>
      **服务端：**
      ```sql
      vllm serve/models/Qwen3-32B \
      --host 0.0.0.0 \
      --port $port   \
      --dtype float16 \
      --tensor-parallel-size $tp \
      --trust-remote-code \
      --max-num-seqs 1024 \
      --distributed-executor-backend=mp \
      --enable-prefix-caching  \
      --enable-chunked-prefill  \
      --kv-cache-dtype fp8_e5m2
      ```
      **locust脚本：**
      ```python
      import json
      import random
      import numpy as np
      import os
      import requests
      from locust import HttpUser, task, between
      from time import time
      from typing import Any, AsyncGenerator, Collection, Dict, List, Optional, Tuple
      from transformers import PreTrainedTokenizerBase
      try:
          from vllm.transformers_utils.tokenizer import get_tokenizer
      except ImportError:
          from backend_request_func import get_tokenizer
      requests.packages.urllib3.disable_warnings()
      data_folder = '/mnt/das18_test/longtext_test/wiki_text'
      # 遍历文件夹中所有 txt 文件
      txt_files = sorted([f for f in os.listdir(data_folder) if f.endswith('.txt')])
      class QwenModel(HttpUser):
          proxy = {}
          def on_start(self):
              pass
          def on_stop(self):
              pass
          @task
          def ask_and_reply(self):
              url = "http://0.0.0.0:8002/v1/chat/completions"
              headers = {"Content-Type": "application/json"}
              for txt_file in txt_files:
                  file_path = os.path.join(data_folder, txt_file)
                  # 读取文本内容
                  with open(file_path, 'r', encoding='utf-8') as f:
                      long_text = f.read()
                  # 构建请求 payload
                  payload = {
                      "model": "/models/Qwen3-32B",  # 替换为实际模型名称
                      "messages": [{"role": "user", "content": long_text}],
                      "max_tokens": 1000,
                      "temperature": 0.6,
                  }
                  # 发送请求
                  response = requests.post(url, headers=headers, data=json.dumps(payload))
                  # 处理响应
                  if response.status_code == 200:
                      result = response.json()
                      print(f"文件 {txt_file} 响应内容:\n", result['choices'][0]['message']['content'], "\n")
                  else:
                      print(f"文件 {txt_file} 请求失败 (状态码 {response.status_code}): {response.text}")
                      
      ```
       **客户端命令：**
      ```plaintext
      locust -f qwen3_32b_locust.py --host http://0.0.0.0:8002 --headless -u 10 -r 0.5 --run-time 840h       
      ```
    </lark-td>
    <lark-td>
      export HIP_VISIBLE_DEVICES=0,1,2,3
      export HSA_FORCE_FINE_GRAIN_PCIE=1
      export NCCL_MAX_NCHANNELS=16
      export NCCL_MIN_NCHANNELS=16
      export NCCL_P2P_LEVEL=SYS
      export NCCL_LAUNCH_MODE=GROUP
      export ALLREDUCE_STREAM_WITH_COMPUTE=1
      export VLLM_RPC_TIMEOUT=1800000
      export VLLM_NUMA_BIND=1
      export VLLM_RANK0_NUMA=3
      export VLLM_RANK1_NUMA=1
      export VLLM_RANK2_NUMA=1
      export VLLM_RANK3_NUMA=0
      export VLLM_SPEC_DECODE_EAGER=1
      export VLLM_ZERO_OVERHEAD=1 # zero
      export VLLM_USE_PD_SPLIT=1
      export VLLM_TORCH_PROFILER_DIR=/mnt/torchprof
      export VLLM_USE_FUSED_RMS_ROPE=1
      export VLLM_USE_OPT_RESHAPE_AND_CACHE=1
    </lark-td>
    <lark-td>
      **服务端：**
      ```sql
      vllm serve /models/Qwen3-32B \
      --host 0.0.0.0 \
      --port $port   \
      --dtype float16 \
      --tensor-parallel-size $tp \
      --trust-remote-code \
      --max-num-seqs 1024 \
      --distributed-executor-backend=mp \
      --enable-prefix-caching  \
      --enable-chunked-prefill  \
      --kv-cache-dtype fp8_e5m2 \
      2>&1 | tee  ${logpath}.log
      ```
      **客户端命令:**
      ```sql
      model_path=/models/Qwen3-32B
      python3 multi-round-qa.py \
      --num-users 10 \
      --num-rounds 5 \
      --qps 0.3 \
      --shared-system-prompt 1000 \
      --user-history-prompt 2000 \
      --answer-len 100 \
      --model ${model_path} \
      --base-url http://localhost:8001/v1 
      ```
    </lark-td>
  </lark-tr>
</lark-table>

测试结果展示

<sheet token="Nd9Csh0xchDvEOtT17Cciey1ni6_nWrJB9"/>

<sheet token="Nd9Csh0xchDvEOtT17Cciey1ni6_DcwKBV"/>

功耗频率曲线

<image token="RHZ6bb8LyoMIEtxc01ycd8GtnNc" width="3228" height="924" align="center"/>

## 测试总结

对发版镜像进行稳定性测试，测试方式分为长上下文连续压测以及多轮对话压测，以此评估发版镜像的稳定性，保障客户运行。测试模型选用DeepSeek-R1-Distill-Llama-70B和Qwen3-32B，测试显卡是BW1000，长上下文和qa多轮对话压测测试过程连续未中断。

# 兼容性篇 <mention-user id="ou_40f335d52091292b141a99186f59ef49"/>

## 测试概要

对发版DAS1.8的whl包+DTK256.04在各个操作系统上的可用性进行验证。

### 测试目的

对发版DAS1.8的whl包+DTK26.04在各个操作系统上的可用性进行验证，硬件环境NMZ卡，操作系统包括kylinv10、ubuntu20.04.1、nfs4.0、openeuler22.03、rocklinux8.6、tencentos3。

### 前置条件

1、准备NMZ加速卡的硬件环境，同2.1.2节

2、准备das1.8发版whl包

vllm011-0305：
```bash {wrap}
http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/9b0/11f1845165e7d/torch-2.5.1+das.opt1.dtk2604-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=9b011f1845165e7dd3d7f7ea47c147d4e664e6afc1fe078aca95b7d70fbfbde2

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/e3f/e5fbafbdf7906/triton-3.1.0+das.opt1.dtk2604-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=e3fe5fbafbdf79066ccd29eb91e4c855e228f4ccc5fee769d4cd0a8d54be9c36

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/e76/37bc8cc4d3c32/torchvision-0.20.1+das.opt1.dtk2604-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=e7637bc8cc4d3c329c5ed90cc6b600af2d9fe6a40d1d69bffe6cf0fd18f5fa4a

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/796/491f663f511d7/torchaudio-2.5.1+das.opt1.dtk2604-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=796491f663f511d73bc0c0818f86205b7daa28dc41d8a2ae5f8e83596b6fcda0

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/eba/40e048f6f9130/flash_attn-2.6.1+das.opt1.dtk2604.torch251-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=eba40e048f6f9130f065dc0ebda3fca93b0fe4c71c19cc5c1df4aaf9dfe7364d

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/4a8/e2db7125ccc4e/flash_mla-1.0.0+das.opt1.dtk2604.torch251-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=4a8e2db7125ccc4ea0f33d621b0ffe99c2f4b9b0071a0647909824a3a0f44072

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/b91/ee1d78e5c49ca/lightop-0.6.0+das.dtk2604.torch251-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=b91ee1d78e5c49cad5a523006e341e29f5e00bbd047defe0a2356d50968b8041

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/be9/7a6b6f4b01eb3/lmslim-0.3.1+das.opt1.dtk2604.torch251-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=be97a6b6f4b01eb33865a870f56c3c71adea47d74d22726cd97f5d9fc0886eef

http://10.16.4.1:9929/das_release/dtk2604-rc1/+f/042/967885d19a676/vllm-0.11.0+das.opt1.rc4.dtk2604-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=042967885d19a6769d0fad47d3c0548195fc106b8b5a3167d493a69c433a4619

 pip install vllm==0.11.0 -i http://10.16.4.1:9929/release/dtk2604/+simple/ --trusted-host 10.16.4.1
```

torch2.7-0305：
```bash {wrap}
apex @ http://10.16.4.1:8000/customized/apex/dtk-26.04-NMZ-0119-centos8/1.5.0%2Bdas.opt1.dtk25042-89c00191-torch271/apex-1.5.0%2Bdas.opt1.dtk2604.20260128.g89c00191-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=59b63903dc25caf39a07395d62d929cef47f46a8ccc20d8a3a1faf4e47b7da99

cupy @ http://10.16.4.1:8000/temp/cupy-12.3.0-cp310-cp310-linux_x86_64.whl#sha256=be01dc95748dd56d122f8b4e1f8a6a3c9a0da352d59d0ed30fc4aa503744ef74

dcu-megatron @ http://10.16.4.1:8000/customized/dcu_megatron/dtk-26.04-beta-0130-centos8/core_v0.15.0-ae67d7e2-torch2.7.1/dcu_megatron-0.15.0%2Bdas.opt1.dtk2604.torch271.20260204.gae67d7e2-py3-none-any.whl#sha256=6c7692c1f4e6b8dc8d841e469fdb5a55bbc4a3d2159aac4224f6ea0f81636389

flash_attn @ http://10.16.4.1:8000/customized/flash-attn/dtk-26.04-DCC2602-0210-centos8/2.6.1%2Bdas.opt1.dtk2604-6c6c3ed1-torch2.7.1/flash_attn-2.6.1%2Bdas.opt1.dtk2604.torch271.20260304.g6c6c3ed1-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=092b717b55251bf3f8be06e2ed1881a0d244b4f50aa97d9657899f8b197c9de7

flash_mla @ http://10.16.4.1:8000/customized/flashmla/dtk-26.04-beta-0130-centos8/master-dev-for-yuanbao-4742da75-torch2.7.1/flash_mla-1.0.0%2Bdas.opt1.dtk2604.torch271.20260303.g4742da75-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=84e19520c1800f0270b4c3261a2393dee740ffa7e5314652f7f91e48b03b77b5

lightop @ http://10.16.4.1:8000/customized/lightop/dtk-26.04-beta-0130-centos8/lightop-dev-1fdc3ed2-torch2.7.1/lightop-0.6.0%2Bdas.dtk2604.torch271.20260210.g1fdc3ed2-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=b19df40f1d91312652c50ec5042584cafb4266a8e7276f95e2c03a487aa2bf32
lmslim @ http://10.16.4.1:8000/customized/lmslim/dtk-26.04-beta-0130-centos8/0.3.1%2Bdas.opt4.dtk2604-c8327617-torch2.7.1/lmslim-0.3.1%2Bdas.opt4.dtk2604.torch271.20260210.gc8327617-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=d706ea843cdc242d704fd27d8d5c8a023b542b1fcb1627c5fe8335df96ce42f6

megatron-core @ http://10.16.4.1:8000/customized/dcu_megatron/dtk-26.04-beta-0130-centos8/core_v0.15.0-0d7e02bd-torch2.7.1/megatron_core-0.15.0%2Bdas.opt1.dtk2604.torch271.20260204.g0d7e02bd-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=8a5cce29cc1108f06c49b659d6edcc1792a7c5101e29ab5116e10c662dee804e

ray @ http://10.16.4.1:8000/temp/ray-2.48.0-cp310-cp310-manylinux2014_x86_64.whl#sha256=649ed9442dc2d39135c593b6cf0c38e8355170b92672365ab7a3cbc958c42634

torch @ http://10.16.4.1:8000/customized/pytorch/dtk-26.04-beta-0130-centos8/2.7.1%2Bdas.opt1.dtk2604-rc1-04100de4/torch-2.7.1%2Bdas.opt1.dtk2604.20260206.g04100de4-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=352a7284cbf797467a31e42f1be641c93aa7f42920c2efae89fd3967b63164db

torchvision @ http://10.16.4.1:8000/customized/vision/dtk-26.04-beta-0130-centos8/0.22.0%2Bdas.opt1.dtk25042-2a371b9c-torch2.7.1/torchvision-0.22.0%2Bdas.opt1.dtk2604.20260210.g2a371b9c-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=ac4a94258403e2c4b5435a8043ccc0f94183ac6aa816355705493e598242d7ce

transformer_engine @ http://10.16.4.1:8000/customized/transformer_engine/dtk-26.04-NMZ-0119-centos8/release_v2.10-d954fc54-torch2.7/transformer_engine-2.10.0%2Bdas.opt1.dtk2604.20260202.gd954fc54-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=b97aa89f16017f79176373edff981ee6c7b71831db5913c2ed793aa9cc582ec4

triton @ http://10.16.4.1:8000/customized/triton/dtk-26.04-beta-0130-centos8/triton-lang-6964b550-torch2.7.1/triton-3.1.0%2Bdas.opt2.dtk2604.torch271.20260211.g6964b550-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=76d78fd703eda5237c1a479de0a51472a49def1e5c6e47a5f972d08321348572

vllm @ http://10.16.4.1:8000/customized/vllm/dtk-26.04-beta-0130-centos8/v0.11.0-dev-49a30c70-torch2.7.1/vllm-0.11.0%2Bdas.opt1.rc4.dtk2604.torch271.20260305.g49a30c70-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=2d96640f545f907a799ff91ffb3b4bd3e02a4ef49ff4a50732ee70dfbcabbda8
```

torch290:
```bash {wrap}
amdsmi @ http://10.16.4.1:8000/customized/amdsmi/dtk-26.04-beta-0130-centos8/amdsmi-torch2.9.1/amdsmi-24.5.3%2B02cbffb.dirty-py3-none-any.whl#sha256=4621271c17e982f5cd3eeef4c4814e85060299ef1ee874a5f7d40a1fe1c4b054
cupy @ http://10.16.4.1:8000/temp/cupy-12.3.0-cp310-cp310-linux_x86_64.whl#sha256=be01dc95748dd56d122f8b4e1f8a6a3c9a0da352d59d0ed30fc4aa503744ef74
flash_attn @ http://10.16.4.1:8000/customized/flash-attn/dtk-26.04-beta-0130-centos8/master-dev-7808665e-torch2.9.1/flash_attn-2.6.1%2Bdas.opt1.dtk2604.torch291.20260210.g7808665e-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=738835ab87b8dc7dbc33bf66088fab1cd6476ff8bf06c395e5b4cb2754b279f9
flash_mla @ http://10.16.4.1:8000/customized/flashmla/dtk-26.04-NMZ-0119-centos8/model1-06612d65-torch2.9.1/flash_mla-1.0.0-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=967d5a64017aee38fdd64ee59437a7842eecf8a9a4f166d8947f47da4b9804b0
lightop @ http://10.16.4.1:8000/customized/lightop/dtk-26.04-beta-0130-centos8/lightop-dev-1fdc3ed2-torch2.9.1/lightop-0.6.0%2Bdas.dtk2604.torch291.20260211.g1fdc3ed2-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=758636fbad6f03a0cebd6f0ca606f3f9987615eb52e32e9ef1be30b61b10843e
lmslim @ http://10.16.4.1:8000/customized/lmslim/dtk-26.04-beta-0130-centos8/0.3.1%2Bdas.opt4.dtk2604-c8327617-torch2.9.1/lmslim-0.3.1%2Bdas.opt4.dtk2604.torch291.20260211.gc8327617-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=218bd85c1b85857d056f2c6cbc2d9becd90b9ecde552b924c30c6a70d0b93b27
ray @ http://10.16.4.1:8000/temp/ray-2.48.0-cp310-cp310-manylinux2014_x86_64.whl#sha256=649ed9442dc2d39135c593b6cf0c38e8355170b92672365ab7a3cbc958c42634
torch @ http://10.16.4.1:8000/customized/pytorch/dtk-26.04-beta-0130-centos8/2.9.0%2Bdas.opt1.dtk2604-rc1-275d08c2/torch-2.9.0%2Bdas.opt1.dtk2604.20260206.g275d08c2-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=2399cb630f0c0a19c48064e71b0116ed75793a24c49b712b6d011d8dcd6baed3
torchvision @ http://10.16.4.1:8000/customized/vision/dtk-26.04-beta-0130-centos8/0.24.0-dev-f0277aff-torch2.9.1/torchvision-0.24.0%2Bdas.opt1.dtk2604.20260210.gf0277aff-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=ed652100529e347a8e3a16ff6e4f4dd6022fa0546a948bd425c51856caaa75ae
triton @ http://10.16.4.1:8000/customized/triton/dtk-26.04-beta-0130-centos8/triton3.3-1329924c-torch2.9.1/triton-3.3.0%2Bdas.opt2.dtk2604.torch291.20260210.g1329924c-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=8e8f45f8bc17a33dd0ddb2efc9b6f9a6c3c115a75e94ec46235a7fddc7ea4822
vllm @ http://10.16.4.1:8000/customized/vllm/dtk-26.04-beta-0130-centos8/v0.11.0-torch2.9-2e73602b-torch2.9.1/vllm-0.11.0%2Bdas.opt1.rc2.torch29.dtk2604.20260211.g2e73602b-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=05f706dc3d9d7658e3c1a1fd0eb7a440f318a71624f2f2e5dfc1051dbaeace51
```


3、准备DTK26.04软件包

/public/opendas/ArchivedFile/dtk-pkg/dtk26.04/DTK-26.04-rc4-ubuntu20.04-x86_64.tar.gz

/public/opendas/ArchivedFile/dtk-pkg/dtk26.04/DTK-26.04-rc4-centos8-x86_64.tar.gz

### 测试要求

<lark-table rows="7" cols="4" column-widths="121,332,298,279">

  <lark-tr>
    <lark-td>
      操作系统
    </lark-td>
    <lark-td>
      使用镜像
    </lark-td>
    <lark-td>
      DTK
    </lark-td>
    <lark-td>
      测试要求
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      kylinv10
    </lark-td>
    <lark-td>
      sugonhub/kylin:v10-dev
    </lark-td>
    <lark-td>
      DTK-26.04-rc4-centos8-x86_64.tar.gz
    </lark-td>
    <lark-td>
      1、whl包安装验证；2、组件可用性验证
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      ubuntu20.04.1
    </lark-td>
    <lark-td>
      sugonhub/ubuntu:20.04.1-dev
    </lark-td>
    <lark-td>
      DTK-26.04-rc4-ubuntu20.04-x86_64.tar.gz
    </lark-td>
    <lark-td>
      1、whl包安装验证；2、组件可用性验证
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      nfs4.0
    </lark-td>
    <lark-td>
      sugonhub/nfs:4.0-dev
    </lark-td>
    <lark-td>
      DTK-26.04-rc4-centos8-x86_64.tar.gz
    </lark-td>
    <lark-td>
      1、whl包安装验证；2、组件可用性验证
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      openeuler22.03
    </lark-td>
    <lark-td>
      sugonhub/openeuler:22.03-dev
    </lark-td>
    <lark-td>
      DTK-26.04-rc4-centos8-x86_64.tar.gz
    </lark-td>
    <lark-td>
      1、whl包安装验证；2、组件可用性验证
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      rocklinux8.6
    </lark-td>
    <lark-td>
      [image.sourcefind.cn:5000/dcu/admin/base/dev:rockylinux8.6-mpi5.0-gcc10.3-cmake3.29-py3.10-mkl2020.4.304](http%3A%2F%2Fimage.sourcefind.cn%3A5000%2Fdcu%2Fadmin%2Fbase%2Fdev%3Arockylinux8.6-mpi5.0-gcc10.3-cmake3.29-py3.10-mkl2020.4.304)
    </lark-td>
    <lark-td>
      DTK-26.04-rc4-centos8-x86_64.tar.gz
    </lark-td>
    <lark-td>
      1、whl包安装验证；2、组件可用性验证
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      tencentos3
    </lark-td>
    <lark-td>
      tencentos/tencentos3:latest
    </lark-td>
    <lark-td>
      DTK-26.04-rc4-centos8-x86_64.tar.gz
    </lark-td>
    <lark-td>
      1、whl包安装验证；2、组件可用性验证
    </lark-td>
  </lark-tr>
</lark-table>

## 测试执行

### kylinv10

#### 1、创建容器：

docker run -it --privileged --group-add video   -v /public/home/zhumc/vllm-test/das18test/18whltest:/public/home/zhumc/vllm-test/das18test/18whltest -v /public/opendas/ArchivedFile/dtk-pkg/dtk26.04:/public/opendas/ArchivedFile/dtk-pkg/dtk26.04:ro   -v /opt/hyhal:/opt/hyhal:ro  --shm-size=32G --network=host --name zmc-kylin sugonhub/kylin:v10-dev bash

#### 2、安装python

bash python310-install.sh
```bash {wrap}
export PATH=/usr/local/bin:${PATH}
export LD_LIBRARY_PATH=/usr/local/lib/:/usr/local/lib64/:${LD_LIBRARY_PATH}
export PYTHONPATH=/usr/local/:${PYTHONPATH}
cd /tmp \
&&     wget -O python.tgz http://10.16.4.1:8000/pkg/python/Python-3.10.12.tgz \
&& mkdir python-tmp \
&& tar -xvf python.tgz -C ./python-tmp --strip-components 1 \
&& cd python-tmp \
&& ./configure \
        --enable-shared \
&& make -j $CPUS \
&& make install \
&& rm -rf /tmp/python* \
        && ln -s /usr/local/bin/python3 /usr/local/bin/python \
&& ln -sf /usr/local/bin/pip3 /usr/local/bin/pip
export PATH=/usr/local/bin:${PATH}
export LD_LIBRARY_PATH=/usr/local/lib/:/usr/local/lib64/:${LD_LIBRARY_PATH}
export PYTHONPATH=/usr/local/:${PYTHONPATH}
```

#### 3、安装dtk-centos
```python {wrap}
cp /public/opendas/ArchivedFile/dtk-pkg/dtk26.04/DTK-26.04-rc4-centos8-x86_64.tar.gz  /opt/
cd /opt/
tar -zxvf DTK-26.04-rc4-centos8-x86_64.tar.gz
ln -s dtk-26.04-rc4  dtk
source /opt/dtk/env.sh
rocm-smi
```

#### 4、安装das包

vllm011-0305
```bash {wrap}
 pip install vllm==0.11.0 -i http://10.16.4.1:9929/release/dtk2604/+simple/ --trusted-host 10.16.4.1
 pip install torchaudio==2.5.1 -i http://10.16.4.1:9929/release/dtk2604/+simple/ --trusted-host 10.16.4.1
```

torch271：
```bash {wrap}
 pip install torch==2.7.1 -i http://10.16.4.1:9929/release/dtk2604/+simple/ --trusted-host 10.16.4.1
 pip install http://10.16.4.1:8000/customized/apex/dtk-26.04-NMZ-0119-centos8/1.5.0%2Bdas.opt1.dtk25042-89c00191-torch271/apex-1.5.0%2Bdas.opt1.dtk2604.20260128.g89c00191-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=59b63903dc25caf39a07395d62d929cef47f46a8ccc20d8a3a1faf4e47b7da99
 pip install http://10.16.4.1:8000/temp/cupy-12.3.0-cp310-cp310-linux_x86_64.whl#sha256=be01dc95748dd56d122f8b4e1f8a6a3c9a0da352d59d0ed30fc4aa503744ef74
 pip install http://10.16.4.1:8000/customized/dcu_megatron/dtk-26.04-beta-0130-centos8/core_v0.15.0-ae67d7e2-torch2.7.1/dcu_megatron-0.15.0%2Bdas.opt1.dtk2604.torch271.20260204.gae67d7e2-py3-none-any.whl#sha256=6c7692c1f4e6b8dc8d841e469fdb5a55bbc4a3d2159aac4224f6ea0f81636389
 pip install http://10.16.4.1:8000/customized/flash-attn/dtk-26.04-DCC2602-0210-centos8/2.6.1%2Bdas.opt1.dtk2604-6c6c3ed1-torch2.7.1/flash_attn-2.6.1%2Bdas.opt1.dtk2604.torch271.20260304.g6c6c3ed1-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=092b717b55251bf3f8be06e2ed1881a0d244b4f50aa97d9657899f8b197c9de7
 pip install http://10.16.4.1:8000/customized/flashmla/dtk-26.04-beta-0130-centos8/master-dev-for-yuanbao-4742da75-torch2.7.1/flash_mla-1.0.0%2Bdas.opt1.dtk2604.torch271.20260303.g4742da75-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=84e19520c1800f0270b4c3261a2393dee740ffa7e5314652f7f91e48b03b77b5
 pip install http://10.16.4.1:8000/customized/lightop/dtk-26.04-beta-0130-centos8/lightop-dev-1fdc3ed2-torch2.7.1/lightop-0.6.0%2Bdas.dtk2604.torch271.20260210.g1fdc3ed2-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=b19df40f1d91312652c50ec5042584cafb4266a8e7276f95e2c03a487aa2bf32
 pip install http://10.16.4.1:8000/customized/lmslim/dtk-26.04-beta-0130-centos8/0.3.1%2Bdas.opt4.dtk2604-c8327617-torch2.7.1/lmslim-0.3.1%2Bdas.opt4.dtk2604.torch271.20260210.gc8327617-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=d706ea843cdc242d704fd27d8d5c8a023b542b1fcb1627c5fe8335df96ce42f6
 pip install http://10.16.4.1:8000/customized/dcu_megatron/dtk-26.04-beta-0130-centos8/core_v0.15.0-0d7e02bd-torch2.7.1/megatron_core-0.15.0%2Bdas.opt1.dtk2604.torch271.20260204.g0d7e02bd-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=8a5cce29cc1108f06c49b659d6edcc1792a7c5101e29ab5116e10c662dee804e
 pip install http://10.16.4.1:8000/temp/ray-2.48.0-cp310-cp310-manylinux2014_x86_64.whl#sha256=649ed9442dc2d39135c593b6cf0c38e8355170b92672365ab7a3cbc958c42634
 pip install  http://10.16.4.1:8000/customized/pytorch/dtk-26.04-beta-0130-centos8/2.7.1%2Bdas.opt1.dtk2604-rc1-04100de4/torch-2.7.1%2Bdas.opt1.dtk2604.20260206.g04100de4-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=352a7284cbf797467a31e42f1be641c93aa7f42920c2efae89fd3967b63164db
 pip install http://10.16.4.1:8000/customized/vision/dtk-26.04-beta-0130-centos8/0.22.0%2Bdas.opt1.dtk25042-2a371b9c-torch2.7.1/torchvision-0.22.0%2Bdas.opt1.dtk2604.20260210.g2a371b9c-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=ac4a94258403e2c4b5435a8043ccc0f94183ac6aa816355705493e598242d7ce
 pip install http://10.16.4.1:8000/customized/transformer_engine/dtk-26.04-NMZ-0119-centos8/release_v2.10-d954fc54-torch2.7/transformer_engine-2.10.0%2Bdas.opt1.dtk2604.20260202.gd954fc54-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=b97aa89f16017f79176373edff981ee6c7b71831db5913c2ed793aa9cc582ec4
 pip install  http://10.16.4.1:8000/customized/triton/dtk-26.04-beta-0130-centos8/triton-lang-6964b550-torch2.7.1/triton-3.1.0%2Bdas.opt2.dtk2604.torch271.20260211.g6964b550-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=76d78fd703eda5237c1a479de0a51472a49def1e5c6e47a5f972d08321348572
 pip install http://10.16.4.1:8000/customized/vllm/dtk-26.04-beta-0130-centos8/v0.11.0-dev-49a30c70-torch2.7.1/vllm-0.11.0%2Bdas.opt1.rc4.dtk2604.torch271.20260305.g49a30c70-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=2d96640f545f907a799ff91ffb3b4bd3e02a4ef49ff4a50732ee70dfbcabbda8

```

torch290：
```bash {wrap}
pip install http://10.16.4.1:8000/customized/amdsmi/dtk-26.04-beta-0130-centos8/amdsmi-torch2.9.1/amdsmi-24.5.3%2B02cbffb.dirty-py3-none-any.whl#sha256=4621271c17e982f5cd3eeef4c4814e85060299ef1ee874a5f7d40a1fe1c4b054
pip install http://10.16.4.1:8000/temp/cupy-12.3.0-cp310-cp310-linux_x86_64.whl#sha256=be01dc95748dd56d122f8b4e1f8a6a3c9a0da352d59d0ed30fc4aa503744ef74
pip install  http://10.16.4.1:8000/customized/flash-attn/dtk-26.04-beta-0130-centos8/master-dev-7808665e-torch2.9.1/flash_attn-2.6.1%2Bdas.opt1.dtk2604.torch291.20260210.g7808665e-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=738835ab87b8dc7dbc33bf66088fab1cd6476ff8bf06c395e5b4cb2754b279f9
pip install  http://10.16.4.1:8000/customized/flashmla/dtk-26.04-NMZ-0119-centos8/model1-06612d65-torch2.9.1/flash_mla-1.0.0-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=967d5a64017aee38fdd64ee59437a7842eecf8a9a4f166d8947f47da4b9804b0
pip install http://10.16.4.1:8000/customized/lightop/dtk-26.04-beta-0130-centos8/lightop-dev-1fdc3ed2-torch2.9.1/lightop-0.6.0%2Bdas.dtk2604.torch291.20260211.g1fdc3ed2-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=758636fbad6f03a0cebd6f0ca606f3f9987615eb52e32e9ef1be30b61b10843e
pip install http://10.16.4.1:8000/customized/lmslim/dtk-26.04-beta-0130-centos8/0.3.1%2Bdas.opt4.dtk2604-c8327617-torch2.9.1/lmslim-0.3.1%2Bdas.opt4.dtk2604.torch291.20260211.gc8327617-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=218bd85c1b85857d056f2c6cbc2d9becd90b9ecde552b924c30c6a70d0b93b27
pip install  http://10.16.4.1:8000/temp/ray-2.48.0-cp310-cp310-manylinux2014_x86_64.whl#sha256=649ed9442dc2d39135c593b6cf0c38e8355170b92672365ab7a3cbc958c42634
pip install  http://10.16.4.1:8000/customized/pytorch/dtk-26.04-beta-0130-centos8/2.9.0%2Bdas.opt1.dtk2604-rc1-275d08c2/torch-2.9.0%2Bdas.opt1.dtk2604.20260206.g275d08c2-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=2399cb630f0c0a19c48064e71b0116ed75793a24c49b712b6d011d8dcd6baed3
pip install  http://10.16.4.1:8000/customized/vision/dtk-26.04-beta-0130-centos8/0.24.0-dev-f0277aff-torch2.9.1/torchvision-0.24.0%2Bdas.opt1.dtk2604.20260210.gf0277aff-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=ed652100529e347a8e3a16ff6e4f4dd6022fa0546a948bd425c51856caaa75ae
pip install  http://10.16.4.1:8000/customized/triton/dtk-26.04-beta-0130-centos8/triton3.3-1329924c-torch2.9.1/triton-3.3.0%2Bdas.opt2.dtk2604.torch291.20260210.g1329924c-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=8e8f45f8bc17a33dd0ddb2efc9b6f9a6c3c115a75e94ec46235a7fddc7ea4822
pip install  http://10.16.4.1:8000/customized/vllm/dtk-26.04-beta-0130-centos8/v0.11.0-torch2.9-2e73602b-torch2.9.1/vllm-0.11.0%2Bdas.opt1.rc2.torch29.dtk2604.20260211.g2e73602b-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=05f706dc3d9d7658e3c1a1fd0eb7a440f318a71624f2f2e5dfc1051dbaeace51
```


#### 5、测试导入whl包

#vllm011-0305
```python {wrap}
#vllm011-0305
python -c "import torch;print(torch.__version__,'\n',torch.__hcu_version__,'\n',torch.cuda.is_available())"
python -c "import torchvision;print(torchvision.__version__,'\n')"
python -c "import torchaudio;print(torchaudio.__version__,'\n')"
python -c "import lightop;print(lightop.__version__,'\n',lightop.__dcu_version__)"
python -c "import flash_attn;print(flash_attn.__version__,'\n',flash_attn.__hcu_version__)"
python -c "import vllm;print(vllm.__version__,'\n')"
python -c "import triton;print(triton.__version__,'\n')"
python -c "import torch;import lmslim;print(lmslim.__version__,'\n',lmslim.__hcu_version__)"
python -c "import flash_mla;print(flash_mla.__version__,'\n')"
```

cp libgfortran.so.5.0.0 /usr/lib64/

ln -s /usr/lib64/libgfortran.so.5.0.0 /usr/lib64/libgfortran.so.5

export LD_LIBRARY_PATH=/usr/lib64:/usr/lib:$LD_LIBRARY_PATH

#torch271
```python {wrap}
# torch271
python -c "import torch;print('torch:', torch.__version__, '\n', 'hcu_version:', getattr(torch, '__hcu_version__', 'N/A'), '\n', 'cuda_available:', torch.cuda.is_available(), '\n')"
python -c "import torchvision;print('torchvision:', torchvision.__version__)"
python -c "import apex;print('apex:', apex.__version__)"
python -c "import lightop;print('lightop:', lightop.__version__, '\n', lightop.__dcu_version__)"
python -c "import flash_attn;print('flash_attn:', flash_attn.__version__)"
python -c "import flash_mla;print('flash_mla:', flash_mla.__version__)"
python -c "import vllm;print('vllm:', vllm.__version__)"
python -c "import triton;print('triton:', triton.__version__)"
python -c "import lmslim;print('lmslim:', lmslim.__version__)"
python -c "import transformer_engine;print('transformer_engine:', transformer_engine.__version__)"
python -c "import megatron;"
python -c "import dcu_megatron;"
#megatron和dcu_megatron要先下载Megatron-LM-core_r0.15.0代码（git clone https://github.com/NVIDIA/Megatron-LM.git），修改pretrain_gpt.py文件，增加一行：from dcu_megatron import megatron_adaptor 在代码目录里面导入模块
```

#torch290
```bash {wrap}
python -c "import torch;print('torch:', torch.__version__, '\n', 'hcu_version:', getattr(torch, '__hcu_version__', 'N/A'), '\n', 'cuda_available:', torch.cuda.is_available(), '\n')"
python -c "import torchvision;print('torchvision:', torchvision.__version__)"
python -c "import lightop;print('lightop:', lightop.__version__, '\n', lightop.__dcu_version__)"
python -c "import flash_attn;print('flash_attn:', flash_attn.__version__)"
python -c "import flash_mla;print('flash_mla:', flash_mla.__version__)"
python -c "import vllm;print('vllm:', vllm.__version__)"
python -c "import triton;print('triton:', triton.__version__)"
python -c "import lmslim;print('lmslim:', lmslim.__version__)"

```


#### 6、测试结果

vllm011-0305:

<image token="GXF0biHVFodM4SxyIzpcFXuLnqe" width="808" height="311" align="center"/>

torch271：

<image token="NeHzbjII3othZMxfJwacvfwYn1o" width="808" height="259" align="center"/>

<image token="RkOFbWpUcoauUWxYLl2cJ9jknkd" width="1421" height="120" align="center"/>

torch290：

<image token="Sgykb734koO4qlxZlfXcPh1mnyc" width="648" height="216" align="center"/>

### ubuntu20.04.1

测试步骤同4.1.1.1

docker run -it --privileged --group-add video   -v /public/home/zhumc/vllm-test/das18test/18whltest:/public/home/zhumc/vllm-test/das18test/18whltest -v /public/opendas/ArchivedFile/dtk-pkg/dtk26.04:/public/opendas/ArchivedFile/dtk-pkg/dtk26.04:ro   -v /opt/hyhal:/opt/hyhal:ro  --shm-size=32G --network=host --name zmc-ubuntu sugonhub/ubuntu:20.04.1-dev bash

apt-get update

apt-get install libgfortran5

安装dtk-ubuntu：
```bash {wrap}
cp /public/opendas/ArchivedFile/dtk-pkg/dtk26.04/DTK-26.04-rc4-ubuntu20.04-x86_64.tar.gz  /opt/
cd /opt/
tar -zxvf DTK-26.04-rc4-ubuntu20.04-x86_64.tar.gz
ln -s dtk-26.04-rc4  dtk
source /opt/dtk/env.sh
rocm-smi
```

测试结果

vllm011-0305:

<image token="ELEmbOoQEox5IHxsS2FcHcBKnpZ" width="813" height="314" align="center"/>

torch271：

<image token="AaZNbZ0SuoJwYZxPaeucRVaunvv" width="818" height="255" align="center"/>

<image token="TDPJbekWnoHWVixuFvDcGYranHf" width="1415" height="129" align="center"/>

torch290：

<image token="GZPtbhuQpogjfMxVsXactYyRnhh" width="714" height="216" align="center"/>

### nfs4.0

测试步骤同4.1.1.1

docker run -it --privileged --group-add video   -v /public/home/zhumc/vllm-test/das18test/18whltest:/public/home/zhumc/vllm-test/das18test/18whltest -v /public/opendas/ArchivedFile/dtk-pkg/dtk26.04:/public/opendas/ArchivedFile/dtk-pkg/dtk26.04:ro   -v /opt/hyhal:/opt/hyhal:ro  --shm-size=32G --network=host  --name zmc-nfs  sugonhub/nfs:4.0-dev bash

cp -r zstd-dev /

cd /zstd-dev/

make

make install -j 16

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

<image token="SvDkb3tOzoLFquxlJRFcQa8pn5d" width="808" height="555" align="center"/>

torch271：

<image token="U2lob5YJNowlHNxXC9ScwJx5nLe" width="646" height="736" align="center"/>

<image token="IWQab3rhboG4ycxNuU9cKFkQnne" width="1411" height="232" align="center"/>

torch290：

<image token="WE07bA0OMoWPJQxGlHNcH37Ynof" width="639" height="461" align="center"/>

### openeuler22.03

测试步骤同4.1.1.1

docker run -it --privileged --group-add video   -v /public/home/zhumc/vllm-test/das18test/18whltest:/public/home/zhumc/vllm-test/das18test/18whltest -v /public/opendas/ArchivedFile/dtk-pkg/dtk26.04:/public/opendas/ArchivedFile/dtk-pkg/dtk26.04:ro   -v /opt/hyhal:/opt/hyhal:ro  --shm-size=32G --network=host --name zmc-openeuler  sugonhub/openeuler:22.03-dev bash

yum install kmod -y

<image token="NdGwbUmEOoisLwxP8W9cIPCXnzf" width="811" height="334" align="center"/>

torch271：

<image token="B1t0bzyAiopgChxs43ycxTlbnMc" width="823" height="257" align="center"/>

<image token="JbChbEiiwoyW5kxIhN5cP7y0nGc" width="1441" height="120" align="center"/>

torch290：

<image token="LiPeb9l78oBCFqxGhQJcPE4Ynif" width="642" height="233" align="center"/>

### rocklinux8.6

docker run -it --privileged --group-add video   -v /public/home/zhumc/vllm-test/das18test/18whltest:/public/home/zhumc/vllm-test/das18test/18whltest -v /public/opendas/ArchivedFile/dtk-pkg/dtk26.04:/public/opendas/ArchivedFile/dtk-pkg/dtk26.04:ro   -v /opt/hyhal:/opt/hyhal:ro  --shm-size=32G --network=host  --name zmc-rocky image.sourcefind.cn:5000/dcu/admin/base/dev:rockylinux8.6-mpi5.0-gcc10.3-cmake3.29-py3.10-mkl2020.4.304 bash

<image token="Sl8sbOpfvotDYVxBp2fcMKMKn6e" width="850" height="304" align="center"/>

torch271：

<image token="Z1DwbHbkZoUTRhxEzRYcUctNncf" width="813" height="260" align="center"/>

<image token="TcbJbC1TDoXpytxeIRFc0Cz9n7f" width="1441" height="117" align="center"/>

torch290：

<image token="EwqRbAzgboYiXTxvaK6cTqRvntc" width="710" height="224" align="center"/>

### tencentos3

docker run -it --privileged --group-add video   -v /public/home/zhumc/vllm-test/das18test/18whltest:/public/home/zhumc/vllm-test/das18test/18whltest -v /public/opendas/ArchivedFile/dtk-pkg/dtk26.04:/public/opendas/ArchivedFile/dtk-pkg/dtk26.04:ro   -v /opt/hyhal:/opt/hyhal:ro  --shm-size=32G --network=host  --name zmc-tencent3  tencentos/tencentos3:latest bash

需要先安装SSL开发库：sudo yum install openssl-devel libffi-devel zlib-devel bzip2-devel sqlite-devel -y

安装lzma开发包：sudo yum install xz-devel

安装NUMA开发库： sudo yum install numactl-devel

安装Open MPI 开发包：sudo yum install openmpi openmpi-devel -y

sudo find / -name "libmpi.so.40" 2>/dev/null

export LD_LIBRARY_PATH=/usr/lib64/openmpi/lib:$LD_LIBRARY_PATH

sudo ldconfig

<image token="H8PtbvYR2o8d5yxXbBWcGKDwnwg" width="818" height="328" align="center"/>

torch271：

<image token="T18sbi3TyoA84kxn29sc7Fo4nfh" width="830" height="252" align="center"/>

<image token="Ct7ab6dYzoMa1Vxej0IcwwkDnhg" width="1437" height="136" align="center"/>

torch290：

<image token="WwS9boMbqo53TdxDOxwcHrX8nIc" width="657" height="226" align="center"/>

## 测试结果和问题

测试结果：

<lark-table rows="4" cols="7" column-widths="136,100,139,93,139,100,100">

  <lark-tr>
    <lark-td>
    </lark-td>
    <lark-td>
      kylinv10
    </lark-td>
    <lark-td>
      ubuntu20.04.1
    </lark-td>
    <lark-td>
      nfs4.0
    </lark-td>
    <lark-td>
      openeuler22.03
    </lark-td>
    <lark-td>
      rocklinux8.6
    </lark-td>
    <lark-td>
      tencentos3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      vllm011-0305
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      torch271
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      torch290
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
    <lark-td>
      通过
    </lark-td>
  </lark-tr>
</lark-table>

问题记录：无

## 测试总结

das1.8全面适配glibc2.28环境，测试系统：Ubuntu20.04.1、Rockylinux8.6、Openeuler22.03、Nfs4、Kylinv10、tencentos3

## 关联资源

- [[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1发版测试--c5gU8nud|DAS1.8+DTK26.04-RC1发版测试]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/sheets/JsqTsIrqihHEKotzS7Yc5gU8nud)
- [[飞书知识图谱/资源/DAS1.8发版模型测试过程数据——数据展示平台--c7dfznIf|DAS1.8发版模型测试过程数据——数据展示平台]] · 飞书源链接不可用
- [[飞书知识图谱/资源/DAS1.8+DTK26.04-RC1测试过程数据【sglang】--cz1KinDh|DAS1.8+DTK26.04-RC1测试过程数据【sglang】]] · 飞书源链接不可用
- [[飞书知识图谱/文档/VLLM测试指导（QTeam）--cJOhOnug|VLLM测试指导（QTeam）]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/KwaLdxOLmoFnysxWuMqcJOhOnug)
- [[飞书知识图谱/文档/Sglang测试指导（QTeam）--chGjinEd|Sglang测试指导（QTeam）]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/Q3YVdFn9Koy0PRxhf5uchGjinEd)

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1|DeepSeek-R1]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-INT8|DeepSeek-R1-Channel-INT8]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B|DeepSeek-R1-Distill-Llama-70B]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B-main|DeepSeek-R1-Distill-Llama-70B-main]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3-W4A8-V2-tx|DeepSeek-V3-W4A8-V2-tx]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3.2-bf16|DeepSeek-V3.2-bf16]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3.2-Channel-INT8|DeepSeek-V3.2-Channel-INT8]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/llama2-13b|llama2-13b]]（model）
- [[飞书知识图谱/实体/Llama2-13b|Llama2-13b]]（model）
- [[飞书知识图谱/实体/llama2-7b|llama2-7b]]（model）
- [[飞书知识图谱/实体/Llama2-7b|Llama2-7b]]（model）
- [[飞书知识图谱/实体/Locust|Locust]]（tool）
- [[飞书知识图谱/实体/Megatron-LM|Megatron-LM]]（framework）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OpenCompass|OpenCompass]]（framework）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/qwen1.5-14b|qwen1.5-14b]]（model）
- [[飞书知识图谱/实体/Qwen1.5-14b|Qwen1.5-14b]]（model）
- [[飞书知识图谱/实体/qwen1.5-32b|qwen1.5-32b]]（model）
- [[飞书知识图谱/实体/Qwen1.5-32b|Qwen1.5-32b]]（model）
- [[飞书知识图谱/实体/qwen2.5-VL-72B|qwen2.5-VL-72B]]（model）
- [[飞书知识图谱/实体/Qwen2.5-VL-72B-Instruct|Qwen2.5-VL-72B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen2.5-VL-72B-Instruct-quantized.w8a8|Qwen2.5-VL-72B-Instruct-quantized.w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-235B-A22B|Qwen3-235B-A22B]]（model）
- [[飞书知识图谱/实体/Qwen3-30B|Qwen3-30B]]（model）
- [[飞书知识图谱/实体/Qwen3-30B-A3B|Qwen3-30B-A3B]]（model）
- [[飞书知识图谱/实体/Qwen3-32B|Qwen3-32B]]（model）
- [[飞书知识图谱/实体/Qwen3-32B-fp16|Qwen3-32B-fp16]]（model）
- [[飞书知识图谱/实体/Qwen3-8B|Qwen3-8B]]（model）
- [[飞书知识图谱/实体/qwen3-next|qwen3-next]]（model）
- [[飞书知识图谱/实体/qwen3-VL-32B|qwen3-VL-32B]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-32B-Instruct|Qwen3-VL-32B-Instruct]]（model）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/SLA|SLA]]（metric）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

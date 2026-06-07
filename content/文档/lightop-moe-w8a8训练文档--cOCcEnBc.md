---
title: lightop-moe-w8a8训练文档
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type:
  - 崩溃
source: feishu
feishu_token: JpvddOk9ooRUFHxZiLDcOCcEnBc
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/JpvddOk9ooRUFHxZiLDcOCcEnBc
last_synced_at: 2026-06-07T17:07:53.393Z
---

# lightop-moe-w8a8训练文档

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/JpvddOk9ooRUFHxZiLDcOCcEnBc)

# moe-w8a8介绍

moe_w8a8目前支持：

- channle_wise_w8a8_int8
- channel_wise_w8a8_fp8
- 支持平台：BMZ, NMZ_64cu, NMZ_72cu
- **lightop仓库链接：** http://42.228.13.241:10068/dcutoolkit/deeplearing/lightop

# 运行调用过程

## 运行流程
```plaintext {wrap}
#step1 
通过下面函数接口获取信息，是否支持输入的shape和对应的moe_gemm1_gemm2的配置列表
对应的路径lightop/lightop/configs里寻找是否有支持的config

cuda_config1, cuda_config2, status = get_moe_cuda_marlin_config(
                E, m, n1, k1, n2, k2, topk, device_name, num_cus, compute_type
            )
            
            
            
#step2  调用函数接口运行

moe_gemm_marlin_w8a8(qinput1, w1_marlin, intermediate_cache1, qa1_scale, weight1_scale,
                  None, sorted_token_ids, expert_ids, num_tokens_post_padded, topk, cuda_config1)
                  
   
```

## 测试性能和精度脚本

脚本路径：lightop/test/moe_w8a8/release_moe_w8a8_marlin.py

测试指令：
```sql {wrap}
测试gemm1: [E, N, K] -> [160, 384, 5120]
测试gemm2: [E, N, K] -> [160, 5120, 192]
m_list  :  输入的bs
topk    :  每个token选择的专家


HIP_VISIBLE_DEVICES=5  python3 ./release_moe_w8a8_marlin.py     \
        --E [160] \
        --n1-list '[384]' \
        --n2-list '[5120]' \
        --k-list '[192]' \
        --m-list '[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,32,64,128]' \
        --topk 8 \
        --out-dtype bfloat16 \
        --use-channel-wise \
        --output-file GLM_tp8_new.xlsx
```

# 训练过程

**脚本路径：**lightop/test/moe_w8a8/benchmark_moe_int8_marlin.py

**训练指令：**
```plaintext {wrap}
HIP_VISIBLE_DEVICES=5 python benchmark_moe_int8_marlin.py --tune
```

**修改输入参数：**

比如训练qwen3.5:

- 修改文件 lightop/test/moe_w8a8/benchmark_moe_int8_marlin.py
- 对应的gemm1_shape: [E, N, K] -> [256, 2048, 3072]
- batch_size: 输入bs，一般为1-8192

<image token="IejbbZ0DuolpvqxBC77cGhYsnFA" width="1456" height="891" align="center"/>

**config过滤**

针对k不为128倍数的情况下，部分config可能崩溃，需要下面的gemm1_list和gemm2_list：

- 比如k为192 打开755行的config过滤

<image token="Q4p7b2IJYolv98xl1GecHavvnYg" width="1535" height="1047" align="center"/>

#### 运行

- 运行指令
  ```plaintext
  python benchmark_moe_int8_marlin.py --tune
  
  训练完后会生成对应的config.json文件
  MOE_BLOCKINT8_CUDA_MARLIN_512_3072_GFX936_CU80.json
  MOE_BLOCKINT8_CUDA_MARLIN_3072_256_GFX936_CU80.json
  ```

- config代表的数字含义：
	- 表示对应的数字和具体滑块的映射
	- 映射表： lightop/lightop/cscr/moew8a8/moe_w8a8_config.h

**结果放入：**

- 训练完后生成两个json文件：
	- 后缀: GFX938->NMZ  GFX936->BMZ
	- 后缀：CU80, CU72, CU64 -> 代表各个使用卡的CU数量

<image token="ZxqPbAIwcoVRO0xuRCkcNGgNn8b" width="704" height="583" align="center"/>

- 放入对应的路径：lightop/lightop/configs
- 再编译出包即可

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/崩溃|崩溃]]（issue）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

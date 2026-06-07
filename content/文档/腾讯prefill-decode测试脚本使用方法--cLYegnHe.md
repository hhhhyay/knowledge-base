---
title: 腾讯prefill-decode测试脚本使用方法
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-V3
  - GLM-5-W8A8
  - glm5-w8a8
  - GLM5-w8a8
  - Kimi-2.5-int4
  - Kimi-K2.5
  - Minimax-m2.5
  - MiniMax-M2.5-W8A8
  - Qwen-72B
framework:
  - SGLang
  - vLLM
issue_type:
  - 精度问题
  - OOM
source: feishu
feishu_token: UfJnwQdjhiz4lakOBY4cLYegnHe
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/UfJnwQdjhiz4lakOBY4cLYegnHe
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 腾讯prefill-decode测试脚本使用方法

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/UfJnwQdjhiz4lakOBY4cLYegnHe)

# 脚本与测试说明

**安装依赖**：pip install aiohttp numpy matplotlib 

测试过程数据[[飞书知识图谱/资源/云一prefill&amp;decode测试过程数据--cQB2xnPd|云一prefill&decode测试过程数据]]

**README**:[[飞书知识图谱/资源/README_decode.md--cdg35n9d|README_decode.md]][[飞书知识图谱/资源/README_prefill.md--cJDGpnDe|README_prefill.md]]，此部分有对应脚本详细的解释

脚本：

<block-ref id="CXKvdDd2qoUe7VxeYjDcFhynnSg"/>

<file token="YvDvb4WU1o5WJuxssDccYIWknhc" name="prefill_throughput.py"/>

<block-ref id="Tqc1dkK0doNCWyxLAi0cGS9Vn8f"/>

<file token="A5R9blgfGoyyGYxmlQlcIo7qn6b" name="decode_throughput.py"/>

### 使用方法（腾讯提供）

python3 ../prefill_throughput.py --base-url [http://127.0.0.1:30002](http%3A%2F%2F127.0.0.1%3A30002%2F) --model R1-0528 --max-input-length-k 128

python3 ../decode_throughput.py --base-url [http://127.0.0.1:30002](http%3A%2F%2F127.0.0.1%3A30002%2F) --model R1-0528 --max-input-length-k 128 --max-total-kvcache-length-k 500 --max-batch-size 128

### 测试方式

1. **目前测试统一默认tp8。**(如果测试decode时，用到是dp方案，如dp8，需要在跑decode_throughput.py 时加上 **--dp-size=8**，其他的以此类推)
1. 测试出现oom的情况，可以视情况**减少max-input-length-k**，但至少需要覆盖到**45k(GLM5-w8a8)/50k(Minimax-m2.5)/53k(kimi2.5)**
1. **如果非pd分离起，**<text bgcolor="light-yellow">**需要分别起服务**</text><text bgcolor="light-yellow">，例如</text><text bgcolor="light-yellow">**先起tp8的服务测试prefill_throughput.py**</text><text bgcolor="light-yellow">，</text><text bgcolor="light-yellow">**再起dp8(tp8)的服务测试decode_throughput.py**</text>** **<text bgcolor="red">**（重要）**</text>

### 基本用法
```shell {wrap}
python decode_throughput.py \
    --base-url http://127.0.0.1:8080 \
    --model DeepSeek-V3 \
    --max-input-length-k 128 \
    --max-total-kvcache-length-k 300 \
    --max-batch-size 32
```

### 自定义测试范围

```shell {wrap}
python decode_throughput.py \
    --base-url http://127.0.0.1:8080 \
    --model Qwen-72B \
    --min-input-length-k 4 \
    --max-input-length-k 256 \
    --max-input-length-step-k 64 \
    --min-batch-size 4 \
    --max-batch-size 64 \
    --max-batch-size-step 16 \
    --num-repeats 5
```

### 仅重新绘图
```shell {wrap}
python decode_throughput.py \
    --replot \
    --result-path ./results/2024-01-01-12-00-00_decode_tpot_results.json \
    --save-path ./results
    
```

# 具体测试方法（实测，nmz&h20）：

<text bgcolor="red">**为保证GLM5能覆盖到45k输入场景，--max-input-length-k 设置成50；minimax 覆盖50k输入场景，--max-input-length-k 设置成55；kimi覆盖53k输入场景--max-input-length-k 设置成55；**</text>

<callout emoji="trophy" background-color="light-orange" border-color="light-orange">

**重要：decode测试时，--max-total-kvcache-length-k 设置需要根据server log中实际可用kv cache容量适当调大，示例：glm5-w8a8实际可用容量约550k，--max-total-kvcache-length-k可设置550k充分利用大显存优势；其余模型根据自身情况设置。**
</callout>

<image token="JShxbhybqo0aboxnahCc980snFL" width="1006" height="279" align="center"/>

## GLM5-w8a8

#### server:
```bash {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
export VLLM_TORCH_PROFILER_DIR=/prof  
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7     
export ALLREDUCE_STREAM_WITH_COMPUTE=1           
export NCCL_MIN_NCHANNELS=16                      
export NCCL_MAX_NCHANNELS=16                   
export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1   
export VLLM_RPC_TIMEOUT=1800000
export VLLM_USE_PIECEWISE=1 # 需暂时打开piecewise，fullgraph有精度问题   
export VLLM_REJECT_SAMPLE_OPT=1 # 宽松采样，提高mtp接受率从而提高tpot性能，略微影响精度 default 1
#===============融合算子=================
export USE_FUSED_RMS_QUANT=0 #default 0
export USE_FUSED_SILU_MUL_QUANT=1 #default 0
#===============共享专家融合 已知精度异常=============
export VLLM_ROCM_USE_AITER=0 #default 0
export VLLM_ROCM_USE_AITER_MOE=0 #default 0
export VLLM_ROCM_USE_AITER_FUSION_SHARED_EXPERTS=0 #default 0
#注意：--compilation-config中需要添加"custom_ops": ["all", "+rms_norm"]
export VLLM_SPEC_DECODE_EAGER=0 #MTP使用eager下发default 0
export VLLM_USE_GLOBAL_CACHE13=1 #减少显存碎片化 default 0
export VLLM_FUSED_MOE_CHUNK_SIZE=16384 #default 32768
export VLLM_CUSTOM_CACHE=1 #default 1
export VLLM_USE_OPT_CAT=1
export VLLM_USE_FUSED_FILL_RMS_CAT=1 #default 1(mtp可能存在精度问题)
#export VLLM_USE_LIGHTOP_MOE_SUM_MUL_ADD=0 #w4a16/awq # 需要关闭
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=0 # 不能和kvfp8一起开
export VLLM_USE_V32_ENCODE=1
export VLLM_USE_FLASH_MLA=1
export VLLM_DISABLE_DSA=0
export USE_LIGHTOP_TOPK=1
export USE_LIGHTOP_PER_TOKEN_GROUP_QUANT_FP8=1
export USE_LIGHTOP_CONVERT_REQ_INDEX_TO_GLOBAL_INDEX=1

#export LD_LIBRARY_PATH=/workspace/hipblaslt-install/lib:$LD_LIBRARY_PATH
export VLLM_USE_LIGHTOP_MOE_SUM_MUL_ADD=1
MODEL_PATH=/model/vllm-fp8-models/GLM-5-W8A8/

vllm serve "$MODEL_PATH" \
    -q slimquant_marlin \
    --port 9349 \
    --trust-remote-code \
    --dtype bfloat16 \
    -tp 8 \
    --max-model-len 72000 \
    --gpu-memory-utilization 0.92 \
    --disable-log-requests \
    --enable-chunked-prefill \
    --max-num-batched-tokens 16384 \
    --enable-prefix-caching \
    --kv-cache-dtype fp8_ds_mla \
    -cc '{"pass_config": {"fuse_act_quant": false}}' \
    --speculative_config '{"method": "mtp", "num_speculative_tokens": 2, "quantization": "slimquant_marlin"}'

```

#### prefill
```shell {wrap}
python prefill_throughput.py \
    --base-url http://127.0.0.1:9349 \
    --model /model/vllm-fp8-models/GLM-5-W8A8/ \
    --max-input-length-k 50
```

#### prefill预计结果：（<text bgcolor="red">示例，仅测试功能</text>）

<image token="CVnBbgO5HoLX8JxM77NcuckInlh" width="1000" height="500" align="center"/>

#### decode:
```shell {wrap}

python decode_throughput.py     --base-url http://127.0.0.1:9349     --model /model/vllm-fp8-models/GLM-5-W8A8/     --max-input-length-k 50     --max-total-kvcache-length-k 500     --max-batch-size 32  
 #dp16ep16
python decode_throughput.py     --base-url http://127.0.0.1:8000    --model /module/GLM-5-W8A8/     --max-input-length-k 50     --max-total-kvcache-length-k 1500     --max-batch-size 10 --dp-size=16
```

#### decode预计结果：（示例,nmz）

<image token="Ftm4brBJdohA02xtlTecUIR1nNg" width="1000" height="500" align="center"/>

#### decode-自定义bs计算:
```shell {wrap}
#tp8
python decode_throughput.py     --base-url http://127.0.0.1:9349     --model /model/vllm-fp8-models/GLM-5-W8A8/     --max-input-length-k 50       --min-batch-size 8   --max-batch-size-step 1 --max-batch-size 16  --max-total-kvcache-length-k 550   
#dp8
python decode_throughput.py     --base-url http://127.0.0.1:9349     --model /model/vllm-fp8-models/GLM-5-W8A8/     --max-input-length-k 50       --min-batch-size 1   --max-batch-size-step 2 --max-batch-size 16  --max-total-kvcache-length-k 295
```

#### decode预计结果：

<image token="QyrZb5BdHobwBWx5gzTchwKtnI9" width="1000" height="500" align="center"/>

## Minimax-m2.5

#### server:
```shell {wrap}

```

#### prefill
```shell {wrap}
python prefill_throughput.py \
    --base-url http://127.0.0.1:8728 \
    --model /mnt/mode/MiniMax-M2.5-W8A8 \
    --max-input-length-k 55

python prefill_throughput.py \
    --base-url http://127.0.0.1:8728 \
    --model /module/MiniMax-M2.5-W8A8 \
    --max-input-length-k 55
#sglang
python prefill_throughput.py \
    --base-url http://127.0.0.1:30000 \
    --model /mnt/mode/MiniMax-M2.5-W8A8 \
    --max-input-length-k 55
    
 #20
 python3 prefill_throughput.py \
    --base-url http://127.0.0.1:8000 \
    --model /models/Mimimax-M2.5-w8a8 \
    --max-input-length-k 55
```

<image token="KX7Jb1r5PoyO3vx1hT1cYl5vnU3" width="1000" height="500" align="center"/>

#### decode:
```shell {wrap}
python decode_throughput.py \
--base-url http://127.0.0.1:8728 \
--model /mnt/mode/MiniMax-M2.5-W8A8 \
--max-input-length-k 55 \
--max-total-kvcache-length-k 810 \
--max-batch-size 3

python decode_throughput.py \
--base-url http://127.0.0.1:8728 \
--model /mnt/mode/MiniMax-M2.5-W8A8 \
--max-input-length-k 55 \
--min-batch-size 32   \
--max-batch-size-step 4 \
--max-batch-size 48  \
--max-total-kvcache-length-k 6400

#Bw1000
python decode_throughput.py \
--base-url http://127.0.0.1:8728 \
--model /module/MiniMax-M2.5-W8A8 \
--max-input-length-k 55 \
--max-total-kvcache-length-k 1800 \
--max-batch-size 3

python decode_throughput.py \
--base-url http://127.0.0.1:8728 \
--model /module/MiniMax-M2.5-W8A8 \
--max-input-length-k 55 \
--min-batch-size 32   \
--max-batch-size-step 4 \
--max-batch-size 48  \
--max-total-kvcache-length-k 1800 

#sglang
python decode_throughput.py \
--base-url http://127.0.0.1:30000 \
--model /mnt/mode/MiniMax-M2.5-W8A8 \
--max-input-length-k 55 \
--max-batch-size 128 \
--max-total-kvcache-length-k 2400 \ #log/1024 #cache size token 


python decode_throughput.py \
--base-url http://127.0.0.1:30000 \
--model /mnt/mode/MiniMax-M2.5-W8A8 \
--max-input-length-k 55 \
--min-batch-size 1 \
--max-batch-size 32 \
--max-total-kvcache-length-k 2400 \ #log/1024 #cache size token 
--dp-size 2 #tp不需要

#H20
python3 decode_throughput.py \
--base-url http://127.0.0.1:8000 \
--model /models/Mimimax-M2.5-w8a8 \
--max-input-length-k 55 \
--max-total-kvcache-length-k 500 \
--max-batch-size 32
```

<image token="BYUTbspXuoqq4ixC6Wdcfnn0nTX" width="1000" height="500" align="center"/>

## Kimi-2.5-int4

#### server:
```bash {wrap}
#!/bin/bash
rm -rf ~/.cache
rm -rf ~/.triton

export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3

export VLLM_TORCH_PROFILER_DIR=/mnt/kimi2.5/prof/
export VLLM_USE_LIGHTOP=1
export VLLM_USE_PIECEWISE=0
export VLLM_USE_LIGHTOP_FILL_MOE_ALIGN=1
export VLLM_1D_MROPE=1
export USE_FUSED_SILU_MUL_QUANT=0
export USE_FUSED_RMS_QUANT=0
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=1
export VLLM_USE_FUSED_FILL_RMS_CAT=0
export VLLM_USE_LIGHTOP_FUSED_TOPP_TOPK=1
export VLLM_USE_FUSED_RMS_ROPE=1
export VLLM_W8A8_BACKEND=3
export MOE_NN=0
export VLLM_V1_USE_FUSED_QKV_SPLIT_RMS_ROPE_KVSTORE=0
export VLLM_USE_FLASH_ATTN_FP8=1
export LD_LIBRARY_PATH=/opt/dtk-26.04/rocblas-install/lib:$LD_LIBRARY_PATH

model_path=/model/Kimi-K2.5
model=${model_path##*/}
tp=8
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
port=8032
logpath="./server/${model}-tp${tp}-$time"
data_type="bfloat16"
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

vllm serve $model_path \
    --port $port \
    --max-model-len 81920 \
    --dtype $data_type \
    -tp $tp \
    --trust-remote-code \
    --gpu-memory-utilization 0.9 \
    --enable-prefix-caching \
    2>&1 | tee  ${logpath}-$time.log

```

#### prefill
```shell {wrap}
python prefill_throughput.py \
    --base-url http://127.0.0.1:8032 \
    --model /model/Kimi-K2.5 \
    --max-input-length-k 55
```

#### decode:
```shell {wrap}
python decode_throughput.py     --base-url http://127.0.0.1:8032     --model /model/Kimi-K2.5     --max-input-length-k 55    --max-total-kvcache-length-k 500     --max-batch-size 32
```

## 关联资源

- [[飞书知识图谱/资源/云一prefill&amp;decode测试过程数据--cQB2xnPd|云一prefill&amp;decode测试过程数据]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/WuQ7wTf7Vic0OSkNMbKcQB2xnPd)
- [[飞书知识图谱/资源/README_decode.md--cdg35n9d|README_decode.md]] · 飞书源链接不可用
- [[飞书知识图谱/资源/README_prefill.md--cJDGpnDe|README_prefill.md]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/精度问题|精度问题]]（issue）
- [[飞书知识图谱/实体/DeepSeek-V3|DeepSeek-V3]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/glm5-w8a8|glm5-w8a8]]（model）
- [[飞书知识图谱/实体/GLM5-w8a8|GLM5-w8a8]]（model）
- [[飞书知识图谱/实体/Kimi-2.5-int4|Kimi-2.5-int4]]（model）
- [[飞书知识图谱/实体/Kimi-K2.5|Kimi-K2.5]]（model）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/Minimax-m2.5|Minimax-m2.5]]（model）
- [[飞书知识图谱/实体/MiniMax-M2.5-W8A8|MiniMax-M2.5-W8A8]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OOM|OOM]]（issue）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/Qwen-72B|Qwen-72B]]（model）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

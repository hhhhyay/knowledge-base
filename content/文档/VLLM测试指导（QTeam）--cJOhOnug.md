---
title: VLLM测试指导（QTeam）
tags:
  - AI
  - LLM
  - Benchmark
model:
  - deepseek-r1
  - DeepSeek-R1
  - DeepSeek-R1-0528-bf16
  - DeepSeek-R1-0528-W4A8-V2
  - DeepSeek-R1-bf16
  - DeepSeek-R1-Channel-INT8
  - DeepSeek-R1-Distill-Llama-70B
  - DeepSeek-R1-Distill-Llama-70B-main
  - DeepSeek-R1-Distill-Llama-70B-quantized.w8a8
  - DeepSeek-R1-Distill-Qwen-32B
  - DeepSeek-V3-W4A8-V2-tx
  - deepseek-v3.1
  - DeepSeek-V3.1-Terminus
  - DeepSeek-V3.1-Terminus-fp8
  - DeepSeek-V3.1-Terminus-fp8-tp8-vllm
  - DeepSeek-V3.2-Channel-INT8
  - GLM-4
  - GLM-4-32B-0414
  - GLM-4-32B-0414-vllm011
  - GLM-4.1V-9B-Thinking
  - Qwen2.5-VL-32B-Instruct
  - Qwen2.5-VL-72B-Instruct
  - Qwen2.5-VL-72B-Instruct-int8-vllm011
  - Qwen2.5-VL-72B-Instruct-quantized.w8a8
  - Qwen3-235B-A22B
  - Qwen3-235B-A22B-Instruct-2507
  - Qwen3-235B-A22B-Thinking-2507
  - Qwen3-235B-A22B-Thinking-2507-vllm011-660-1
  - Qwen3-235B-A22B-W8A8
  - Qwen3-30B-A3B
  - Qwen3-30B-A3B-Thinking-2507
  - Qwen3-30B-A3B-thinking-2507-W8A8
  - Qwen3-30B-A3B-Thinking-2507-w8a8
  - Qwen3-30B-A3B.w8a8
  - Qwen3-32B
  - Qwen3-32B-W8A8
  - Qwen3-32B.w8a8
  - Qwen3-4B-Thinking-2507
  - Qwen3-4B-Thinking-2507-tx-vllm011-0109
  - Qwen3-8B
  - Qwen3-8B.w8a8
  - Qwen3-Coder-480B-A35B-Instruct
  - Qwen3-VL-235B-A22B-Instruct
  - Qwen3-VL-235B-A22B-Thinking
  - Qwen3-VL-235BB-A22B-Thinking
  - Qwen3-VL-30B-A3B-Thinking
  - Qwen3-VL-30B-Thinking
  - Qwen3-VL-4B-Instruct
  - Qwen3-VL-4B-Thinking
  - Qwen3-VL-4B-Thinking-0109
  - Qwen3-VL-8B-thinking
  - Qwen3-VL-8B-Thinking
framework:
  - PyTorch
  - vLLM
issue_type:
  - OOM
source: feishu
feishu_token: KwaLdxOLmoFnysxWuMqcJOhOnug
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/KwaLdxOLmoFnysxWuMqcJOhOnug
last_synced_at: 2026-06-07T17:07:53.393Z
---

# VLLM测试指导（QTeam）

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/KwaLdxOLmoFnysxWuMqcJOhOnug)

（1）模型为二级标题；测试模式为三级标题；加速卡为四级标题；

（2）对应模型需要备注编写人员；

（3）模型名称需要与社区命名一致；如果是量化模型需要明确是社区量化还是内部量化；

（4）如果不同加速卡有测试区别，需要在不同加速卡下明确，相同部分全部放在加速卡上统一编写；

（5）总结出来测试方法，不明确具体镜像；

（6）注意编写格式，如使用代码块，编写整洁；

# 导航

<add-ons component-id="" component-type-id="blk_637dcc698597401c1a8fd711" record="{"ignoreCataLogRecordIds":[],"isShowAllLevel":false,"showCataLogLevel":2,"viewType":"normal"}"/>

## 统一设置

## DeepSeeek-V3-W4A8-V2-tx（内部量化）【王昊宇】

### 测试模式
```bash {wrap}
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1 

vllm serve /model/DeepSeek-V3-W4A8-V2-tx \
 --port $port \
 --trust-remote-code \
 --dtype bfloat16 \
 -tp 8 \
 --disable-cascade-attn 
```

#### BW1000
```bash {wrap}
--kv-cache-dtype fp8_e5m2
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3
```

#### K100_AI

暂无

## Qwen3-30B-A3B【王昊宇】

### 测试模式
```bash {wrap}
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


vllm serve /model/Qwen3-30B-A3B \
        --dtype float16 \
        --trust-remote-code \
        --tensor-parallel-size 2 \
        --disable-cascade-attn
```

#### BW1000
```bash {wrap}
--kv-cache-dtype fp8_e5m2 
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 \
-tp 1
```

#### K100_AI

暂无

## Qwen3-8B【王昊宇】

### 测试模式
```bash {wrap}
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


vllm serve /model/Qwen3-8B \
        --dtype float16 \
        --trust-remote-code \
        --tensor-parallel-size 1 \
        --disable-cascade-attn
```

#### BW1000
```bash {wrap}
--kv-cache-dtype fp8_e5m2 
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 
```

#### K100_AI

暂无

## Qwen3-8B.w8a8（内部量化）【王昊宇】

### 测试模式
```bash {wrap}
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
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1

vllm serve /model/Qwen3-8B.w8a8 \
        --dtype float16 \
        --tensor-parallel-size 1 \
        --disable-cascade-attn 
```

#### BW1000
```bash {wrap}
--kv-cache-dtype fp8_e5m2 
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 
```

#### K100_AI

暂无

## Qwen3-30B-A3B.w8a8(社区量化)【王昊宇】

### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export HSA_FORCE_FINE_GRAIN_PCIE=0
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1

vllm serve /model/qwen3/Qwen3-30B-A3B.w8a8 \
 --dtype bfloat16 \
 -tp 2 \
 --trust-remote-code \
 --disable-cascade-attn 

```

#### BW1000
```bash {wrap}
--kv-cache-dtype fp8_e5m2 \
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 \
-tp 1
```

#### K100_AI

暂无

## Qwen3-VL-235B-A22B-Thinking【王昊宇】

### 测试模式
```bash {wrap}
export VLLM_HOST_IP=12.12.12.74
export NCCL_SOCKET_IFNAME=ib0
export GLOO_SOCKET_IFNAME=ib0
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_IB_HCA=mlx5_0:1,mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000

vllm serve /model/Qwen3-VL-235BB-A22B-Thinking \
        --dtype float16 \
        --disable-cascade-attn \
        --distributed-executor-backend ray \
        --tensor-parallel-size 16
```

#### BW1000
```bash {wrap}
--kv-cache-dtype fp8_e5m2 
```

#### NMZ
```bash {wrap}
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0 #根据实际情况绑定
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3

vllm serve /model/Qwen3-VL-235BB-A22B-Thinking  \
        --dtype float16 \
        --disable-cascade-attn \
        --tensor-parallel-size 8
```

#### K100_AI

暂无

## Qwen3-235B-A22B【石鹏飞】

### 测试模式
```bash
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export VLLM_NUMA_BIND=1
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000
export NCCL_NET_GDR_LEVEL=7
export NCCL_SDMA_COPY_ENABLE=0
export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_PD_SPLIT=1

model_path=/model/qwen3/Qwen3-235B-A22B
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="float16"
tp=8
port=8816
gpu_memory=0.95

log_date=$(date "+%Y-%m-%d")
log_dir="xxx_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --dtype ${data_type} \
 --host 0.0.0.0 \
 --port ${port} \
 --trust-remote-code \
 -tp ${tp} \
 --max-model-len 32768 \
 --distributed-executor-backend mp \
 --gpu-memory-utilization $gpu_memory \
 --no-enable-prefix-caching \
 --no-enable-chunked-prefill \
 2>&1 | tee "${log_dir}/serve_${time}.log"
```

#### BW1000
```bash {wrap}
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4

--kv-cache-dtype fp8_e5m2
```

#### NMZ
```bash
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3

gpu_memory=0.9
--kv-cache-dtype fp8_e4m3
```

#### K100_AI
```bash
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
```

## Qwen3-235B-A22B-W8A8（内部量化）【石鹏飞】

### 测试模式
```shell
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export VLLM_NUMA_BIND=1
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000
export NCCL_NET_GDR_LEVEL=7
export NCCL_SDMA_COPY_ENABLE=0
export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_PD_SPLIT=1

model_path=/model/vllm-w8a8-models/Qwen3-235B-A22B-W8A8
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="bfloat16"
tp=8
port=8878
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
log_dir="${bwtype}_${model}/${log_date}"
mkdir -p "${log_dir}"
vllm serve ${model_path} \
 --dtype ${data_type} \
 --host 0.0.0.0 \
 --port ${port} \
 --trust-remote-code \
 -tp ${tp} \
 --max-model-len 40960 \
 --gpu-memory-utilization $gpu_memory \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"
```

#### BW1000
```shell
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3

bwtype="bw1000"
--kv-cache-dtype fp8_e5m2 \
```

#### NMZ
```shell
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3

bwtype="nmz"
--kv-cache-dtype fp8_e4m3 \
```

#### K100_AI
```shell
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
#k100ai
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_W8A8_BACKEND=1

gpu_memory=0.95
bwtype="k100ai"
```

## Qwen3-VL-235B-A22B-Instruct【朱明翠】

### 测试模式

环境变量：
```bash {wrap}
export VLLM_HOST_IP=12.12.12.74
export NCCL_SOCKET_IFNAME=ib0
export GLOO_SOCKET_IFNAME=ib0
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_IB_HCA=mlx5_0:1,mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000
export NCCL_TOPO_FILE="/public/home/chenwch/zhumc/topo-input.xml"
unset NCCL_ALGO
export NCCL_NET_GDR_LEVEL=7
export NCCL_SDMA_COPY_ENABLE=0
export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_V1=1
export VLLM_ZERO_OVERHEAD=0
export RAY_CGRAPH_get_timeout=18000
export RAY_DAG_GET_TIMEOUT_S=18000
export VLLM_USE_PD_SPLIT=1
export VLLM_ENABLE_TBO=0

export VLLM_SCHED_ENABLE_MINIMAL_INJECTION=1
export VLLM_USE_FUSE_SILU_AND_MUL=0
export VLLM_USE_FUSED_RMS_ROPE=0
export VLLM_USE_LIGHTOP_FILL_MOE_ALIGN=0
export VLLM_USE_LIGHTOP_MOE_ALIGN=0
export VLLM_USE_OPT_RESHAPE_AND_CACHE=0
```

server：
```bash {wrap}
model_path=/public/opendas/DL_DATA/llm-models/Qwen3-VL-235B-A22B-Instruct
model=${model_path##*/}
tp=16
logpath="./server/Qwen3-VL-235B-A22B-Instruct/"
current_time=$(date +"%Y%m%d-%H%M")
logpath="./Qwen3-VL-235B-A22B-Instruct/${model}-tp${tp}-${current_time}"
port=8100
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
#
python3 -m vllm.entrypoints.openai.api_server \
 --model ${model_path} \
 --dtype float16 \
 --host 0.0.0.0 \
 --port ${port} \
 -tp ${tp}  \
 --gpu-memory-utilization 0.9 \
 --trust-remote-code \
 --distributed-executor-backend ray \
 --block-size 64 \
 --max-num-seq 256 \
 --max-num-batched-tokens 32768 \
 --max-model-len 32768 \
 --enable-chunked-prefill \
 --no-enable-prefix-caching   >&1 | tee  ${logpath}.log

```

#### BW1000
```plaintext {wrap}
--kv-cache-dtype fp8_e5m2
```

#### NMZ
```plaintext {wrap}
--kv-cache-dtype fp8_e4m3
-tp8
```

#### K100_AI
```bash {wrap}
暂无
```

## Qwen3-235B-A22B-Instruct-2507【石鹏飞】

### 测试模式
```bash
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000
export NCCL_NET_GDR_LEVEL=7
export NCCL_SDMA_COPY_ENABLE=0
export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_PD_SPLIT=1

model_path=/model/Qwen3-235B-A22B-Instruct-2507
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="float16"
tp=8
port=8507
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
log_dir="${bwtype}_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --dtype ${data_type} \
 --host 0.0.0.0 \
 --port ${port} \
 --trust-remote-code \
 -tp ${tp} \
 --distributed-executor-backend mp \
 --gpu-memory-utilization $gpu_memory \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"
```

#### BW1000
```bash
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4

bwtype="bw1000"
--max-model-len 61440 \
--kv-cache-dtype fp8_e5m2 \
```

#### NMZ
```shell
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3

bwtype="nmz"
--max-model-len 61440 \
--max-num-batched-tokens 204800 \
--kv-cache-dtype fp8_e4m3 \
```

#### K100_AI
```bash
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
#k100ai
export ALLREDUCE_STREAM_WITH_COMPUTE=1
bwtype="k100ai"
gpu_memory=0.95

--max-model-len 40960 \
```

## Qwen3-32B-W8A8（社区量化）【石鹏飞】

### 测试模式
```bash
rm -rf ~/.cache/
rm -rf ~/.triton/

export HIP_VISIBLE_DEVICES=6
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=5
#export VLLM_RANK1_NUMA=1
#export VLLM_RANK2_NUMA=1
#export VLLM_RANK3_NUMA=1
export VLLM_SPEC_DECODE_EAGER=1
export VLLM_ZERO_OVERHEAD=1 # zero
export VLLM_USE_PD_SPLIT=1
export VLLM_TORCH_PROFILER_DIR=/mnt/torchprof

model_path=/mnt/mode/vllm-w8a8-models/Qwen3-32B.w8a8
model=${model_path##*/}
tp=1
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="./logs-server"
data_type="fp16"
port=8258
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

#--max-num-seqs 1024
#--enforce-eager
#--kv-cache-dtype fp8_e4m3
vllm serve $model_path \
        --host 0.0.0.0 \
        --port $port   \
        --dtype  float16 \
        --tensor-parallel-size $tp   \
        --trust-remote-code  \
        --distributed-executor-backend=mp \
        --enable-prefix-caching \
        --enable-chunked-prefill \
        --max-model-len 32768 \
        --disable-cascade-attn \
        --kv-cache-dtype fp8_e4m3 \
        2>&1 | tee  ${logpath}/${model}_$time.log
```

#### BW1000（石鹏飞）

暂无

#### NMZ

待测试

#### K100_AI

## Qwen3-32B【石鹏飞】

### 测试模式
```bash
export HIP_VISIBLE_DEVICES=0,1,2,3
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_USE_PD_SPLIT=1
export VLLM_USE_OPT_ZEROS=1

model_path=/model/qwen3/Qwen3-32B
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="float16"
tp=4
port=8811
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
log_dir="bw1000_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --host 0.0.0.0 \
 --port $port \
 --gpu-memory-utilization $gpu_memory \
 --distributed-executor-backend mp \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"
```

#### BW1000（石鹏飞）

暂无

#### NMZ

待测试

#### K100_AI

## Qwen2.5-VL-72B-Instruct【石鹏飞】

### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000

model_path=/model/qwen2.5/Qwen2.5-VL-72B-Instruct
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="float16"
tp=8
port=8972
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
log_dir="bw1000_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --dtype ${data_type} \
 --host 0.0.0.0 \
 --port ${port} \
 --trust-remote-code \
 -tp ${tp} \
 --distributed-executor-backend mp \
 --gpu-memory-utilization $gpu_memory \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"

```

#### BW1000（石鹏飞）

暂无

#### NMZ

待测试

#### K100_AI

## Qwen2.5-VL-72B-Instruct-quantized.w8a8（社区量化）【胡雅晴】

### 测试模式
```bash
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000
#0114 bug修复
export LD_LIBRARY_PATH=/home/docker_data/whl_datas/0114_for1226_q2.5v72w8/hipblaslt-install/lib/:$LD_LIBRARY_PATH
export W8A8_SUPPORT_METHODS=3

model_path=/model/Qwen2.5-VL-72B-Instruct-quantized.w8a8
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="float16"
tp=8
port=8972
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
log_dir="bw1000_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --dtype ${data_type} \
 --host 0.0.0.0 \
 --port ${port} \
 --trust-remote-code \
 -tp ${tp} \
 --distributed-executor-backend mp \
 --gpu-memory-utilization $gpu_memory \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"
```

#### BW1000（石鹏飞）

暂无

#### NMZ

待测试

#### K100_AI

## DeepSeek-R1-Distill-Llama-70B【石鹏飞】

### 测试模式
```bash
export HIP_VISIBLE_DEVICES=0,1,2,3
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_USE_PD_SPLIT=1
export VLLM_USE_OPT_ZEROS=1

model_path=/model/DeepSeek-R1-Distill-Llama-70B
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="float16"
tp=4
port=8870
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
log_dir="bw100_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --dtype ${data_type} \
 --host 0.0.0.0 \
 --port ${port} \
 -tp ${tp} \
 --trust-remote-code \
 --distributed-executor-backend mp \
 --gpu-memory-utilization $gpu_memory \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"

```

#### BW1000（石鹏飞）

暂无

#### NMZ

待测试

#### K100_AI

## DeepSeek-R1-Distill-Qwen-32B【石鹏飞】

### 测试模式
```bash
export HIP_VISIBLE_DEVICES=4,5,6,7
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export VLLM_NUMA_BIND=1
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_USE_PD_SPLIT=1
export VLLM_USE_OPT_ZEROS=1

model_path=/model/DeepSeek-R1-Distill-Qwen-32B
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="float16"
tp=4
port=8132
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
log_dir="bw1000_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --host 0.0.0.0 \
 --port $port \
 --gpu-memory-utilization $gpu_memory \
 --distributed-executor-backend mp \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"
```

#### BW1000（石鹏飞）

暂无

#### NMZ

待测试

#### K100_AI

## Qwen3-VL-30B-A3B-Thinking【刘治宦】

### 测试模式

nmz使用fp8_e4m3,bw1000使用fp8_e5m2
```bash
export HIP_VISIBLE_DEVICES=6,7
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=5
export VLLM_RANK1_NUMA=4
#export VLLM_ZERO_OVERHEAD=1   # zero
export VLLM_USE_PIECEWISE=1
export PYTORCH_MIOPEN_SUGGEST_NDHWC=0
model_path=/model/Qwen3-VL-30B-A3B-Thinking/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=2
logpath="./server/vllm011/bw1000/Qwen3-VL-30B-Thinking/"
port=9229

if [ ! -d "${logpath}" ]; then
    mkdir -p "${logpath}"
fi
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --distributed-executor-backend=mp \
    --gpu-memory-utilization 0.9 \
    --no-enable-prefix-caching --enable-chunked-prefill --disable-cascade-attn >&1 | tee  ${logpath}.log
```

#### BW1000

#如需测试腾讯TFCC业务，测试相关数据集，需要按下面设置
```bash
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --distributed-executor-backend=mp \
    --limit-mm-per-prompt '{"image": 4}' \
    --gpu-memory-utilization 0.8 \
    --kv-cache-dtype fp8_e5m2 \
    --max-num-batched-tokens 20480 --no-enable-prefix-caching --enable-chunked-prefill --disable-cascade-attn >&1 | tee  ${logpath}.log
```

#### NMZ
```bash
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --distributed-executor-backend=mp \
    --limit-mm-per-prompt '{"image": 4}' \
    --gpu-memory-utilization 0.8 \
    --kv-cache-dtype fp8_e4m3 \
    --max-num-batched-tokens 20480 --no-enable-prefix-caching --enable-chunked-prefill --disable-cascade-attn >&1 | tee  ${logpath}.log
```

#### K100_AI

**区别：tp4**

## Qwen3-VL-8B-Thinking【刘治宦】

nmz使用fp8_e4m3,bw1000使用fp8_e5m2

#### 测试模式
```bash
export HIP_VISIBLE_DEVICES=4
#export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
#export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=7

model_path=../../Qwen3-VL-8B-Thinking/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=1
logpath="./tp2-server/Qwen3-VL-8B-thinking/"
port=9348
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --gpu-memory-utilization 0.9 \
    --enable-prefix-caching --disable-cascade-attn --max-model-len 244096 >&1 | tee  ${logpath}.log

```

#### BW1000

#如需测试腾讯TFCC业务，测试相关数据集，需要按下面设置

#可选择打开    --kv-cache-dtype fp8_e5m2 \
```bash
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --gpu-memory-utilization 0.9 \
    --kv-cache-dtype fp8_e5m2 \
    --enable-prefix-caching --disable-cascade-attn --max-model-len 244096 \
    --allowed-local-media-path ../../tencentDataset >&1 | tee  ${logpath}.log
```

#### NMZ
```bash
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --gpu-memory-utilization 0.9 \
    --kv-cache-dtype fp8_e4m3 \
    --enable-prefix-caching --disable-cascade-attn --max-model-len 244096 \
    --allowed-local-media-path ../../tencentDataset >&1 | tee  ${logpath}.log
```

#### K100_AI

使用tp2测试

## Qwen3-VL-4B-Instruct【刘治宦】

#### 测试模式

nmz使用fp8_e4m3,bw1000使用fp8_e5m2
```bash
export HIP_VISIBLE_DEVICES=0
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export PYTORCH_MIOPEN_SUGGEST_NDHWC=0

model_path=../../Qwen3-VL-4B-Instruct/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=1

logpath="./1218tp8-server/Qwen3-VL-4B-Instruct/"
port=9340
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --limit-mm-per-prompt '{"image": 15}' \
    --gpu_memory_utilization 0.90 \
    --no-enable-prefix-caching --disable-cascade-attn \
    2>&1 | tee  ${logpath}.log

```

#### BW1000

#可选择打开    --kv-cache-dtype fp8_e5m2 \
```bash
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --kv-cache-dtype fp8_e5m2 \
    --limit-mm-per-prompt '{"image": 15}' \
    --gpu_memory_utilization 0.90 \
    --no-enable-prefix-caching --disable-cascade-attn \
    2>&1 | tee  ${logpath}.log
```

#### NMZ

#tp1

#可选择打开    --kv-cache-dtype fp8_e4m3 \
```shell
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --kv-cache-dtype fp8_e4m3 \
    --limit-mm-per-prompt '{"image": 15}' \
    --gpu_memory_utilization 0.90 \
    --no-enable-prefix-caching --disable-cascade-attn \
    2>&1 | tee  ${logpath}.log
```

#### K100_AI
```bash {wrap}
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --limit-mm-per-prompt '{"image": 15}' \
    --gpu_memory_utilization 0.90 \
    --no-enable-prefix-caching --disable-cascade-attn \
    2>&1 | tee  ${logpath}.log
```

## GLM-4.1V-9B-Thinking【刘治宦】

#### 测试模式

nmz使用fp8_e4m3,bw1000使用fp8_e5m2
```bash
export HIP_VISIBLE_DEVICES=4
#export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
#export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0


model_path=/model/GLM-4.1V-9B-Thinking/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=1
logpath="./server/das1.7/GKM-4.1-9B/"
port=9345
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --max-seq-len-to-capture 262144 \
    --no-enable_prefix_caching \
    --max-num-seqs 1024 --distributed-executor-backend=mp   2>&1 | tee  ${logpath}.log

```

#### BW1000

#依照测试模式

#可选择打开    --kv-cache-dtype fp8_e5m2 \

#### NMZ

#依照测试模式

#可选择打开    --kv-cache-dtype fp8_e4m3 \

#### K100_AI

tp2

## DeepSeek-R1【刘治宦】

#### 测试模式

nmz-tp8,bw1000-tp16,h20-tp16
```bash
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=0
export VLLM_USE_LIGHTOP_MOE_SUM_MUL_ADD=0

#model_path=/model/DeepSeek-R1/
model_path=/model/DeepSeek-R1
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="bfloat16"
tp=8
logpath="./tp8-server/deepseek-r1/"
port=9348
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --gpu-memory-utilization 0.9 \
 --disable-cascade-attn \
 --host 0.0.0.0 \
 --port $port \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 1}' \
 --block-size 64 \
 --max-model-len 65536 \
 --no-enable-prefix-caching  2>&1 | tee "${log_dir}/serve_${time}.log"

```

#### BW1000

tp16#可选择打开    --kv-cache-dtype fp8_e5m2 \
```shell {wrap}
vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --gpu-memory-utilization 0.9 \
 --disable-cascade-attn \
 --host 0.0.0.0 \
 --port $port \
 --kv-cache-dtype fp8_e5m2
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 1}' \
 --block-size 64 \
 --max-model-len 65536 \
 --no-enable-prefix-caching  2>&1 | tee "${log_dir}/serve_${time}.log"
```

#### NMZ

NMZ测试腾讯id111数据集需要调整--max-model-len 71680 \
```bash
VLLM_USE_V1=0 vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --gpu-memory-utilization 0.9 \
 --disable-cascade-attn \
 --host 0.0.0.0 \
 --port $port \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 1}' \
 --block-size 64 \
  --kv-cache-dtype fp8_e4m3
 --max-model-len 71680 \
 --no-enable-prefix-caching  2>&1 | tee "${log_dir}/serve_${time}.log"
```

## DeepSeek-R1-Channel-INT8【刘治宦】

#### 测试模式

#### NMZ
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
export VLLM_ZERO_OVERHEAD=1 # zero
#export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=0
export VLLM_USE_LIGHTOP_MOE_SUM_MUL_ADD=0

export VLLM_USE_FUSED_QA_KVA_GEMM=0 
export VLLM_USE_FUSED_DTBMM=0

#model_path=/model/DeepSeek-R1/
model_path=/model/vllm-w8a8-models/DeepSeek-R1-Channel-INT8
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="bfloat16"
tp=8
logpath="./tp8-server/deepseek-r1/"
port=9349
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --gpu-memory-utilization 0.9 \
 --disable-cascade-attn \
 --host 0.0.0.0 \
 --port $port \
 --block-size 64 \
 --kv-cache-dtype fp8_e4m3 \
 --no-enable-prefix-caching  2>&1 | tee "${log_dir}/serve_${time}.log"

```

#### BW1000
```bash {wrap}
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_HOST_IP=$(hostname -I | awk '{print $1}')
export NCCL_SOCKET_IFNAME=ens19f0
export GLOO_SOCKET_IFNAME=ens19f0
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_IB_HCA=mlx5_1:1,mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_8:1,mlx5_9:1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_MLA_DISABLE=0
export VLLM_USE_FLASH_MLA=1
export NCCL_TOPO_FILE="/mnt/das18_test/newtopo.xml"
export VLLM_FUSED_MOE_CHUNK_SIZE=16384
export VLLM_ZERO_OVERHEAD=1 # zero
export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_PD_SPLIT=1
```


```shell {wrap}
model_path=/models/vllm-w8a8-models/DeepSeek-R1-Channel-INT8
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

```

/K100-AI
```shell {wrap}
model_path=/models/vllm-w8a8-models/DeepSeek-R1-Channel-INT8
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
--enforce-eager

```

## Qwen3-30B-A3B-Thinking-2507【朱明翠】

#### 测试模式
```bash
#!/bin/bash
export HIP_VISIBLE_DEVICES=0,1
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
#export VLLM_ZERO_OVERHEAD=1 # zero
model_path=/public/opendas/DL_DATA/llm-models/qwen3/Qwen3-30B-A3B-Thinking-2507
export VLLM_USE_PD_SPLIT=1
export VLLM_USE_V1=1
#1203bw镜像30b模型专用
export VLLM_USE_OPT_RESHAPE_AND_CACHE=1
#1211xinjia
export VLLM_USE_MARLIN_W16A16_MOE=1
export MOE_NN=0
#1215新加
export VLLM_USE_FUSED_RMS_ROPE=1
export VLLM_V1_USE_REDUCED_TOPK_TOPP_SAMPLER=1
export VLLM_CUSTOM_CACHE=0

model=${model_path##*/}
tp=2
time=$(date "+%m%d-%H%M")
mode="cudagraph"
#mode="cudagraph"
logpath="./server/${model}-${mode}-$time"
data_type="fp16"
csv_name="${model}-tp${tp}-$time"
port=8089
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

vllm serve $model_path --max-model-len 32768  --dtype float16  -tp 2 --disable-log-requests --trust-remote-code --port $port   --disable-cascade-attn 2>&1 | tee  ${logpath}.log


```

#### BW1000
```plaintext {wrap}
--kv-cache-dtype fp8_e5m2
```

#### NMZ
```plaintext {wrap}
--kv-cache-dtype fp8_e4m3
-tp 1
```

#### K100_AI
```bash {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1 #K100AI

```

## Qwen3-30B-A3B-Thinking-2507-w8a8【朱明翠】

#### 测试模式
```bash {wrap}
#!/bin/bash
export HIP_VISIBLE_DEVICES=1
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
export VLLM_ZERO_OVERHEAD=1 # zero
model_path=/public/opendas/DL_DATA/llm-models/vllm-w8a8-models/Qwen3-30B-A3B-thinking-2507-W8A8
#管宇新加
export VLLM_USE_PD_SPLIT=1
export VLLM_USE_V1=1
#1203bw镜像30b模型专用
export VLLM_USE_OPT_RESHAPE_AND_CACHE=1
#1211xinjia
export VLLM_USE_MARLIN_W16A16_MOE=1
export MOE_NN=0
#1215新加
export VLLM_USE_FUSED_RMS_ROPE=1
export VLLM_V1_USE_REDUCED_TOPK_TOPP_SAMPLER=1
export VLLM_CUSTOM_CACHE=0

model=${model_path##*/}
tp=1
time=$(date "+%m%d-%H%M")
mode="cudagraph"
#mode="cudagraph"
logpath="./server/${model}-${mode}-$time"
data_type="fp16"
csv_name="${model}-tp${tp}-$time"
port=8081
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi


vllm serve $model_path --max-num-seqs 512 --max-model-len 4096  --dtype bfloat16  -tp 1 --disable-log-requests --trust-remote-code --port $port  --no-enable-prefix-caching  --enable-chunked-prefill  --max-num-batched-tokens 32768 2>&1 | tee  ${logpath}.log

```

#### BW1000
```plaintext {wrap}
--kv-cache-dtype fp8_e5m2
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 
```

#### K100_AI
```plaintext {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1 #K100AI
```

## Qwen2.5-VL-32B-Instruct【朱明翠】

#### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_USE_OPT_ZEROS=1
model_path=/public/opendas/DL_DATA/llm-models/qwen2.5/Qwen2.5-VL-32B-Instruct
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=4
logpath="./server/das1.7/Qwen3-VL-4B-Thinking/"
port=8145
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --max-model-len 128000 --no-enable-prefix-caching --enable-chunked-prefill --distributed-executor-backend=mp  --disable-cascade-attn  >&1 | tee  ${logpath}.log


```

#### BW1000
```plaintext {wrap}
--kv-cache-dtype fp8_e5m2
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 
-tp 2
```

#### K100_AI
```plaintext {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1 #K100AI
```

## Qwen3-Coder-480B-A35B-Instruct【朱明翠】

#### 测试模式
```bash {wrap}
#!/bin/bash
# NUMA绑定，提高RCCL稳定性
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3
# 通信调优参数
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export VLLM_RPC_TIMEOUT=1800000

export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export VLLM_USE_PD_SPLIT=1
export VLLM_USE_MARLIN_W16A16_MOE=1
export MOE_NN=0
export VLLM_USE_FUSE_SILU_AND_MUL=1
export VLLM_USE_FUSED_RMS_ROPE=1
export VLLM_USE_LIGHTOP_FILL_MOE_ALIGN=1
export VLLM_USE_LIGHTOP_MOE_ALIGN=1
export VLLM_USE_OPT_RESHAPE_AND_CACHE=1


model_path=/public/opendas/DL_DATA/llm-models/qwen3/Qwen3-Coder-480B-A35B-Instruct
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
logpath="./server/Qwen3-Coder-480B-A35B-Instruct/"
port=8000
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi


vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 -tp 16 \
 --host 0.0.0.0 \
 --port $port \
 --max-model-len 16384 \
 --block-size 64 \
 --disable-log-requests \
 --enable-chunked-prefill \
 --max-num-batched-tokens 10240 \
 --disable-custom-all-reduce \
 --no-enable-prefix-caching   2>&1 | tee  ${logpath}.log

```

#### BW1000
```plaintext {wrap}
--kv-cache-dtype fp8_e5m2
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 
-tp 8
```

#### K100_AI
```plaintext {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1 #K100AI
```

## DeepSeek-V3.2-Channel-INT8【朱明翠】

#### 测试模式
```bash {wrap}
rm -rf ~/.triton
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=1
export VLLM_REJECT_SAMPLE_OPT=1
export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1
# 可选项
export VLLM_ZERO_OVERHEAD=1 # 零消耗
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=1
export VLLM_SPEC_DECODE_EAGER=1
export VLLM_USE_GLOBAL_CACHE13=1
export VLLM_FUSED_MOE_CHUNK_SIZE=16384
export VLLM_USE_FLASH_MLA_FP8=1
export VLLM_CUSTOM_CACHE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1
export VLLM_USE_FUSED_FILL_RMS_CAT=1
export VLLM_USE_FUSED_QA_KVA_GEMM=1
export VLLM_USE_CAT_MLA=1
time=$(date "+%m%d-%H%M")
vllm serve /public/opendas/DL_DATA/llm-models/vllm-w8a8-models/DeepSeek-V3.2-Channel-INT8/  \
 --trust-remote-code \
 -q slimquant_marlin \
 -tp 8  \
 --port 8000 \
 --dtype bfloat16 \
 --max-model-len 40960 \
 --max-num-seqs 256 \
 --max-num-batched-tokens 16384 \
 --max-seq-len-to-capture 16384 \
 --disable-log-requests \
 --enable-prefix-caching \
 --enable-chunked-prefill \
 --block-size 64 \
 --gpu-memory-utilization 0.90 \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 3}' \
 2>&1 | tee ./server-ifb/DeepSeek-R1-Channel-INT8-${time}.log


```

#### BW1000
```plaintext {wrap}
--kv-cache-dtype fp8_e5m2
-tp 16
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 
```

#### K100_AI
```plaintext {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1 #K100AI
-tp 16
```

## DeepSeek-R1-0528-W4A8-V2【朱明翠】

#### 测试模式
```bash {wrap}
rm -rf ~/.triton/

export ALLREDUCE_STREAM_WITH_COMPUTE=1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export HIP_VISIBLE_DEVICES=4,5,6,7
export VLLM_SPEC_DECODE_EAGER=1
export DEBUG_CLR_GRAPH_PACKET_CAPTURE=false
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
export VLLM_ENABLE_TBO=1
#export VLLM_ZERO_OVERHEAD=1 # zero

export VLLM_USE_LIGHTOP_MOE_SUM_MUL_ADD=1
export USE_FUSED_RMS_QUANT=1
export DEBUG_CLR_GRAPH_PACKET_CAPTURE=false
export VLLM_SPEC_DECODE_EAGER=1
#----pp并行开启如下变量----
export VLLM_SCHED_ENABLE_MINIMAL_INJECTION=1
#----如遇到运行过程中的OOM增加下面设置----
export VLLM_USE_GLOBAL_CACHE13=1
export VLLM_FUSED_MOE_CHUNK_SIZE=8192
export VLLM_USE_OPT_OP=1
export VLLM_ZERO_OVERHEAD=0
export VLLM_ENABLE_TBO=0
#export VLLM_PP_LAYER_PARTITION=33,28
#界林额外加的
#export VLLM_USE_OPT_ZEROS=1
#------------11.21更新-----------------
#export VLLM_USE_PD_SPLIT=1 这个可以注释
export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1

export VLLM_USE_CUDA_GRAPH_SIZES=1 #(默认)
export VLLM_USE_LIGHTOP_FILL_MOE_ALIGN=1
export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_PP_SYNC=1

export USE_FUSED_SILU_MUL_QUANT=1

export VLLM_REJECT_SAMPLE_OPT=0

export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=1

log_date=$(date "+%Y-%m-%d")
log_dir="./${log_date}"
if ! mkdir -p "${log_dir}"; then
    exit 1
fi
model_path=/public/opendas/DL_DATA/llm-models/vllm-w8a8-models/DeepSeek-R1-0528-W4A8-V2
model=${model_path##*/}
time=$(date "+%Y-%m-%d-%H-%M-%S")
data_type="bfloat16"
tp=4
port=8082
gpu_memory=0.90
vllm serve ${model_path} \
 --port ${port} \
 --trust-remote-code \
 --dtype ${data_type} \
 --max-model-len 40960 \
 -q slimquant_w4a8_marlin \
 --max-seq-len-to-capture 8192 \
 -tp ${tp} \
 --gpu-memory-utilization ${gpu_memory} \
 --max-num-seqs 256 \
 --max-num-batched-tokens 8192 \
 --block-size 64 \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 3}' \
 --enable-chunked-prefill \
 --enable-prefix-caching \
 --compilation-config '{"full_cuda_graph": true}'   \
 2>&1 | tee "${log_dir}/${model}_serve_${time}.log"


```

#### BW1000
```plaintext {wrap}
--kv-cache-dtype fp8_e5m2
-tp 8
```

#### NMZ
```bash {wrap}
--kv-cache-dtype fp8_e4m3 

```

#### K100_AI
```plaintext {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1 #K100AI
-tp 8
```

## Qwen3-4B-Thinking-2507【胡雅晴】

#### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=6
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=1
export VLLM_USE_OPT_ZEROS=1
export VLLM_ZERO_OVERHEAD=1 # zero

model_path=/mnt/model/qwen3/Qwen3-4B-Thinking-2507/

#model_path=/mnt/model/Qwen3-8B/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=1
logpath="./server/0109-k100ai/Qwen3-4B-Thinking-2507-tx-vllm011-0109"
port=8121
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 VLLM_ZERO_OVERHEAD=1 vllm serve ${model_path} \
        -tp 1 --trust-remote-code --port $port \
        --no-enable-prefix-caching --enable-chunked-prefill \
        --max-num-seqs 512  --disable-cascade-attn >&1 | tee  ${logpath}.log

```

### bw1000

添加 --kv-cache-dtype fp8_e5m2

### NMZ

添加--kv-cache-dtype fp8_e4m3

### k100ai
```plaintext {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1
```

## Qwen3-VL-4B-Thinking【胡雅晴】

### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=5
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export VLLM_RPC_TIMEOUT=1800000
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=1

export VLLM_ZERO_OVERHEAD=1 # zero
export VLLM_USE_OPT_ZEROS=1

model_path=/mnt/model/qwen3/Qwen3-VL-4B-Thinking/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=1
logpath="./server/0109-k100ai/Qwen3-VL-4B-Thinking-0109/"
port=8675
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --limit-mm-per-prompt '{"image": 4}'  \
    --no-enable-prefix-caching --enable-chunked-prefill \
    --disable-cascade-attn   >&1 | tee  ${logpath}.log
```

### bw1000

添加--kv-cache-dtype fp8_e5m2

### NMZ

添加--kv-cache-dtype fp8_e4m3

### k100ai
```plaintext {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
```


## QwQ-32B【赵自广】

### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3
model_path=/model/QwQ-32B/

model=${model_path##*/}
time=$(date "+%m%d-%H%M")
data_type="float16"
tp=4
logpath="/mnt/tfcc_new/vllm/benchmarks/tc_opt/logs/QwQ-32B/${time}"
port=8000
if [ ! -d ${logpath} ]; then
    mkdir -p ${logpath}
fi

# 在后台启动 vllm serve
vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --gpu-memory-utilization 0.9 \
    --trust-remote-code \
    --max-num-seqs 1024 --distributed-executor-backend=mp --no-enable-prefix-caching  --disable-cascade-attn  2>&1 | tee  ${logpath}/serve-kaixiang-0112-jingdu.log &

```

### bw1000
```python {wrap}
--kv-cache-dtype fp8_e5m2
```

### NMZ
```python {wrap}
--kv-cache-dtype fp8_e4m3
```

### k100ai
```bash {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1
```

## DeepSeek-R1-Distill-Llama-70B【赵自广】
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3

model_path=/models/deepseek-r1/DeepSeek-R1-Distill-Llama-70B-main/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=4
logpath="./server/nmz/DeepSeek-R1-Distill-Llama-70B-main"
port=8141
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi


vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --gpu-memory-utilization 0.9 \
    --trust-remote-code \
    --max-num-seqs 1024 --distributed-executor-backend=mp --no-enable-prefix-caching  --disable-cascade-attn  2>&1 | tee  ${logpath}/serve-kaixiang-0105-jingdu.log &

```

### bw1000
```python {wrap}
--kv-cache-dtype fp8_e5m2
```

### NMZ
```python {wrap}
--kv-cache-dtype fp8_e4m3
```

### k100ai
```bash {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1
```

## DeepSeek-R1-Distill-Llama-70B-quantized.w8a8【赵自广】
```bash {wrap}
export HIP_VISIBLE_DEVICES=2,3,4,5

model_path=/models/vllm-w8a8-models/DeepSeek-R1-Distill-Llama-70B-quantized.w8a8/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=4
logpath="./server/nmz/DeepSeek-R1-Distill-Llama-70B-quantized.w8a8"
port=8142
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi



vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --gpu-memory-utilization 0.9 \
    --trust-remote-code \
    --max-num-seqs 1024 --distributed-executor-backend=mp --no-enable-prefix-caching  --disable-cascade-attn  2>&1 | tee  ${logpath}/serve-kaixiang-0105-jingdu.log &

```

### bw1000
```python {wrap}
--kv-cache-dtype fp8_e5m2
```

### NMZ
```python {wrap}
--kv-cache-dtype fp8_e4m3
```

### k100ai
```bash {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1
```


## DeepSeek-R1-bf16【赵自广】
```bash {wrap}
export VLLM_USE_OPT_ZEROS=1 
export VLLM_USE_V1=0 
export ALLREDUCE_STREAM_WITH_COMPUTE=1 
export TORCH_BLAS_PREFER_HIPBLASLT=0 
export HIP_USE_GRAPH_QUEUE_POOL=1 
export USE_FUSED_RMS_QUANT=0 
export USE_FUSED_SILU_MUL_QUANT=0 
export VLLM_USE_LIGHTOP_FILL_MOE_ALIGN=0 
export AllTOAll_STREAM_WITH_COMPUTE=1 
export GATHER_STREAM_WITH_COMPUTE=1 
export Allgather_Base_STREAM_WITH_COMPUTE=1 
export NCCL_SIMPLE_CHANNELS=32 
export LMSLIM_USE_LIGHTOP=1 
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=0 
export export VLLM_USE_GLOBAL_CACHE13=1  
export VLLM_USE_FUSED_QA_KVA_GEMM=0 
export VLLM_USE_FUSED_DTBMM=0
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
model_path=/models/deepseek-r1/DeepSeek-R1-bf16/
# model_path=/model/DeepSeek-R1-0528-bf16
model=${model_path##*/}
data_type="bfloat16"
tp=16
port=8828
gpu_memory=0.9

log_date=$(date "+%Y-%m-%d")
time=$(date "+%Y-%m-%d-%H-%M-%S")
log_dir="nmz_${model}/${log_date}"
mkdir -p "${log_dir}"

vllm serve ${model_path} \
 --trust-remote-code \
 --distributed-executor-backend ray \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --gpu-memory-utilization $gpu_memory \
 --host 0.0.0.0 \
 --port $port \
 --max-model-len 40960 \
 --max-seq-len-to-capture 40960 \
 --max-num-batched-tokens 40960 \
 --disable-log-requests \
 --block-size 64 \
 --kv-cache-dtype fp8_e4m3 \
 --speculative_config '{"num_speculative_tokens": 3}'  \
 --enable-chunked-prefill \
 --no-enable-prefix-caching \
 --disable-cascade-attn \
 2>&1 | tee "${log_dir}/serve_${time}.log"

```

### bw1000
```python {wrap}
--kv-cache-dtype fp8_e5m2
```

### NMZ
```python {wrap}
--kv-cache-dtype fp8_e4m3
```

### k100ai
```bash {wrap}
待测试
```

## GLM-4-32B-0414【胡雅晴】

### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=4,5
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=1
export VLLM_RANK1_NUMA=1

export VLLM_USE_OPT_ZEROS=1

export VLLM_ZERO_OVERHEAD=1 # zero

export VLLM_USE_PD_SPLIT=1
export VLLM_USE_V1=1


model_path=/mnt/model/GLM-4/GLM-4-32B-0414/


#export LD_LIBRARY_PATH=/hyq/tfcc/rocblas-install-1127-bug111580/rocblas-install/lib/:$LD_LIBRARY_PATH

model=${model_path##*/}
time=$(date "+%m%d-%H%M")
data_type="bfloat16"
tp=2
logpath="./server/0109-k100ai/GLM-4-32B-0414-vllm011/${time}"
port=8133
if [ ! -d ${logpath} ]; then
    mkdir -p ${logpath}
fi

vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype bfloat16 --tensor-parallel-size $tp \
    --gpu-memory-utilization 0.9 \
    --trust-remote-code \
    --max-num-seqs 1024 \
    --distributed-executor-backend=mp \
    --no-enable-prefix-caching  \
    --enable-chunked-prefill --disable-cascade-attn  2>&1 | tee  ${logpath}/serve.log
```

### bw1000

添加--kv-cache-dtype fp8_e5m2

### NMZ

添加--kv-cache-dtype fp8_e4m3

### k100ai
```bash {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1
```

## Qwen2.5-VL-72B-Instruct-quantized.w8a8【胡雅晴】

### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1


export VLLM_ZERO_OVERHEAD=1 # zero
export VLLM_USE_OPT_ZEROS=1

model_path=/mnt/model/qwen2.5/Qwen2.5-VL-72B-Instruct-quantized.w8a8/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=8
logpath="./server/k100ai/Qwen2.5-VL-72B-Instruct-int8-vllm011"
port=8140
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

VLLM_USE_PD_SPLIT=1  VLLM_USE_V1=1 vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype bfloat16 --tensor-parallel-size $tp \
    --trust-remote-code \
    --no-enable-prefix-caching \
    --enable-chunked-prefill \
    --disable-cascade-attn >&1 | tee  ${logpath}/server.log

```

### bw1000

export TORCHDYNAMO_DISABLE=1

添加--kv-cache-dtype fp8_e5m2

### NMZ

export TORCHDYNAMO_DISABLE=1

添加--kv-cache-dtype fp8_e4m3

### k100ai
```bash {wrap}
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1
```

## Qwen3-235B-A22B-Thinking-2507【胡雅晴】

### 测试模式
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
#export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK1_NUMA=1
export VLLM_RANK1_NUMA=1
export VLLM_RANK1_NUMA=1
export VLLM_RANK1_NUMA=1

export VLLM_USE_OPT_ZEROS=1

export VLLM_ZERO_OVERHEAD=1 # zero

export VLLM_USE_PD_SPLIT=1
export VLLM_USE_V1=1


model_path=/mnt/model/qwen3/Qwen3-235B-A22B-Thinking-2507/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="float16"
tp=8
logpath="./server/Qwen3-235B-A22B-Thinking-2507-vllm011-660-1"
port=8144
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

vllm serve ${model_path} \
    --host 0.0.0.0 --port $port \
    --dtype float16 --tensor-parallel-size $tp \
    --trust-remote-code  --no-enable-prefix-caching  \
    --enable-chunked-prefill  --disable-cascade-attn   >&1 | tee  ${logpath}.log


```

### bw1000

添加--kv-cache-dtype fp8_e5m2

### NMZ

添加--kv-cache-dtype fp8_e4m3

### k100ai

待测试

## DeepSeek-V3.1-Terminus-fp8【胡雅晴】
```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=1
export VLLM_RANK4_NUMA=2
export VLLM_RANK5_NUMA=2
export VLLM_RANK6_NUMA=3
export VLLM_RANK7_NUMA=3
export VLLM_ZERO_OVERHEAD=1 # zero
export VLLM_USE_OPT_ZEROS=1
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=0
export VLLM_USE_LIGHTOP_MOE_SUM_MUL_ADD=0

export VLLM_USE_FUSED_QA_KVA_GEMM=0
export VLLM_USE_FUSED_DTBMM=0


model_path=/mnt/model/deepseek-v3.1/DeepSeek-V3.1-Terminus/
model=${model_path##*/}
time=$(date "+%m%d-%H:%M")
data_type="bfloat16"
tp=8
logpath="./server/0226/nmz/DeepSeek-V3.1-Terminus-fp8-tp8-vllm/"
port=9348
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi


vllm serve ${model_path} \
 --trust-remote-code \
 --dtype $data_type \
 --tensor-parallel-size $tp \
 --gpu-memory-utilization 0.9 \
 --disable-cascade-attn \
 --host 0.0.0.0 \
 --port $port \
 --block-size 64 \
 --max-model-len 40960 \
 --max-num-batched-tokens 204800 \
 --no-enable-prefix-caching  \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 1}'  2>&1 | tee "${logpath}/serve_${time}.log"

```

### bw1000

待测试

### NMZ

添加--kv-cache-dtype fp8_e4m3

### k100ai

待测试

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/deepseek-r1|deepseek-r1]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1|DeepSeek-R1]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-0528-bf16|DeepSeek-R1-0528-bf16]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-0528-W4A8-V2|DeepSeek-R1-0528-W4A8-V2]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-bf16|DeepSeek-R1-bf16]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-INT8|DeepSeek-R1-Channel-INT8]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B|DeepSeek-R1-Distill-Llama-70B]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B-main|DeepSeek-R1-Distill-Llama-70B-main]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B-quantized.w8a8|DeepSeek-R1-Distill-Llama-70B-quantized.w8a8]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Qwen-32B|DeepSeek-R1-Distill-Qwen-32B]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3-W4A8-V2-tx|DeepSeek-V3-W4A8-V2-tx]]（model）
- [[飞书知识图谱/实体/deepseek-v3.1|deepseek-v3.1]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3.1-Terminus|DeepSeek-V3.1-Terminus]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3.1-Terminus-fp8|DeepSeek-V3.1-Terminus-fp8]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3.1-Terminus-fp8-tp8-vllm|DeepSeek-V3.1-Terminus-fp8-tp8-vllm]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3.2-Channel-INT8|DeepSeek-V3.2-Channel-INT8]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-4|GLM-4]]（model）
- [[飞书知识图谱/实体/GLM-4-32B-0414|GLM-4-32B-0414]]（model）
- [[飞书知识图谱/实体/GLM-4-32B-0414-vllm011|GLM-4-32B-0414-vllm011]]（model）
- [[飞书知识图谱/实体/GLM-4.1V-9B-Thinking|GLM-4.1V-9B-Thinking]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OOM|OOM]]（issue）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）
- [[飞书知识图谱/实体/Qwen2.5-VL-32B-Instruct|Qwen2.5-VL-32B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen2.5-VL-72B-Instruct|Qwen2.5-VL-72B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen2.5-VL-72B-Instruct-int8-vllm011|Qwen2.5-VL-72B-Instruct-int8-vllm011]]（model）
- [[飞书知识图谱/实体/Qwen2.5-VL-72B-Instruct-quantized.w8a8|Qwen2.5-VL-72B-Instruct-quantized.w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-235B-A22B|Qwen3-235B-A22B]]（model）
- [[飞书知识图谱/实体/Qwen3-235B-A22B-Instruct-2507|Qwen3-235B-A22B-Instruct-2507]]（model）
- [[飞书知识图谱/实体/Qwen3-235B-A22B-Thinking-2507|Qwen3-235B-A22B-Thinking-2507]]（model）
- [[飞书知识图谱/实体/Qwen3-235B-A22B-Thinking-2507-vllm011-660-1|Qwen3-235B-A22B-Thinking-2507-vllm011-660-1]]（model）
- [[飞书知识图谱/实体/Qwen3-235B-A22B-W8A8|Qwen3-235B-A22B-W8A8]]（model）
- [[飞书知识图谱/实体/Qwen3-30B-A3B|Qwen3-30B-A3B]]（model）
- [[飞书知识图谱/实体/Qwen3-30B-A3B-Thinking-2507|Qwen3-30B-A3B-Thinking-2507]]（model）
- [[飞书知识图谱/实体/Qwen3-30B-A3B-thinking-2507-W8A8|Qwen3-30B-A3B-thinking-2507-W8A8]]（model）
- [[飞书知识图谱/实体/Qwen3-30B-A3B-Thinking-2507-w8a8|Qwen3-30B-A3B-Thinking-2507-w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-30B-A3B.w8a8|Qwen3-30B-A3B.w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-32B|Qwen3-32B]]（model）
- [[飞书知识图谱/实体/Qwen3-32B-W8A8|Qwen3-32B-W8A8]]（model）
- [[飞书知识图谱/实体/Qwen3-32B.w8a8|Qwen3-32B.w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-4B-Thinking-2507|Qwen3-4B-Thinking-2507]]（model）
- [[飞书知识图谱/实体/Qwen3-4B-Thinking-2507-tx-vllm011-0109|Qwen3-4B-Thinking-2507-tx-vllm011-0109]]（model）
- [[飞书知识图谱/实体/Qwen3-8B|Qwen3-8B]]（model）
- [[飞书知识图谱/实体/Qwen3-8B.w8a8|Qwen3-8B.w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-Coder-480B-A35B-Instruct|Qwen3-Coder-480B-A35B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-235B-A22B-Instruct|Qwen3-VL-235B-A22B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-235B-A22B-Thinking|Qwen3-VL-235B-A22B-Thinking]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-235BB-A22B-Thinking|Qwen3-VL-235BB-A22B-Thinking]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-30B-A3B-Thinking|Qwen3-VL-30B-A3B-Thinking]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-30B-Thinking|Qwen3-VL-30B-Thinking]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-4B-Instruct|Qwen3-VL-4B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-4B-Thinking|Qwen3-VL-4B-Thinking]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-4B-Thinking-0109|Qwen3-VL-4B-Thinking-0109]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-8B-thinking|Qwen3-VL-8B-thinking]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-8B-Thinking|Qwen3-VL-8B-Thinking]]（model）
- [[飞书知识图谱/实体/TBO|TBO]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

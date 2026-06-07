---
title: PD分离
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Distill-Llama-70B
  - Qwen3-8B
framework:
  - vLLM
issue_type: []
source: feishu
feishu_token: LkDhd7wbRo8ZMrxEMGAcQY0Dnp3
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/LkDhd7wbRo8ZMrxEMGAcQY0Dnp3
last_synced_at: 2026-06-07T17:07:53.393Z
---

# PD分离

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/LkDhd7wbRo8ZMrxEMGAcQY0Dnp3)

## PD分离（减少首字耗时，增加吞吐量）

 为什么要进行 PD 分离？(动机)

传统的 LLM 服务（如 HuggingFace Transformers 的默认实现）通常将一个请求的 Prefill 和 Decode 阶段视为一个不可分割的整体。这种做法会导致严重的性能瓶颈，原因在于 P 阶段和 D 阶段的计算特性截然不同。

我们可以通过一个表格清晰地对比二者的差异：

<lark-table rows="6" cols="3" column-widths="100,336,370">

  <lark-tr>
    <lark-td>
      特性
    </lark-td>
    <lark-td>
      **P 阶段 (Prefill)**
    </lark-td>
    <lark-td>
      **D 阶段 (Decode)**
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **计算模式**
    </lark-td>
    <lark-td>
      大规模并行计算 (处理整个长序列)
    </lark-td>
    <lark-td>
      增量、串行计算 (一次处理一个 token)
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **性能瓶颈**
    </lark-td>
    <lark-td>
      **计算密集型 (Compute-Bound)**
    </lark-td>
    <lark-td>
      **内存带宽密集型 (Memory-Bandwidth-Bound)**
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **主要操作**
    </lark-td>
    <lark-td>
      大型矩阵乘法 (`GEMM`)
    </lark-td>
    <lark-td>
      向量-矩阵乘法 (`GEMV`)，主要时间花在读KV缓存
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **KV缓存操作**
    </lark-td>
    <lark-td>
      **写入 (Write)**：为整个 prompt 生成并写入 KV 缓存
    </lark-td>
    <lark-td>
      **读取 (Read) + 追加 (Append)**：读取全部历史KV缓存，并为新生成的 token 追加KV
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **并行度**
    </lark-td>
    <lark-td>
      非常高，可以有效利用 GPU 的所有计算单元
    </lark-td>
    <lark-td>
      较低，受限于一次生成一个 token 的自回归特性
    </lark-td>
  </lark-tr>
</lark-table>

vLLM中的PD分离架构

vLLM的PD分离架构主要由三个组件构成：<text bgcolor="light-yellow">Proxy API server、vLLM prefill和vLLM decode</text>。以下是它们的工作原理：

1️⃣ 当请求到来时，首先会进入Proxy API server。Proxy API server会将请求转发给vLLM prefill。

2️⃣ vLLM prefill负责进行prefill操作，生成kv cache，并将这个缓存转发给vLLM decode。

3️⃣ Proxy API server继续将请求发送给vLLM decode。vLLM decode会执行drop_select操作，这个操作主要是获取kv和跳过prefill。完成drop_select后，vLLM decode会生成First token。

4️⃣ vLLM decode阶段也会生成First token。Proxy API server最终获取的所有token都是从vLLM decode获取的。vLLM prefill只负责生成kv cache。

通过这种方式，vLLM的PD分离架构能够实现高效的请求处理和缓存管理。

原文链接：https://blog.csdn.net/m_aigc2022/article/details/148722354

nv环境跑pd分离
```bash {wrap}
#!/bin/bash
# This file demonstrates the example usage of disaggregated prefilling
# We will launch 2 vllm instances (1 for prefill and 1 for decode),
# and then transfer the KV cache between them.

set -xe
export VLLM_USE_V1=0
MODEL_NAME="/model/local_model/DeepSeek-R1-Distill-Llama-70B"
MAX_MODEL_LEN=24720
GPU_MEMORY_UTILIZATION=0.95
gpu_id_prefill="0,1"
gpu_id_decoder="2,3"
kv_cache_type="SharedStorageConnector"  # SharedStorageConnector/ LMCacheConnectorV1 / NixlConnector

# Trap the SIGINT signal (triggered by Ctrl+C)
trap 'cleanup' INT

export VLLM_HOST_IP=$(hostname -I | awk '{print $1}')

# a function that waits vLLM server to start
wait_for_server() {
  local port=$1
  timeout 1200 bash -c "
    until curl -s localhost:${port}/v1/completions > /dev/null; do
      sleep 1
    done" && return 0 || return 1
}


# You can also adjust --kv-ip and --kv-port for distributed inference.
gpu_count_prefill=$(echo "$gpu_id_prefill" | tr ',' '\n' | wc -l)
gpu_count_decode=$(echo "$gpu_id_decoder" | tr ',' '\n' | wc -l)

    echo "Using LMCacheConnectorV1 for kv cache"
        # prefilling instance, which is the KV producer
    CUDA_VISIBLE_DEVICES=$gpu_id_prefill python3 -m vllm.entrypoints.openai.api_server  \
                    --model $MODEL_NAME \
                    --disable-log-requests \
                    --port 8100 \
                    --max-model-len $MAX_MODEL_LEN \
                    --gpu-memory-utilization $GPU_MEMORY_UTILIZATION \
                    --trust-remote-code \
                    --tensor-parallel-size $gpu_count_prefill \
                    --kv-transfer-config \
                    '{"kv_connector":"LMCacheConnector","kv_role":"kv_producer"}' 2>&1 | tee producer-v0.log &
#                   '{"kv_connector":"LMCacheConnector","kv_role":"kv_producer","kv_connector_extra_config": {"discard_partial_chunks": false, "lmcache_rpc_port": "producer1"}}' 2>&1 | tee producer-v0.log


#                    '{"kv_connector":"LMCacheConnectorV1","kv_role":"kv_producer","kv_connector_extra_config": {"discard_partial_chunks": false, "lmcache_rpc_port": "producer1"}}' 2>&1 | tee producer.log &

    CUDA_VISIBLE_DEVICES=$gpu_id_decoder python3 -m vllm.entrypoints.openai.api_server \
                    --model $MODEL_NAME  \
                    --disable-log-requests \
                    --port 8200 \
                    --max-model-len $MAX_MODEL_LEN \
                    --gpu-memory-utilization $GPU_MEMORY_UTILIZATION \
                    --trust-remote-code \
                    --tensor-parallel-size $gpu_count_decode \
                    --kv-transfer-config \
                    '{"kv_connector":"LMCacheConnector","kv_role":"kv_consumer"}' 2>&1 | tee consumer-v0.log &
#                   '{"kv_connector":"LMCacheConnector","kv_role":"kv_consumer","kv_connector_extra_config": {"discard_partial_chunks": false, "lmcache_rpc_port": "consumer1"}}' 2>&1 | tee consumer-v0.log &
#                    '{"kv_connector":"LMCacheConnectorV1","kv_role":"kv_consumer","kv_connector_extra_config": {"discard_partial_chunks": false, "lmcache_rpc_port": "consumer1"}}' 2>&1 | tee consumer.log &


# wait until prefill and decode instances are ready
wait_for_server 8100
wait_for_server 8200

python3  disagg_proxy_demo.py 2>&1 | tee connect.log &
```

```plaintext {wrap}
python disagg_proxy_demo.py --model /model/local_model/DeepSeek-R1-Distill-Llama-70B --prefill 0.0.0.0:8100 --decode 0.0.0.0:8200 
```

客户端：
```bash {wrap}
#model_path=/model/Qwen3-8B
model_path=/model/local_model/DeepSeek-R1-Distill-Llama-70B
#export VLLM_NUMA_BIND=1


model=${model_path##*/}
tp=4
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="./pdlog/${model}-${mode}-$time-len27k-pd"
#logdir="${logpath}/${model}-tp${tp}-${time}-${num_prompts}-in${input}-out${output}.log"
#logdir=${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log
data_type="bf16"
csv_name="${model}-tp${tp}-$time"
port=8000

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"
#pairs=("1000 1000" "1000 1000" "1000 1000" )
#pairs=("1000 1000" "2000 1" "2000 2000" "4096 1" "512 1024" )
pairs=("16384 1024")
#pairs=("4096 1024")
#pairs=("512 512")
#pairs=("2048 2048")
#pairs=("2 1000" "2 2000")
#pairs=("2 128" "2 1000" "2 2000" "512 1" "512 512" "512 1024" "1000 1" "2000 1" "4096 1" "1000 1000" "2000 2000")
#pairs=("2 128" "2 1000" "2 2000" "512 1" "512 512" "1000 1" "1000 1000" "2000 1" "2000 2000" "4096 1" "512 1024" )
#pairs=("2 128" "2 1000" "2 2000" "512 1" "512 512" "1000 1")
echo "data_type,batch,prompt_tokens,completion_tokens,Output_token_throughput(tok/s),Total_Token_throughput(tok/s),Mean_TTFT(ms),Mean_TPOT(ms),Mean_throughput,duration" > $all_log


#for batch in $(seq 20 1 25) ; do
for batch in 15 16 17 ; do
    for pair in "${pairs[@]}"; do
#       echo "data_type,batch,prompt_tokens,completion_tokens,tp,Benchmark_duration,Total_input_tokens,Total_generated_tokens,Request_throughput,Output_token_throughput,Total_Token_throughput,Mean_TTFT,Mean_TPOT,Mean_ITL" >> $all_log
        prompt_tokens=${pair%% *}
        completion_tokens=${pair#* }
        vllm bench serve \
        --model ${model_path} \
        --random-input-len $prompt_tokens \
        --random-output-len $completion_tokens \
        --port $port \
        --dataset-name random \
        --num-prompts $batch --max-concurrency $batch --ignore-eos --seed 42 2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log

        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total Token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`

        Mean_TTFT=`grep -a "^Mean TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`

        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`

        Mean_throughput=$(echo "scale=2; 1000/$Mean_TPOT" | bc -l)

        echo "$data_type,$batch,$prompt_tokens,$completion_tokens,$Output_token_throughput,$Total_Token_throughput,$Mean_TTFT,$Mean_TPOT,$Mean_throughput,$Benchmark_duration">> $all_log

     done
done


cat $all_log

```


https://github.com/LMCache/LMCache

[[飞书知识图谱/资源/PD分离数据--cEImpnne|PD分离数据]]

## 关联资源

- [[飞书知识图谱/资源/PD分离数据--cEImpnne|PD分离数据]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/吞吐量|吞吐量]]（metric）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B|DeepSeek-R1-Distill-Llama-70B]]（model）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/PD 分离|PD 分离]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/Qwen3-8B|Qwen3-8B]]（model）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

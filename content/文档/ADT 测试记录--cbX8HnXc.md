---
title: ADT 测试记录
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-0528-Channel-INT8
  - DeepSeek-R1-Channel-FP8
framework:
  - SGLang
  - vLLM
issue_type: []
source: feishu
feishu_token: SqyWdDzZ2ovQXcxquPbcbX8HnXc
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/SqyWdDzZ2ovQXcxquPbcbX8HnXc
last_synced_at: 2026-06-07T17:07:53.393Z
---

# ADT 测试记录

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/SqyWdDzZ2ovQXcxquPbcbX8HnXc)

# dpsk-r1 fp8 sglang

p：tp4 pp2

d: tp8 *3

镜像tag： daily-20260428-2004

## prefill
```bash
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
#export ROCSHMEM_TOPO_FILE_FORCE="/mnt/gbj/topo_400g.config"

export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_bond_0,mlx5_bond_1,mlx5_bond_2,mlx5_bond_3
export ROCSHMEM_HEAP_SIZE=3737418240
export ROCSHMEM_MAX_NUM_CONTEXTS=48
export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128

#export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100

export NCCL_SOCKET_IFNAME=eth1
export GLOO_SOCKET_IFNAME=eth1

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要

#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16

export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # disable in w8a8
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_MLA_CAT=0  # set to 0 if nmz
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_USE_FUSED_RMS_QUANT=0  # disable in w8a8
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # disable in w8a8
export SGLANG_USE_RMS_QUANT_PATH=0
export USE_FUSED_RMS_QUANT_PATH=1

export MC_ALLOWED_IBV_DEVICES=mlx5_bond_0

export DEEPEP_ENABLE_LL_DISPATCH_OPT=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
#export MC_TOPO_FILE_FORCE=/data0/dpsk-r1/sglang-v0.5.10rc0_dev/python/mc_topo.config

export SGLANG_HOST_IP=28.81.64.28
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
export MC_LOG_LEVE=INFO

export MC_IB_GID_INDEX=3

model_path=/data0/models/DeepSeek-R1-Channel-FP8/
model=${model_path##*/}
tp=2
pp=4
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=140000
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="pserver/$model-tp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-cuda-graph "
option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="16,15,15,15"
option+=" --pp-max-micro-batch-size 1"
option+=" --disaggregation-ib-device mlx5_bond_0"
option+=" --disaggregation-mode prefill --load-balance-method round_robin "

#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8  --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30003 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${logpath}-$time.log

```


## decode
```bash
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1

export NCCL_SOCKET_IFNAME=eth1
export GLOO_SOCKET_IFNAME=eth1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）

#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
host_ip=$(hostname -I | awk '{print $1}')
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
#export W8A8_SUPPORT_METHODS=1
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_bond_0,mlx5_bond_1,mlx5_bond_2,mlx5_bond_3
#export MC_TOPO_FILE_FORCE=/mnt/fasttrans/mc_topo.config
export MC_IB_GID_INDEX=3
export SGLANG_HOST_IP=$host_ip


model_path=/data0/models/DeepSeek-R1-Channel-FP8/
model=${model_path##*/}
tp=8 #K100AI为8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=40000
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="pserver/$model-tp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
option+=" --disaggregation-mode decode --prefill-round-robin-balance "
option+=" --disaggregation-ib-device mlx5_bond_0,mlx5_bond_1,mlx5_bond_2,mlx5_bond_3"
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
# export SGLANG_PP_LAYER_PARTITION="16,15,15,15"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
#option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 1"
# option+=" --pp-async-batch-depth 1"
#option+=" --schedule-conservativeness 0.3"
#option+=" --disaggregation-decode-dp 8 --disaggregation-decode-tp 8"
# option+=" --disaggregation-ib-device mlx5_6,mlx5_7,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
# option+=" --disaggregation-mode prefill --load-balance-method round_robin "
option+=" --max-running-requests 1024 "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 64 "

#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30004 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${logpath}-$time.log
```


代理
```plaintext
python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://28.81.64.28:30003 --decode http://28.81.64.168:30004 --decode http://28.81.64.206:30004 --decode http://28.81.64.207:30004 --policy round_robin --port 3000
```



```plaintext
curl -X POST http://localhost:3000/v1/chat/completions \ 
 -H "Content-Type: application/json" \  
 -d '{    "model": "/data0/models/DeepSeek-R1-Channel-FP8/",    "messages": [{"role": "user", "content": "请用一句话介绍人工智能。"}],    "max_tokens": 50,    "temperature": 0.7,    "stream": false  }'
```


```bash {wrap}
model_path=/data0/models/DeepSeek-R1-Channel-FP8/
model=${model_path##*/}
tp=8
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="./client/${model}-${mode}-$time-test"
#logdir="${logpath}/${model}-tp${tp}-${time}-${num_prompts}-in${input}-out${output}.log"
#logdir=${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log
data_type="fp16"
port=3000
csv_name="${model}-tp${tp}-$time"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"


echo "input,output,request_rate,num_prompts,concurrency,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log

pairs=(
  "3366 1076"
)

#--profile
for pair in "${pairs[@]}"; do
  prompt_tokens=$(echo "$pair" | awk '{print $1}')
  completion_tokens=$(echo "$pair" | awk '{print $2}')
for batch in 10; do
#for batch in 256; do
        concurrency_multiplier=8
        if [ $batch -gt 128 ]; then
            concurrency_multiplier=4
        fi
        vllm bench serve \
        --model ${model_path} \
        --dataset-name random \
        --random-input-len $prompt_tokens \
        --random-output-len $completion_tokens \
        --num-prompts $((batch*concurrency_multiplier)) \
        --port $port \
        --metric-percentiles 95,99 \
        --max-concurrency $batch  \
        --ignore-eos \
        2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log

        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total Token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        request_rate=`grep -a "^Traffic request rate" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`


        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_TTFT=`grep -a "^Mean TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Median_TTFT=`grep -a "^Median TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Median_TPOT=`grep -a "^Median TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Median_ITL=`grep -a "^Median ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        qps=`grep -a "^Request throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P99_TTFT=`grep -a "^P99 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P99_TPOT=`grep -a "^P99 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P99_ITL=`grep -a "^P99 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_TTFT=`grep -a "^P95 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_TPOT=`grep -a "^P95 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_ITL=`grep -a "^P95 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_ITL=`grep -a "^Mean ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`

        echo "$prompt_tokens,$completion_tokens,$request_rate,$((batch*concurrency_multiplier)),$batch,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log
     done
done

```


# vllm

## 操作：
```bash
docker run -it \
--name shantf_vllm_adt \
--privileged \
--shm-size=500G \
--cap-add=SYS_PTRACE \
--security-opt seccomp=unconfined \
--ulimit memlock=-1:-1 \
--ipc=host \
--network host \
--device=/dev/kfd --device=/dev/dri/ \
--group-add video \
-v /opt/hyhal:/opt/hyhal:ro -v /data0/:/mnt -v /data1:/data1 \
harbor.sourcefind.cn:5443/dcu/admin/base/vllm:0.9.2-ubuntu22.04-dtk26.04-0130-py3.10-20260204 \
/bin/bash
```

p参数：

环境变量：
```bash {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=3

export VLLM_REJECT_SAMPLE_OPT=1
export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1

# 可选项
export VLLM_ZERO_OVERHEAD=0
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=1
export VLLM_SPEC_DECODE_EAGER=1
export VLLM_USE_GLOBAL_CACHE13=1
export VLLM_FUSED_MOE_CHUNK_SIZE=16384

export VLLM_USE_FLASH_MLA_FP8=1
export USE_FUSED_RMS_QUANT=1

export VLLM_CUSTOM_CACHE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1

export VLLM_USE_FUSED_FILL_RMS_CAT=1
export VLLM_USE_FLASH_ATTN_FP8=1

export VLLM_ZERO_OVERHEAD_ENHANCE=0
export VLLM_USE_FUSED_QA_KVA_GEMM=1

export VLLM_ENABLE_SHARED_EXPERTS_FUSION=1
export VLLM_USE_FUSED_DTBMM=1
export VLLM_USE_CAT_MLA=1

export VLLM_FUSE_CAT_AND_CAST_FP8=1
export VLLM_PP_LAYER_PARTITION=16,15,15,15

export VLLM_FUSED_GATHER_CACHE_CONVERT_FP8=1
export VLLM_FUSED_RN_ROPE_INT8_QUANT=1

export VLLM_HOST_IP=28.81.64.28
export NCCL_SOCKET_IFNAME=eth1
export GLOO_SOCKET_IFNAME=eth1
export NCCL_NET_GDR_READ=1
export NCCL_NET_GDR_LEVEL=7
export NCCL_IB_HCA=mlx5_bond_0:1,mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_NET_DEVICES=mlx5_bond_0:1,mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_IB_GID_INDEX=3
export NCCL_IB_GID_INDEX=3
export RCCL_MODEL_MATCHING_DISABLE=1
```

参数
```json
rm -rf ~/.cache
rm -rf ~/.triton
vllm serve /mnt/models/DeepSeek-R1-0528-Channel-INT8 \
 --trust-remote-code \
 -q slimquant_marlin \
 -tp 2 -pp 4 \
 --port 20012 \
 --dtype bfloat16 \
 --max-model-len 16384 \
 --max-num-seqs 256 \
 --max-num-batched-tokens 6144 \
 --max-seq-len-to-capture 6144 \
 --disable-log-requests \
 --no-enable-prefix-caching \
 --enable-chunked-prefill \
 --block-size 64 \
 --gpu-memory-utilization 0.90 \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 2}' \
 --kv-cache-dtype fp8_e4m3 \
 --enforce-eager \
 --kv-transfer-config '{"kv_connector":"P2pNcclConnector","kv_role":"kv_producer","kv_buffer_size":"1e4","kv_port":"21001","kv_connector_extra_config":{"enable_asymmetric_p2p":true,"remote_tp_size":8,"remote_pp_size":1,"proxy_ip":"28.81.64.28","proxy_port":"30006","http_port":"20012","send_type":"PUT_ASYNC"}}'
```

d参数：

环境变量：
```bash {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=3

export VLLM_REJECT_SAMPLE_OPT=1
export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1

# 可选项
export VLLM_ZERO_OVERHEAD=1
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=1
export VLLM_SPEC_DECODE_EAGER=1
export VLLM_USE_GLOBAL_CACHE13=1
export VLLM_FUSED_MOE_CHUNK_SIZE=16384

export VLLM_USE_FLASH_MLA_FP8=1
export USE_FUSED_RMS_QUANT=1

export VLLM_CUSTOM_CACHE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1

export VLLM_USE_FUSED_FILL_RMS_CAT=1
export VLLM_USE_FLASH_ATTN_FP8=1

export VLLM_ZERO_OVERHEAD_ENHANCE=1
export VLLM_USE_FUSED_QA_KVA_GEMM=1

export VLLM_ENABLE_SHARED_EXPERTS_FUSION=1
export VLLM_USE_FUSED_DTBMM=1
export VLLM_USE_CAT_MLA=1

export VLLM_HOST_IP=$(hostname -I | awk '{print $1}')
export NCCL_SOCKET_IFNAME=eth1
export GLOO_SOCKET_IFNAME=eth1
export NCCL_NET_GDR_READ=1
export NCCL_NET_GDR_LEVEL=7
export NCCL_IB_HCA=mlx5_bond_0:1,mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_NET_DEVICES=mlx5_bond_0:1,mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_IB_GID_INDEX=3
export NCCL_IB_GID_INDEX=3
export RCCL_MODEL_MATCHING_DISABLE=1
```

参数：
```json {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
vllm serve /mnt/models/DeepSeek-R1-0528-Channel-INT8 \
 --trust-remote-code \
 -q slimquant_marlin \
 -tp 8 \
 --port 20020 \
 --dtype bfloat16 \
 --max-model-len 40960 \
 --max-num-seqs 256 \
 --max-num-batched-tokens 6144 \
 --max-seq-len-to-capture 6144 \
 --disable-log-requests \
 --no-enable-prefix-caching \
 --enable-chunked-prefill \
 --block-size 64 \
 --gpu-memory-utilization 0.90 \
 --compilation-config '{"full_cuda_graph": true}' \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 2}' \
 --kv-cache-dtype fp8_e4m3 \
 --kv-transfer-config '{"kv_connector":"P2pNcclConnector","kv_role":"kv_consumer","kv_buffer_size":"1e9","kv_port":"22000","kv_connector_extra_config":{"proxy_ip":"28.81.64.28","proxy_port":"30006","http_port":"20020","send_type":"PUT_ASYNC"}}'
```

工具调用
```plaintext {wrap}
64cu刷72cu
/opt/hyhal/firmware/vbios
./hyflash_x86 --index a --update NMZ/DCU-3G-6.314.0034001.990881.gen5.bin --subsysId 0x6434 --xhcl
ipmitool power cycle

72cu刷回64cu
cd /opt/hyhal/firmware/vbios

./hyflash_x86 --index a --update ./NMZ/DCU-3G-6.314.0034001.990881.gen5.bin --subsysId 0x6433--xhcl
```

## 节点重启之后需要设置：
```plaintext {wrap}

mount /dev/nvme0n1p1 /data0
mount /dev/nvme1n1p1 /data1

systemctl restart docker.service

cd /data0
./setqos_hwj.sh 1 2 1
ip a | grep mtu

hy-smi --setsclk 9
hy-smi -c


```

0528模型性能，平均输出长度长，qps性能差。

<image token="MSH2bhhrJoz2PCx8MDhcyOTWn5g" width="2439" height="205" align="center"/>

换成0327模型，平均输出长度短1100左右，qps性能恢复

<image token="HLpkbMTH3o0lXaxsL9lcubfTnVd" width="2424" height="323" align="center"/>

## 报错：

RuntimeError: HIP error: out of memory 需要清理内存

<image token="BVGNbWBQ0oasD9xuM1HcqFaxnxd" width="2478" height="1502" align="center"/>

d卡住，尝试增加天河网卡的环境变量
```bash
export VLLM_HOST_IP=28.81.64.28
export NCCL_SOCKET_IFNAME=bond4
export GLOO_SOCKET_IFNAME=bond4
export NCCL_NET_GDR_READ=1
export NCCL_NET_GDR_LEVEL=7
export NCCL_IB_HCA=mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_NET_DEVICES=mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_IB_GID_INDEX=3
export NCCL_IB_GID_INDEX=3
export RCCL_MODEL_MATCHING_DISABLE=1
```

会出现nccl通信初始化报错，需要修改ib网卡

<image token="Vp2Bb0DA4oeRvWxf3tLcDTEHnlf" width="3020" height="1551" align="center"/>

```bash
export VLLM_HOST_IP=28.81.64.28
export NCCL_SOCKET_IFNAME=eth1
export GLOO_SOCKET_IFNAME=eth1
export NCCL_NET_GDR_READ=1
export NCCL_NET_GDR_LEVEL=7
export NCCL_IB_HCA=mlx5_bond_0:1,mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_NET_DEVICES=mlx5_bond_0:1,mlx5_bond_1:1,mlx5_bond_2:1,mlx5_bond_3:1
export UCX_IB_GID_INDEX=3
export NCCL_IB_GID_INDEX=3
export RCCL_MODEL_MATCHING_DISABLE=1
```

RuntimeError: NCCL error: remote process exited or there was a network error

p：03 d：01 02 04【01 02 64cu 03 04 72cu】

排查出来是node04机器mtu值对不上，不是4200

<image token="KbTVbZh0NoZMg7xWigEcz4kbn3E" width="3039" height="1627" align="center"/>

1p1d【03 04】

<image token="HI5cb3xhVomHACxrh6EcJ6pQndb" width="2984" height="1550" align="center"/>

1p1d【03 01】

Ok

<image token="M0O6bBl3yo4RGoxTo2IcrsXynif" width="2959" height="1539" align="center"/>

1p1d【03 02】

ok

<image token="Yj3pbGWFKoQ4KlxDiYrccMjynEh" width="2961" height="1528" align="center"/>

节点重启之后遇到显卡加载失败，需要重新安装驱动
```plaintext {wrap}
./rock-6.3.31-V1.5.0c.run
一路yes即可
```

<image token="WNiSb8u7joE6tPxdjnpcMPaJnha" width="2556" height="282" align="center"/>

数据见：[[飞书知识图谱/资源/性能数据--czfBXnpf|性能数据]]

## 关联资源

- [[飞书知识图谱/资源/性能数据--czfBXnpf|性能数据]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-0528-Channel-INT8|DeepSeek-R1-0528-Channel-INT8]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-FP8|DeepSeek-R1-Channel-FP8]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/RPS|RPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

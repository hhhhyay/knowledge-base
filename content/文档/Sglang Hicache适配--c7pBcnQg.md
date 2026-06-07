---
title: Sglang Hicache适配
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Distill-Qwen-7B
  - GLM-5-W8A8
  - GLM-w4a8-V2_6_test
  - Qwen3.5-397B-A17B
framework:
  - Mooncake
  - SGLang
issue_type: []
source: feishu
feishu_token: JBE6w2PBgigyPdkAvL2c7pBcnQg
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/JBE6w2PBgigyPdkAvL2c7pBcnQg
last_synced_at: 2026-06-07T17:07:53.393Z
---

# Sglang Hicache适配

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/JBE6w2PBgigyPdkAvL2c7pBcnQg)

# 统一设置

<callout emoji="exclamation" background-color="light-orange" border-color="light-orange">

MHA、GQA类注意力需要设置export SGLANG_KV_LAYOUT_DCU_FA=1、--hicache-mem-layout dcu_layout
</callout>

# IFB

### tp8
```bash
mkdir -p decode_logs
time=$(date "+%m%d-%H%M")
export NODE1_IP="xxx"   # Decode master IP
export MODEL_PATH="/localmodels/GLM-w4a8-V2_6_test"

# Decode 环境变量：按原始 W4A8 D 节点保留，不能用通用精简版替代
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_ENABLE_SPEC_V2=1
export SGLANG_TORCH_PROFILER_DIR=/shangxl/Test/GLM5/w4a8/prof/
export HSA_ENABLE_COREDUMP=1
export USE_DCU_CUSTOM_ALLREDUCE=1
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1
export SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export HIP_KERNEL_BATCH_CEILING=100
export GPU_FORCE_BLIT_COPY_SIZE=16
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export SGLANG_USE_LIGHTOP=1
export SGLANG_ROCM_USE_AITER_MOE=0
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=1
export HIP_GRAPH_USE_CMD_CACHE=0
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export NCCL_IB_HCA=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=ens14f0
export GLOO_SOCKET_IFNAME=ens14f0
export NCCL_IB_GID_INDEX=0
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
export MC_GID_INDEX=3
export SGLANG_HOST_IP=xxx

export MOONCAKE_TE_META_DATA_SERVER="http://xxx:8080/metadata"
export MOONCAKE_GLOBAL_SEGMENT_SIZE="64GB"
export MOONCAKE_PROTOCOL="tcp"
export MOONCAKE_DEVICE="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
export MOONCAKE_MASTER=xxx:50051

sysctl -w kernel.numa_balancing=0 || true

python3 -m sglang.launch_server \
  --model-path "${MODEL_PATH}" \
  --host "${NODE1_IP}" \
  --port 30000 \
  --nnodes 1 \
  --node-rank 0 \
  --dist-init-addr "${NODE1_IP}:5000" \
  --tp-size 8 \
  --pp-size 1 \
  --dp-size 1 \
  --ep-size 1 \
  --trust-remote-code \
  --dtype bfloat16 \
  --kv-cache-dtype bf16 \
  --mem-fraction-static 0.80 \
  --context-length 6000 \
  --page-size 64 \
  --disable-radix-cache \
  --chunked-prefill-size -1 \
  --nsa-prefill-backend flashmla_sparse  \
  --nsa-decode-backend flashmla_sparse \
  --quantization slimquant_w4a8_marlin \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --max-running-requests 512 \
  --enable-hierarchical-cache
  --hicache-ratio 1 \
  --hicache-mem-layout page_first \
  --hicache-io-backend kernel \
  --hicache-write-policy write_through \
  --hicache-storage-backend mooncake \
  --hicache-storage-prefetch-policy best_effort \
  2>&1 | tee tp_logs/tp8-$time.log
```

该节点需要另起一个终端启动moocake_master,--http_metadata_server_host 、MOONCAKE_TE_META_DATA_SERVER、MOONCAKE_MASTER设置为该节点ip
```plaintext
mooncake_master --enable_http_metadata_server=true --http_metadata_server_host=xxx --http_metadata_server_port=8080
```

### tp16

node1
```shell
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export ROCSHMEM_MAX_NUM_CONTEXTS=48
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_HEAP_SIZE=1900000000
export ROCSHMEM_TOPO_FILE_FORCE="/shangxl/Test/2p4d/config/topo.config"
export USE_SPE_MQP=1
export SGLANG_TORCH_PROFILER_DIR=/shangxl/Test/2p4d/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export MC_TOPO_FILE_FORCE=/shangxl/Test/2p4d/config/mc_topo.config

#kv layout
export SGLANG_KV_LAYOUT_DCU_FA=1

export MOONCAKE_TE_META_DATA_SERVER="http://xxx:8080/metadata"
export MOONCAKE_GLOBAL_SEGMENT_SIZE="128GB"
export MOONCAKE_PROTOCOL="tcp"
export DEVICE_LIST="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
export MOONCAKE_DEVICE="$DEVICE_LIST"
export MOONCAKE_MASTER=xxx:50051

sysctl -w kernel.numa_balancing=0

model_path=/localmodels/Qwen3.5-397B-A17B
#model_path=/model/DeepSeek-R1-Distill-Qwen-7B
model=${model_path##*/}
tp=16 #K100AI为8
pp=1
dp=1
ep=1
nodes=2
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
max_model_len=6000
gpu_mem=0.80
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4"
option+=" --mamba-scheduler-strategy extra_buffer"
option+=" --chunked-prefill-size 16384"
option+=" --enable-hierarchical-cache --hicache-ratio 2 --hicache-mem-layout dcu_layout --hicache-io-backend kernel --hicache-storage-backend mooncake --hicache-write-policy write_through --hicache-storage-prefetch-policy timeout"


#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --host $host_ip  --port 30000 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64 2>&1 | tee  ${logpath}/$time.log
```

node2
```bash
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export ROCSHMEM_MAX_NUM_CONTEXTS=48
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_HEAP_SIZE=1900000000
export ROCSHMEM_TOPO_FILE_FORCE="/shangxl/Test/2p4d/config/topo.config"
export USE_SPE_MQP=1
export SGLANG_TORCH_PROFILER_DIR=/shangxl/Test/2p4d/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export MC_TOPO_FILE_FORCE=/shangxl/Test/2p4d/config/mc_topo.config

#kv layout
export SGLANG_KV_LAYOUT_DCU_FA=1

export MOONCAKE_TE_META_DATA_SERVER="http://xxx:8080/metadata"
export MOONCAKE_GLOBAL_SEGMENT_SIZE="128GB"
export MOONCAKE_PROTOCOL="tcp"
export DEVICE_LIST="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
export MOONCAKE_DEVICE="$DEVICE_LIST"
export MOONCAKE_MASTER=xxx:50051

sysctl -w kernel.numa_balancing=0

model_path=/localmodels/Qwen3.5-397B-A17B
#model_path=/model/DeepSeek-R1-Distill-Qwen-7B
model=${model_path##*/}
tp=16 #K100AI为8
pp=1
dp=1
ep=1
nodes=2
rank=1
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=xxxx
max_model_len=6000
gpu_mem=0.80
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4"
option+=" --mamba-scheduler-strategy extra_buffer"
option+=" --chunked-prefill-size 16384"
option+=" --enable-hierarchical-cache --hicache-ratio 2 --hicache-mem-layout dcu_layout --hicache-io-backend kernel --hicache-storage-backend mooncake --hicache-write-policy write_through --hicache-storage-prefetch-policy timeout"


#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --host $host_ip  --port 30000 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64 2>&1 | tee  ${logpath}/$time.log
```

node1或者node2中任选一个节点开启mooncake_master,--http_metadata_server_host 、MOONCAKE_TE_META_DATA_SERVER、MOONCAKE_MASTER设置为选择的节点的ip
```plaintext
mooncake_master --enable_http_metadata_server=true --http_metadata_server_host=xxx --http_metadata_server_port=8080
```

# PD分离

hicache+pd分离主要有两种配置

1. prefill only:仅在prefill节点上启用 HiCache，实现prefill node之间的kv cache共享。
1. Full HiCache with async offloading:在prefill node上启用 HiCache，在decode node上启用异步kv cache 卸载，使得在多轮对话场景中，prefill node能够复用来自decode node的kv cache。

以Full HiCache with async offloading为例：

### P节点
```sql
mkdir -p prefill_logs
time=$(date "+%m%d-%H%M")
export NODE0_IP="xxx"
export MODEL_PATH="/localmodels/GLM-5-W8A8"
export NCCL_SOCKET_IFNAME="ens14f0"
export GLOO_SOCKET_IFNAME="ens14f0"
export NCCL_IB_HCA="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9"
export NCCL_IB_GID_INDEX=3
export USE_DCU_CUSTOM_ALLREDUCE=1
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
export MC_GID_INDEX=0
export SGLANG_USE_LIGHTOP=1
export SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_HOST_IP=10.16.1.22
# export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=600

export MOONCAKE_TE_META_DATA_SERVER="http://xxx:8080/metadata"
export MOONCAKE_GLOBAL_SEGMENT_SIZE="64GB"
export MOONCAKE_PROTOCOL="tcp"
export MOONCAKE_DEVICE="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9"
export MOONCAKE_MASTER=xxx:50051

sysctl -w kernel.numa_balancing=0 || true

python3 -m sglang.launch_server \
  --model-path "${MODEL_PATH}" \
  --host "${NODE0_IP}" \
  --port 30000 \
  --nnodes 1 \
  --node-rank 0 \
  --dist-init-addr "${NODE0_IP}:5000" \
  --tp-size 8 \
  --pp-size 1 \
  --dp-size 1 \
  --ep-size 1 \
  --attn-cp-size 8 \
  --trust-remote-code \
  --dtype bfloat16 \
  --kv-cache-dtype fp8_e4m3 \
  --mem-fraction-static 0.85 \
  --context-length 131072 \
  --page-size 64 \
  --chunked-prefill-size 8192 \
  --pp-max-micro-batch-size 1 \
  --disable-cuda-graph \
  --enable-nsa-prefill-context-parallel \
  --nsa-prefill-cp-mode round-robin-split \
  --nsa-prefill-backend flashmla_sparse \
  --nsa-decode-backend flashmla_kv \
  --quantization slimquant_marlin \
  --disaggregation-ib-device "${NCCL_IB_HCA}" \
  --disaggregation-mode prefill \
  --enable-hierarchical-cache \
  --hicache-ratio 1 \
  --hicache-mem-layout page_first \
  --hicache-io-backend kernel \
  --hicache-write-policy write_through \
  --hicache-storage-backend mooncake \
  --hicache-storage-prefetch-policy best_effort \
  2>&1 | tee prefill_logs/$time.log

```

### D节点
```sql
mkdir -p decode_logs
time=$(date "+%m%d-%H%M")
export NODE2_IP="xxx"   # Decode master IP
export MODEL_PATH="/localmodels/GLM-5-W8A8"

export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
# export SGLANG_ENABLE_SPEC_V2=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/profiling
export HSA_ENABLE_COREDUMP=1
export USE_DCU_CUSTOM_ALLREDUCE=1
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1
export SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export HIP_KERNEL_BATCH_CEILING=100
export GPU_FORCE_BLIT_COPY_SIZE=16
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export SGLANG_USE_LIGHTOP=1
export SGLANG_ROCM_USE_AITER_MOE=0
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=1
export HIP_GRAPH_USE_CMD_CACHE=0
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export NCCL_IB_HCA=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=ens14f0
export GLOO_SOCKET_IFNAME=ens14f0
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
export MC_GID_INDEX=0
export SGLANG_HOST_IP=xxx
# export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=600

export MOONCAKE_TE_META_DATA_SERVER="http://xxxx:8080/metadata"
export MOONCAKE_GLOBAL_SEGMENT_SIZE="64GB"
export MOONCAKE_PROTOCOL="tcp"
export MOONCAKE_DEVICE="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9"
export MOONCAKE_MASTER=xxxx:50051

sysctl -w kernel.numa_balancing=0 || true

python3 -m sglang.launch_server \
  --model-path "${MODEL_PATH}" \
  --host "${NODE2_IP}" \
  --port 30003 \
  --nnodes 1 \
  --node-rank 0 \
  --dist-init-addr "${NODE2_IP}:5000" \
  --tp-size 8 \
  --pp-size 1 \
  --dp-size 1 \
  --ep-size 1 \
  --trust-remote-code \
  --dtype bfloat16 \
  --kv-cache-dtype fp8_e4m3 \
  --mem-fraction-static 0.8 \
  --context-length 131072 \
  --page-size 64 \
  --disable-radix-cache \
  --chunked-prefill-size -1 \
  --nsa-prefill-backend flashmla_sparse \
  --nsa-decode-backend flashmla_kv \
  --quantization slimquant_marlin \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --moe-dense-tp-size 1 \
  --cuda-graph-max-bs 128 \
  --max-running-requests 512 \
  --disaggregation-ib-device "${NCCL_IB_HCA}" \
  --disaggregation-mode decode \
  --hicache-ratio 1 \
  --hicache-mem-layout page_first \
  --hicache-io-backend kernel \
  --hicache-write-policy write_through \
  --hicache-storage-backend mooncake \
  --hicache-storage-prefetch-policy best_effort \
  --disaggregation-decode-enable-offload-kvcache \
  2>&1 | tee decode_logs/$time.log

```

### Mooncake
```plaintext
mooncake_master --enable_http_metadata_server=true --http_metadata_server_host=xxx --http_metadata_server_port=8080
```

### 注意：

<callout emoji="musical_score" background-color="light-orange" border-color="light-orange">

1.p节点设置--enable-hierarchical-cache，而d节点设置--disaggregation-decode-enable-offload-kvcache

2.pd分离时，环境变量MOONCAKE_TE_META_DATA_SERVER、MOONCAKE_MASTER的ip设置为p节点或者d节点中的某一个节点的ip,该节点需要单独开启mooncake_master
</callout>

# 文件

<view type="1">

  <file token="OjhPbesdtoDbK8xbjwacdpggnNa" name="mooncake_transfer_engine-0.3.10.post1+das.opt1.dtk2604-cp310-cp310-manylinux_2_35_x86_64.whl"/>

</view>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Qwen-7B|DeepSeek-R1-Distill-Qwen-7B]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/GLM-w4a8-V2_6_test|GLM-w4a8-V2_6_test]]（model）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/Qwen3.5-397B-A17B|Qwen3.5-397B-A17B]]（model）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）

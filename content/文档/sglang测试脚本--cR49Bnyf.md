---
title: sglang测试脚本
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Distill-Qwen-7B
  - GLM-5-FP8
  - GLM-5-W8A8
  - GLM5-Channelwise-FP8-quantized
  - glm5-int8
  - Qwen3.5-397b
  - Qwen3.5-397B-A17B-Channel-FP8
framework:
  - Mooncake
  - SGLang
  - vLLM
issue_type:
  - 精度问题
source: feishu
feishu_token: TGPew8B0jiAbmHkwuLscR49Bnyf
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/TGPew8B0jiAbmHkwuLscR49Bnyf
last_synced_at: 2026-06-07T17:07:53.393Z
---

# sglang测试脚本

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/TGPew8B0jiAbmHkwuLscR49Bnyf)

# glm5-int8

【tp8】
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_TORCH_PROFILER_DIR=/workspace/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0

#mtp overlap
export SGLANG_ENABLE_SPEC_V2=1
# # triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16

# mooncake PD分离
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ALLREDUCE_STREAM_WITH_COMPUTE=1
# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
export SGLANG_ROCM_USE_AITER_MOE=0
#export TVM_HOME=/workspace/tilelang/3rdparty/tvm
#export TVM_LIBRARY_PATH=$TVM_HOME/build
#export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH
#export W8A8_SUPPORT_METHODS=3

model_path=/module/GLM-5-W8A8
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
attn_cp_size=8
nodes=1
rank=0
host_ip=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${host_ip}" ]; then
    host_ip=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
master_ip=$host_ip
max_model_len=6000
gpu_mem=0.8
port=30000
dist_port=5000
time=$(date "+%m%d-%H%M")
logpath="dserver/${model}-tp${tp}-dp${dp}-ep${ep}-pp${pp}-cp${attn_cp_size}-$(hostname)"
logfile="${logpath}/glm5_int8_${time}.log"


mkdir -p "${logpath}"


option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size 16384"
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
option+=" --quantization slimquant_marlin "
#option+=" --attn-cp-size ${attn_cp_size} "
#option+=" --enable-nsa-prefill-context-parallel  --nsa-prefill-cp-mode round-robin-split "
#option+=" --disable-cuda-graph "
#option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-mode prefill "
#option+=" --pp-max-micro-batch-size 1"

python3 -m sglang.launch_server --model-path "${model_path}" ${option} \
                                --trust-remote-code \
                                --kv-cache-dtype fp8_e4m3 --dtype bfloat16 --mem-fraction-static "${gpu_mem}" \
                                --host "${host_ip}" --port "${port}" --dist-init-addr "${master_ip}:${dist_port}" \
                                --nnodes "${nodes}" --node-rank "${rank}" \
                                --tp-size "${tp}" --pp-size "${pp}" --dp-size "${dp}" --ep-size "${ep}" \
                                2>&1 | tee "${logfile}"

```

【cp8tp8】
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_TORCH_PROFILER_DIR=/workspace/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0

#mtp overlap
export SGLANG_ENABLE_SPEC_V2=1
# # triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
# deep_ep
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_TOPO_FILE_FORCE=/home/topo.config
# mooncake
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ALLREDUCE_STREAM_WITH_COMPUTE=1
# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
export SGLANG_ROCM_USE_AITER_MOE=0
#export TVM_HOME=/workspace/tilelang/3rdparty/tvm
#export TVM_LIBRARY_PATH=$TVM_HOME/build
#export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH
#export W8A8_SUPPORT_METHODS=3

model_path=/module/GLM-5-W8A8
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
attn_cp_size=8
nodes=1
rank=0
host_ip=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${host_ip}" ]; then
    host_ip=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
master_ip=$host_ip
max_model_len=6000
gpu_mem=0.8
port=30000
dist_port=5000
time=$(date "+%m%d-%H%M")
logpath="dserver/${model}-tp${tp}-dp${dp}-ep${ep}-pp${pp}-cp${attn_cp_size}-$(hostname)"
logfile="${logpath}/glm5_int8_${time}.log"


mkdir -p "${logpath}"


option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1 "
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
option+=" --quantization slimquant_marlin "
option+=" --attn-cp-size ${attn_cp_size} "
option+=" --enable-nsa-prefill-context-parallel  --nsa-prefill-cp-mode round-robin-split "
option+=" --disable-cuda-graph "
#option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-mode prefill "
#option+=" --pp-max-micro-batch-size 1"

python3 -m sglang.launch_server --model-path "${model_path}" ${option} \
                                --trust-remote-code \
                                --kv-cache-dtype fp8_e4m3 --dtype bfloat16 --mem-fraction-static "${gpu_mem}" \
                                --host "${host_ip}" --port "${port}" --dist-init-addr "${master_ip}:${dist_port}" \
                                --nnodes "${nodes}" --node-rank "${rank}" \
                                --tp-size "${tp}" --pp-size "${pp}" --dp-size "${dp}" --ep-size "${ep}" \
                                2>&1 | tee "${logfile}"

```

glmd.sh
```markdown {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

# export LD_LIBRARY_PATH=/workspace/nvshmem_install/lib:$LD_LIBRARY_PATH
# export DEEP_EP_DEVICE_TO_HCA_MAPPING=0:mlx5_2:1,1:mlx5_3:1,2:mlx5_4:1,3:mlx5_5:1,4:mlx5_6:1,5:mlx5_7:1,6:mlx5_8:1,7:mlx5_9:1
# export NVSHMEM_SYMMETRIC_SIZE=1900000000

# export ROCSHMEM_MAX_NUM_CONTEXTS=48
# export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
# export ROCSHMEM_HEAP_SIZE=1900000000
# export ROCSHMEM_TOPO_FILE_FORCE="/home/pd_config/mc_topo.config"


export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
#export GPU_MAX_HW_QUEUES=3
#sysctl -w kernel.numa_balancing=0

#mtp overlap
export SGLANG_ENABLE_SPEC_V2=1
# # triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
#export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
#export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
#export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
#export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
#export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
#export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072


export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16


export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # 大ep下无效
export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat
export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条
export SGLANG_USE_FUSED_RMSNORM_ROPE=0  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
export SGLANG_USE_FUSED_RMS_QUANT=0   #glm 有精度问题                                                                                                       
export USE_FUSED_RMS_QUANT=1 # lmslim and vllm
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # w8a8下未适配
export SGLANG_USE_RMS_QUANT_PATH=0


export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
export SGLANG_KV_LAYOUT_DCU_FA=0

export SGLANG_USE_FP8_W8A8_MOE=1
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
#export W8A8_SUPPORT_METHODS=3

# export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
# export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
# export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1

# tvm_ffi
# export USE_ROCM=1
# export TLE_BACKEND=ROCM
# export TLE_USE_CUDA=OFF
# export TLE_USE_ROCM=ON
# export TVM_FFI_DISABLE_TORCH_DLPACK=1

#export TVM_HOME=/workspace/tilelang/3rdparty/tvm
#export TVM_LIBRARY_PATH=$TVM_HOME/build
#export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH
#export PYTHONPATH=/workspace/tilelang:$PYTHONPATH

# export CUDA_HOME=/opt/dtk
# export CPLUS_INCLUDE_PATH=/opt/dtk/include:$CPLUS_INCLUDE_PATH

#/models/GLM-5-FP8 /model/GLM5-Channelwise-FP8-quantized /model/ZhipuAI/GLM-5-FP8 #
#model_path=/module2/GLM-5-W8A8
model_path=/module/GLM-5-W8A8
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
#max_model_len=6000
gpu_mem=0.8
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tpp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

# export HSA_ENABLE_COREDUMP=1
# export HSA_ENABLE_COREDUMP_MEMORY=1
# export HSA_COREDUMP_SAVE_DIR=./home/mtp/
#export GPU_FLUSH_ON_EXECUTION=true #报错调用栈
# export HIPBLASLT_LOG_FILE=hipblaslt-log.log
# export HIPBLASLT_LOG_MASK=32

#export GPU_FLUSH_ON_EXECUTION=1
# export TORCH_USE_CUDA_DSA=1
#export TORCHDYNAMO_DISABLE=1
option="--numa-node 3 1 1 0 7 5 5 4"
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size -1"
# export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
# export CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
#option+=" --max-running-requests 1024 "
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
# option+=" --ep-size $ep "
# option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head"
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
# option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 2"
#option+=" --cuda-graph-max-bs 128 "
#option+=" --page-size 64" --nsa-prefill-backend flashmla_sparse
#option+=" --context-length 6000"
option+=" --disable-cuda-graph  " #--skip-server-warmup
option+=" --quantization slimquant_marlin  "
#option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
#option+=" --disaggregation-mode decode "

# option+=" --quantization w8a8_fp8  "
#option+=" --disaggregation-prefill-pp 8"
# option+=" --enable-single-batch-overlap"
# option+=" --enable-dp-attention --moe-a2a-backend deepep  --deepep-mode low_latency"
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
# option+=" --disaggregation-ib-device mlx5_9 --quantization fp8 --kv-cache-dtype bf16 " --quantization w8a8_int8  --json-model-override-args '{"num_hidden_layers": 6}'
#--quantization slimquant_w4a8_marlin --attention-backend dcu_mla --kv-cache-dtype fp8_e4m3 --quantization slimquant_marlin
python3 -m sglang.launch_server --model-path $model_path  ${option} \
                                --trust-remote-code \
                             --kv-cache-dtype fp8_e4m3  --dtype bfloat16 --mem-fraction-static $gpu_mem \
                                --host $host_ip --port 30000 --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --tp-size $tp --dp-size $dp 2>&1 | tee  ${logpath}-test--tensor$hostname-$time.log

```

ep16.sh
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

# =========================
# 环境变量
# =========================
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_ENABLE_SPEC_V2=1
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
export W8A8_SUPPORT_METHODS=3
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=1
export HIP_GRAPH_USE_CMD_CACHE=0
# export TVM_HOME=/workspace/tilelang/3rdparty/tvm
# export TVM_LIBRARY_PATH=$TVM_HOME/build
# export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_TOPO_FILE_FORCE=/mnt/gbj/topo_400g.config
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1
export ROCBLAS_TENSILE_LIBPATH=/mnt/gbj/rocblas/library_gpu6_glm5_int8

# 可选调试项，默认关闭
# 抓取gemm size
# export ROCBLAS_LAYER=3
# export HIPBLASLT_LOG_LEVEL=3
# 锁定mtp接收率
# export SGLANG_SIMULATE_ACC_LEN=3.2
# export SGLANG_SIMULATE_ACC_METHOD=match-expected
# 抓取hiplog
# export HIP_LOG_LEVEL=4
# export HIP_MODULE_MASK=0x7fffffff
# export HIP_ENABLE_LIST=hip
# export HIP_LOG_OUTPUT_PATH=/workspace/hip_log

# =========================
# 可改配置
# =========================
MODEL_PATH="/module/GLM-5-W8A8/"

# 自动获取本机 IP
HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi

PORT="30023"

NNODES="2"
NODE_RANK="0"

MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

TP_SIZE="16"
PP_SIZE="1"
DP_SIZE="16"
EP_SIZE="16"

CUDA_GRAPH_MAX_BS="16"
MAX_RUNNING_REQUESTS="192"

SPEC_ALGO="EAGLE"
SPEC_NUM_STEPS="3"
SPEC_TOPK="1"
SPEC_NUM_DRAFT_TOKENS="4"

MEM_FRACTION_STATIC="0.8"
CONTEXT_LENGTH="131072"
PAGE_SIZE="64"
KV_CACHE_DTYPE="fp8_e4m3"

DISAGG_MODE="decode"
IB_DEVICES="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

# =========================
# 日志配置
# =========================
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_${SPEC_ALGO,,}_s${SPEC_NUM_STEPS}_k${SPEC_TOPK}_d${SPEC_NUM_DRAFT_TOKENS}_tp${TP_SIZE}_pp${PP_SIZE}_dp${DP_SIZE}_ep${EP_SIZE}_node${NODE_RANK}_${HOST_NAME}_${TIME_STR}.log"

echo "========================================"
echo "log file: $LOG_FILE"
echo "host    : $HOST"
echo "port    : $PORT"
echo "master  : $MASTER_IP"
echo "dist    : $DIST_INIT_ADDR"
echo "node    : $NODE_RANK/$NNODES"
echo "tp/pp/dp/ep: ${TP_SIZE}/${PP_SIZE}/${DP_SIZE}/${EP_SIZE}"
echo "spec    : ${SPEC_ALGO} steps=${SPEC_NUM_STEPS} topk=${SPEC_TOPK} draft=${SPEC_NUM_DRAFT_TOKENS}"
echo "========================================"

python3 -m sglang.launch_server \
  --pp-size "${PP_SIZE}" \
  --tp-size "${TP_SIZE}" \
  --nsa-prefill-backend flashmla_auto \
  --nsa-decode-backend flashmla_kv \
  --context-length "${CONTEXT_LENGTH}" \
  --trust-remote-code \
  --dtype bfloat16 \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --page-size "${PAGE_SIZE}" \
  --kv-cache-dtype "${KV_CACHE_DTYPE}" \
  --model-path "${MODEL_PATH}" \
  --mem-fraction-static "${MEM_FRACTION_STATIC}" \
  --disable-radix-cache \
  --chunked-prefill-size -1 \
  --quantization slimquant_marlin \
  --cuda-graph-max-bs "${CUDA_GRAPH_MAX_BS}" \
  --moe-dense-tp-size 1 \
  --dp-size "${DP_SIZE}" \
  --ep-size "${EP_SIZE}" \
  --enable-dp-attention \
  --moe-a2a-backend deepep \
  --enable-dp-lm-head \
  --port "${PORT}" \
  --host "${HOST}" \
  --disaggregation-ib-device "${IB_DEVICES}" \
  --disaggregation-mode "${DISAGG_MODE}" \
  --max-running-requests "${MAX_RUNNING_REQUESTS}" \
  --speculative-algorithm "${SPEC_ALGO}" \
  --speculative-num-steps "${SPEC_NUM_STEPS}" \
  --speculative-eagle-topk "${SPEC_TOPK}" \
  --speculative-num-draft-tokens "${SPEC_NUM_DRAFT_TOKENS}" \
  --nnodes "${NNODES}" \
  --node-rank "${NODE_RANK}" \
  --dist-init-addr "${DIST_INIT_ADDR}" \
  2>&1 | tee "$LOG_FILE"

```

ep8.sh
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

# =========================
# 环境变量
# =========================
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_ENABLE_SPEC_V2=1
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
export W8A8_SUPPORT_METHODS=3
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=1
export HIP_GRAPH_USE_CMD_CACHE=0
# export TVM_HOME=/workspace/tilelang/3rdparty/tvm
# export TVM_LIBRARY_PATH=$TVM_HOME/build
# export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_TOPO_FILE_FORCE=/mnt/gbj/topo_400g.config
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1
export ROCBLAS_TENSILE_LIBPATH=/mnt/gbj/rocblas/library_gpu6_glm5_int8

# 可选调试项，默认关闭
# 抓取gemm size
# export ROCBLAS_LAYER=3
# export HIPBLASLT_LOG_LEVEL=3
# 锁定mtp接收率
# export SGLANG_SIMULATE_ACC_LEN=3.2
# export SGLANG_SIMULATE_ACC_METHOD=match-expected
# 抓取hiplog
# export HIP_LOG_LEVEL=4
# export HIP_MODULE_MASK=0x7fffffff
# export HIP_ENABLE_LIST=hip
# export HIP_LOG_OUTPUT_PATH=/workspace/hip_log

# =========================
# 可改配置
# =========================
MODEL_PATH="/module/GLM-5-W8A8/"

# 自动获取本机 IP
HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi

PORT="30023"

NNODES="1"
NODE_RANK="0"

MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

TP_SIZE="8"
PP_SIZE="1"
DP_SIZE="8"
EP_SIZE="8"

CUDA_GRAPH_MAX_BS="16"
MAX_RUNNING_REQUESTS="192"

SPEC_ALGO="EAGLE"
SPEC_NUM_STEPS="3"
SPEC_TOPK="1"
SPEC_NUM_DRAFT_TOKENS="4"

MEM_FRACTION_STATIC="0.8"
CONTEXT_LENGTH="131072"
PAGE_SIZE="64"
KV_CACHE_DTYPE="fp8_e4m3"

DISAGG_MODE="decode"
IB_DEVICES="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

# =========================
# 日志配置
# =========================
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_${SPEC_ALGO,,}_s${SPEC_NUM_STEPS}_k${SPEC_TOPK}_d${SPEC_NUM_DRAFT_TOKENS}_tp${TP_SIZE}_pp${PP_SIZE}_dp${DP_SIZE}_ep${EP_SIZE}_node${NODE_RANK}_${HOST_NAME}_${TIME_STR}.log"

echo "========================================"
echo "log file: $LOG_FILE"
echo "host    : $HOST"
echo "port    : $PORT"
echo "master  : $MASTER_IP"
echo "dist    : $DIST_INIT_ADDR"
echo "node    : $NODE_RANK/$NNODES"
echo "tp/pp/dp/ep: ${TP_SIZE}/${PP_SIZE}/${DP_SIZE}/${EP_SIZE}"
echo "spec    : ${SPEC_ALGO} steps=${SPEC_NUM_STEPS} topk=${SPEC_TOPK} draft=${SPEC_NUM_DRAFT_TOKENS}"
echo "========================================"

python3 -m sglang.launch_server \
  --pp-size "${PP_SIZE}" \
  --tp-size "${TP_SIZE}" \
  --nsa-prefill-backend flashmla_auto \
  --nsa-decode-backend flashmla_kv \
  --context-length "${CONTEXT_LENGTH}" \
  --trust-remote-code \
  --dtype bfloat16 \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --page-size "${PAGE_SIZE}" \
  --kv-cache-dtype "${KV_CACHE_DTYPE}" \
  --model-path "${MODEL_PATH}" \
  --mem-fraction-static "${MEM_FRACTION_STATIC}" \
  --disable-radix-cache \
  --chunked-prefill-size -1 \
  --quantization slimquant_marlin \
  --cuda-graph-max-bs "${CUDA_GRAPH_MAX_BS}" \
  --moe-dense-tp-size 1 \
  --dp-size "${DP_SIZE}" \
  --ep-size "${EP_SIZE}" \
  --enable-dp-attention \
  --moe-a2a-backend deepep \
  --enable-dp-lm-head \
  --port "${PORT}" \
  --host "${HOST}" \
  --disaggregation-ib-device "${IB_DEVICES}" \
  --disaggregation-mode "${DISAGG_MODE}" \
  --max-running-requests "${MAX_RUNNING_REQUESTS}" \
  --speculative-algorithm "${SPEC_ALGO}" \
  --speculative-num-steps "${SPEC_NUM_STEPS}" \
  --speculative-eagle-topk "${SPEC_TOPK}" \
  --speculative-num-draft-tokens "${SPEC_NUM_DRAFT_TOKENS}" \
  --nnodes "${NNODES}" \
  --node-rank "${NODE_RANK}" \
  --dist-init-addr "${DIST_INIT_ADDR}" \
  2>&1 | tee "$LOG_FILE"

```

glm_int8c.sh
```bash {wrap}
model_path=/module/GLM-5-W8A8

model=${model_path##*/}
time=$(date "+%m%d-%H%M")
mode="cudagraph"
rr=inf
logpath="client-ali-random/${model}-${mode}-$time-$mode-rr$rr-mem09-$(hostname)"
port=30010
#hostname=$hostname
concurrency_multiplier=1
host_ip=$(hostname -I | awk '{print $1}')
#host_ip=10.16.1.55
#rr=inf
#batch=136
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

pairs=("4000 20")

echo "input,output,request_rate,num_prompts,max_concurrency,concurrency,Peak_concurrent_requests,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,Peak_output_token_throughput,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log
#--sharegpt-range-ratio 1
#for pair in "${pairs[@]}"; do
#for concurrency_multiplier in 1 ;do
for rr  in 1.7 ; do
#        for batch in 100 128 136 140 160  ; do
#        for batch in $(seq 16 4 80) ; do
       for batch in 2000 ; do

#        prompt_tokens=${pair%% *}
#        completion_tokens=${pair#* }
        prompt_tokens=4000
        completion_tokens=2000
        python3 -m sglang.bench_serving --backend sglang \
        --model ${model_path}   --request-rate $rr \
        --port $port --host $host_ip \
        --dataset-name random \
        --random-range-ratio 1 --tokenize-prompt \
        --dataset-path /mnt/gbj/ShareGPT_V3_unfiltered_cleaned_split.json \
        --random-input-len $prompt_tokens \
        --random-output-len $completion_tokens \
        --num-prompts $((batch*concurrency_multiplier))  2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log

        #Total token throughput
        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        request_rate=`grep -a "^Traffic request rate" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
#        Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $2}'`
        Concurrency=`grep -a "^Successful requests:" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $3}'`
        Peak_output_token_throughput=`grep -a "^Peak output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $6}'`
        Peak_concurrent_requests=`grep -a "^Peak concurrent requests" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`



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
        P50_TTFT=`grep -a "^P50 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P50_TPOT=`grep -a "^P50 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P50_ITL=`grep -a "^P50 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_ITL=`grep -a "^Mean ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_TTFT=`grep -a "^P95 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_TPOT=`grep -a "^P95 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_ITL=`grep -a "^P95 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`





        echo "$prompt_tokens,$completion_tokens,$request_rate,$((batch*concurrency_multiplier)),$batch,$Concurrency,$Peak_concurrent_requests,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Peak_concurrent_requests,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log


     done
done

```

# Qwen3.5-397b
```shell {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1


# export LD_LIBRARY_PATH=/workspace/nvshmem_install/lib:$LD_LIBRARY_PATH
# export DEEP_EP_DEVICE_TO_HCA_MAPPING=0:mlx5_2:1,1:mlx5_3:1,2:mlx5_4:1,3:mlx5_5:1,4:mlx5_6:1,5:mlx5_7:1,6:mlx5_8:1,7:mlx5_9:1
# export NVSHMEM_SYMMETRIC_SIZE=1900000000
# export ROCSHMEM_TOPO_FILE="/workspace/built-in-BW-topo-input.xml"
export ROCSHMEM_MAX_NUM_CONTEXTS=48
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_HEAP_SIZE=1900000000
export ROCSHMEM_TOPO_FILE_FORCE="/shangxl/Test/2p4d/config/topo.config"
export USE_SPE_MQP=1

export SGLANG_TORCH_PROFILER_DIR=/mnt/gbj/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0

#mtp overlap
# export SGLANG_ENABLE_SPEC_V2=1
# triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
# export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
# export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
# export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
# export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
# export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
# export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072


export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16


export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export SGLANG_USE_FP8_W8A8_MOE=1
# export SGLANG_USE_FUSED_RMS_QUANT=1
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  # 大ep下无效
# export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat
# export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条
# export SGLANG_USE_FUSED_RMSNORM_ROPE=1  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
# export SGLANG_USE_FUSED_RMS_QUANT=0
# export USE_FUSED_RMS_QUANT=1  # lmslim and vllm
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # w8a8下未适配
# export SGLANG_USE_RMS_QUANT_PATH=1

# export SGLANG_SIMULATED_EXPERT_BALANCE=1
# export LD_LIBRARY_PATH=/shangxl/environment/rocblas-install/lib:$LD_LIBRARY_PATH

#eplb
# export NCCL_NCHANNELS_PER_PEER=2
# export NCCL_MAX_P2P_NCHANNELS=8
# export NCCL_MIN_P2P_NCHANNELS=8

#mooncake
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config

#kv layout
# export SGLANG_KV_LAYOUT_DCU_FA=0

# export MOONCAKE_TE_META_DATA_SERVER="http://10.16.1.69:8080/metadata"
# export MOONCAKE_GLOBAL_SEGMENT_SIZE=816043786240
# export MOONCAKE_PROTOCOL="rdma"
# export DEVICE_LIST="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
# export MOONCAKE_DEVICE="$DEVICE_LIST"
# export MOONCAKE_MASTER=10.16.1.69:50051

#export HSA_ENABLE_COREDUMP=1
#export HSA_ENABLE_COREDUMP_MEMORY=1
#export SGLANG_USE_FUSED_TOPK_SOFTMAX=0 #精度有问题
#export HSA_COREDUMP_SAVE_DIR=/shangxl/Test/qwen/tp16/tmp/
#export GPU_FLUSH_ON_EXECUTION=1

model_path=/module/Qwen3.5-397B-A17B-Channel-FP8
#model_path=/model/DeepSeek-R1-Distill-Qwen-7B
model=${model_path##*/}
tp=4
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
#max_model_len=6500
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4"
#option+=" --mm-attention-backend fa3"
option+=" --disable-radix-cache "
#option+=" --mamba-scheduler-strategy extra_buffer"
#option+=" --mamba-scheduler-strategy no_buffer"
option+=" --chunked-prefill-size -1"
#option+=" --max-running-requests 160 "
#option+=" --disable-cuda-graph "
#option+=" --cuda-graph-max-bs 200"
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1"
#option+=" --enable-torch-compile --torch-compile-max-bs 128"
#option+=" --pp-max-micro-batch-size 2"
#option+=" --schedule-conservativeness 0.3"
# option+=" --enable-hierarchical-cache --hicache-ratio 2 --hicache-mem-layout layer_first --hicache-io-backend kernel --hicache-storage-backend mooncake --hicache-write-policy write_through --hicache-storage-prefetch-policy timeout"
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode prefill "


#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
# python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
sglang serve --model-path $model_path ${option} \
                                --host $host_ip  --port 30000 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16 --kv-cache fp8_e4m3  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64 2>&1 | tee  ${logpath}/$time.log

```

ep.sh
```shell {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

# export LD_LIBRARY_PATH=/workspace/nvshmem_install/lib:$LD_LIBRARY_PATH
# export DEEP_EP_DEVICE_TO_HCA_MAPPING=0:mlx5_2:1,1:mlx5_3:1,2:mlx5_4:1,3:mlx5_5:1,4:mlx5_6:1,5:mlx5_7:1,6:mlx5_8:1,7:mlx5_9:1
# export NVSHMEM_SYMMETRIC_SIZE=1900000000

export ROCSHMEM_MAX_NUM_CONTEXTS=48
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
#export ROCSHMEM_TOPO_FILE_FORCE="/mnt/gbj//topo_400g.config"


export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
#export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0

#mtp overlap
export SGLANG_ENABLE_SPEC_V2=1
# # triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
#export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
#export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
#export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
#export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
#export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
#export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072


export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16


export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # 大ep下无效
export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat
#export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条
export SGLANG_USE_FUSED_RMSNORM_ROPE=0  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
export SGLANG_USE_FUSED_RMS_QUANT=0   #glm 有精度问题                                                                                                       
#export USE_FUSED_RMS_QUANT=1 # lmslim and vllm
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # w8a8下未适配
export SGLANG_USE_RMS_QUANT_PATH=0


export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
export SGLANG_KV_LAYOUT_DCU_FA=0

export SGLANG_USE_FP8_W8A8_MOE=1
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1



model_path=/module/Qwen3.5-397B-A17B-Channel-FP8
model=${model_path##*/}
tp=8 #K100AI为8
pp=1
dp=8
ep=8
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
#max_model_len=6000
gpu_mem=0.8
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tpp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

# export HSA_ENABLE_COREDUMP=1
# export HSA_ENABLE_COREDUMP_MEMORY=1
# export HSA_COREDUMP_SAVE_DIR=./home/mtp/
#export GPU_FLUSH_ON_EXECUTION=true #报错调用栈
# export HIPBLASLT_LOG_FILE=hipblaslt-log.log
# export HIPBLASLT_LOG_MASK=32

#export GPU_FLUSH_ON_EXECUTION=1
# export TORCH_USE_CUDA_DSA=1
#export TORCHDYNAMO_DISABLE=1
option="--numa-node 3 1 1 0 7 5 5 4"
option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size -1"
# export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
# export CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
#option+=" --max-running-requests 1024 "
option+=" --page-size 64 "
#option+=" --mamba-scheduler-strategy extra_buffer"

#option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
# option+=" --ep-size $ep "
#--moe-dense-tp-size 1
option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1  --enable-dp-lm-head"
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
# option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 2"
option+=" --cuda-graph-max-bs 32 "
#option+=" --attention-backend fa3"
#option+=" --page-size 64" --nsa-prefill-backend flashmla_sparse
#option+=" --context-length 6000"
#option+=" --disable-cuda-graph  " #--skip-server-warmup
#option+=" --quantization slimquant_marlin  "
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode decode "

# option+=" --quantization w8a8_fp8  "
#option+=" --disaggregation-prefill-pp 8"
# option+=" --enable-single-batch-overlap"
option+=" --enable-dp-attention --moe-a2a-backend deepep  --deepep-mode low_latency"
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
# option+=" --disaggregation-ib-device mlx5_9 --quantization fp8 --kv-cache-dtype bf16 " --quantization w8a8_int8  --json-model-override-args '{"num_hidden_layers": 6}'
#--quantization slimquant_w4a8_marlin --attention-backend dcu_mla --kv-cache-dtype fp8_e4m3 --quantization slimquant_marlin
python3 -m sglang.launch_server --model-path $model_path  ${option} \
                                --trust-remote-code \
                             --kv-cache-dtype fp8_e4m3  --dtype bfloat16 --mem-fraction-static $gpu_mem \
                                --host $host_ip --port 30000 --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --tp-size $tp --dp-size $dp 2>&1 | tee  ${logpath}-test--tensor$hostname-$time.log

```

# ling-1T
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100

export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
# export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
# export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
# export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
# export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
# export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
# export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16

export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # 大ep下无效
export SGLANG_USE_OPT_CAT=0  # 性能优化版q cat
export SGLANG_USE_FUSED_MLA_CAT=0  # q cat融入mla，不再需要上一条
export SGLANG_USE_FUSED_RMSNORM_ROPE=0  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
export SGLANG_USE_FUSED_RMS_QUANT=0
export USE_FUSED_RMS_QUANT=0  # lmslim and vllm
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # w8a8下未适配
export SGLANG_USE_RMS_QUANT_PATH=0
export SGLANG_USE_FP8_W8A8_MOE=1
export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1

# deep_ep
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_TOPO_FILE_FORCE=/mnt/liucong/NMZ/test/ling1T_1p4d/ep2/topo.config

# mooncake
export MC_TOPO_FILE_FORCE=/mnt/liucong/NMZ/mc_topo.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9

export DEEPEP_ENABLE_LL_DISPATCH_OPT=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要

export ROCBLAS_TENSILE_LIBPATH=/mnt/liucong/NMZ/test/ling1T_1p4d/ep2/library_gpu6_lmhead_bling

model_path=/module/Ling-1T-FP8
model=${model_path##*/}
tp=16 #K100AI为8
pp=1
dp=16
ep=16
nodes=2
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
#master_ip=$host_ip
master_ip=10.16.1.46
max_model_len=6000
gpu_mem=0.90
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
# option+=" --max-running-requests 64 "
option+=" --page-size 64"
option+=" --ep-size $ep "
option+=" --dp-size $dp --moe-dense-tp-size 1 --enable-dp-lm-head --enable-dp-attention"
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
# option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 2"
option+=" --cuda-graph-max-bs 64"
# option+=" --page-size 64"
# option+=" --disable-cuda-graph "
# option+=" --disable-shared-experts-fusion "
#option+=" --disaggregation-prefill-pp 8"
# option+=" --enable-single-batch-overlap"
option+=" --enable-dp-attention --moe-a2a-backend deepep --deepep-mode low_latency"
option+=" --disaggregation-mode decode --prefill-round-robin-balance "
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

python3 -m sglang.launch_server --model-path $model_path  ${option} \
                                --trust-remote-code --context-length 6000 \
                                --dtype bfloat16 --mem-fraction-static $gpu_mem \
                                --host $host_ip --port 30004 --dist-init-addr $master_ip:50000 --nnodes $nodes --node-rank $rank \
                                --tp-size $tp --attention-backend fa3 --kv-cache-dtype fp8_e4m3  \
                                                2>&1 | tee  ${logpath}-model_len$max_model_len-mem$gpu_mem-$time-$hostname.log

```

p.sh
```shell {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
# export GPU_MAX_HW_QUEUES=3
# sysctl -w kernel.numa_balancing=0
#export LD_LIBRARY_PATH=/mnt/gbj/hipblaslt-install/lib/:$LD_LIBRARY_PATH
#export LD_LIBRARY_PATH=/mnt/gbj/rocblas-install/lib/:$LD_LIBRARY_PATH
# #mtp overlap
# export SGLANG_ENABLE_SPEC_V2=1
# # triton改写算子
# export SGLANG_KVALLOC_KERNEL=1
# export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
# export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
# export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
# export SGLANG_GET_LAST_LOC=1
# export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
# export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
# export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
# export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
# export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
# export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
# export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
# export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072


export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16

# export GLOO_SOCKET_IFNAME=enp33s0f3u1


export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # 大ep下无效
# export SGLANG_USE_OPT_CAT=0  # 性能优化版q cat
# export SGLANG_USE_FUSED_MLA_CAT=0  # q cat融入mla，不再需要上一条
# export SGLANG_USE_FUSED_RMSNORM_ROPE=0  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
# export SGLANG_USE_FUSED_RMS_QUANT=0
# export USE_FUSED_RMS_QUANT=0  # lmslim and vllm
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # w8a8下未适配
# export SGLANG_USE_RMS_QUANT_PATH=0


# export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
# export HIP_GRAPH_USE_CMD_CACHE=0
# export SGLANG_USE_FUSED_BAILING_MOE_SUM_ADD=0
export SGLANG_USE_FP8_W8A8_MOE=1
export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1

# custom_allreduce no memory copy
export HIP_KERNEL_EVENT_SYSTENFENCE=1
#export SGLANG_DISAGGREGATION_THREAD_POOL_SIZE=8
#export SGLANG_DISAGGREGATION_QUEUE_SIZE=4
#export SGLANG_DISAGGREGATION_USE_ZERO_COPY=1
#export SGLANG_KV_LAYOUT_DCU_FA=0

# mooncake
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export MC_TOPO_FILE_FORCE=/mnt/liucong/NMZ/mc_topo.config
#export MC_IB_GID_INDEX=1

model_path=/module/Ling-1T-FP8
#model_path=/model/inclusionAI/Ling-1T-FP8
model=${model_path##*/}
tp=2
pp=4
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
# master_ip=10.16.1.36
max_model_len=6000
gpu_mem=0.95
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="pdserver-p/$model-tp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1 "
#option+=" --enable-request-time-stats-logging"
#option+=" --max-running-requests 2 "
# # option+=" --page-size 64"
# # option+=" --ep-size $ep "
# # option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head"
# # option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
# # option+=" --enable-torch-compile --torch-compile-max-bs 64"
export SGLANG_PP_LAYER_PARTITION="20,20,20,20"
#export SGLANG_PP_LAYER_PARTITION="40,40"
#export SGLANG_PP_LAYER_PARTITION="13,9,10,9,10,9,10,10"
# export SGLANG_PP_LAYER_PARTITION="10,10,10,10,10,10,10,10"
option+=" --pp-max-micro-batch-size 1"
#option+=" --cuda-graph-max-bs 170"
# # option+=" --page-size 64"
option+=" --context-length 6000"
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode prefill --load-balance-method round_robin "
option+=" --disable-cuda-graph "
# #option+=" --disaggregation-prefill-pp 8"
# # option+=" --enable-single-batch-overlap"
# # option+=" --enable-dp-attention --moe-a2a-backend deepep  --deepep-mode low_latency"
# #SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
#option+=" --disaggregation-ib-device mlx5_9"
python3 -m sglang.launch_server ${option}  \
  --trust-remote-code \
  --host $host_ip --port 30000 \
  --model-path $model_path \
  --attention-backend fa3 --page-size 64 \
  --tp-size $tp --kv-cache-dtype fp8_e4m3 --pp-size $pp \
  --mem-fraction-static $gpu_mem \
  2>&1 | tee ${logpath}-model_len$max_model_len-mem$gpu_mem-$time-$hostname.log

```

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/精度问题|精度问题]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Qwen-7B|DeepSeek-R1-Distill-Qwen-7B]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/EPLB|EPLB]]（concept）
- [[飞书知识图谱/实体/GLM-5-FP8|GLM-5-FP8]]（model）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/GLM5-Channelwise-FP8-quantized|GLM5-Channelwise-FP8-quantized]]（model）
- [[飞书知识图谱/实体/glm5-int8|glm5-int8]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/NVSHMEM|NVSHMEM]]（concept）
- [[飞书知识图谱/实体/P50|P50]]（metric）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/Qwen3.5-397b|Qwen3.5-397b]]（model）
- [[飞书知识图谱/实体/Qwen3.5-397B-A17B-Channel-FP8|Qwen3.5-397B-A17B-Channel-FP8]]（model）
- [[飞书知识图谱/实体/RDMA|RDMA]]（concept）
- [[飞书知识图谱/实体/RPS|RPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

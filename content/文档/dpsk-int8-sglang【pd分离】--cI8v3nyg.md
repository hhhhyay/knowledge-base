---
title: dpsk-int8-sglang【pd分离】
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Channel-INT8
framework:
  - SGLang
issue_type: []
source: feishu
feishu_token: C47JwNgDkirKUukpiHWcI8v3nyg
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/C47JwNgDkirKUukpiHWcI8v3nyg
last_synced_at: 2026-06-07T17:07:53.393Z
---

# dpsk-int8-sglang【pd分离】

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/C47JwNgDkirKUukpiHWcI8v3nyg)

环境配置：

镜像：~~/public/das/sglang/Images/sgl059_ep.tar.gz【千卡集群】~~

可以使用0416镜像：

[[飞书知识图谱/资源/大文件上传下载--cHqT4nFg|大文件上传下载]] 下载参考文档可以使用快传进行下载

rayfile-c -a wuzh01.hpccube.com -P 65012 -u guobj -w 5306a83256Z3VvYmo=_566ecfdbb066bc1c -no-meta -symbolic-links follow -retry 10 -retrytimeout 30 -o download -s '/images/sgl_das0416.tar.gz' -d <请输入下载目标全路径并替换尖括号及本内容> 

# P节点【pp4tp2-radix-cache】
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_TOPO_FILE_FORCE="/mnt/gbj/topo_400g.config"

export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_6,mlx5_7,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9
export ROCSHMEM_HEAP_SIZE=3737418240
export ROCSHMEM_MAX_NUM_CONTEXTS=48
export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128

export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100

export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1

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
export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1

export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9

export DEEPEP_ENABLE_LL_DISPATCH_OPT=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config

model_path=/module/DeepSeek-R1-Channel-INT8
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

option="--numa-node 3 1 1 0 7 5 5 4 "
option+=" --disable-cuda-graph "
option+=" --chunked-prefill-size -1"
export SGLANG_PP_LAYER_PARTITION="16,15,15,15"
option+=" --pp-max-micro-batch-size 1"
option+=" --disaggregation-ib-device mlx5_6,mlx5_7,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9"
option+=" --disaggregation-mode prefill --load-balance-method round_robin "

#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization slimquant_marlin --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30003 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${logpath}-$time.log


```

# D节点【ep32dp32-mtp2】：
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_TOPO_FILE_FORCE="/mnt/gbj/topo_400g.config"

export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_6,mlx5_7,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9
export ROCSHMEM_HEAP_SIZE=3737418240
export ROCSHMEM_MAX_NUM_CONTEXTS=48
export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128

export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100

export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16

export ALLREDUCE_STREAM_WITH_COMPUTE=1

export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1


export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_MLA_CAT=0  # set to 0 if nmz
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_USE_FUSED_RMS_QUANT=0  # disable in w8a8
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # disable in w8a8
export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1

export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
#export MC_TOPO_FILE_FORCE=/mnt/liucong/NMZ/mc_topo.config

export DEEPEP_ENABLE_LL_DISPATCH_OPT=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export ROCSHMEM_TOPO_FILE_FORCE=/mnt/lijian/Code/DeepEP/tests/topo.config
#export ROCSHMEM_IB_GID_INDEX=0

model_path=/module/DeepSeek-R1-Channel-INT8
model=${model_path##*/}
tp=32 #K100AI为8
pp=1
dp=32
ep=32
nodes=4
rank=0
host_ip=$(hostname -I | awk '{print $1}')
master_ip=10.16.1.42
max_model_len=140000
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="dserver/$model-tp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "

option+=" --chunked-prefill-size -1"
option+=" --page-size 64"
option+=" --ep-size $ep "
option+=" --dp-size $dp --moe-dense-tp-size 1 --enable-dp-lm-head --enable-dp-attention"
option+=" --max-running-requests 1024 "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 32 "
option+=" --enable-dp-attention --moe-a2a-backend deepep --deepep-mode low_latency"
option+=" --disaggregation-mode decode --prefill-round-robin-balance "
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization slimquant_marlin --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30004 --trust-remote-code  \
                                --dist-init-addr $master_ip:5005 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${logpath}-$time.log


```

# router：
```bash {wrap}
python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://10.16.1.36:30003 --decode http://10.16.1.42:30004 --policy round_robin --port 30002
```

## 关联资源

- [[飞书知识图谱/资源/大文件上传下载--cHqT4nFg|大文件上传下载]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-INT8|DeepSeek-R1-Channel-INT8]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）

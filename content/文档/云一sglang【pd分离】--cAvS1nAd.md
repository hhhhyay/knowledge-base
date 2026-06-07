---
title: 云一sglang【pd分离】
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-V3
  - GLM-5-FP8
  - GLM-5-W8A8
  - GLM5-Channelwise-FP8-quantized
  - GLM5-int8
  - glm5-w8a8
  - Kimi-K2.5
framework:
  - Mooncake
  - SGLang
  - vLLM
issue_type:
  - 精度问题
source: feishu
feishu_token: V2powqAKCizHAfklX4kcAvS1nAd
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/V2powqAKCizHAfklX4kcAvS1nAd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 云一sglang【pd分离】

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/V2powqAKCizHAfklX4kcAvS1nAd)

# glm5-w8a8

镜像：[[飞书知识图谱/文档/SGLang-GLM5-W8A8int8-镜像更新记录--cEGyenMf|SGLang-GLM5-int8-镜像更新]]

最新镜像：sglang_glm5_int8_0414_v2.tar

rayfile-c -a wuzh01.hpccube.com -P 65012 -u guobj -w 5306a83256Z3VvYmo=_ed1c657ea795a76a -no-meta -symbolic-links follow -retry 10 -retrytimeout 30 -o download -s '/images/sglang_glm5_int8_0414_v2.tar' -d <请输入下载目标全路径并替换尖括号及本内容> 

~~/public/home/sglang/sglang_glm5_int8_0413_latest.tar~~

与之前区别：替换了deeep、lightop

~~千卡集群：/public/home/sglang/sgl0510rc0_torch290_v4.tar.gz【千卡集群】~~

~~或者使用v3镜像需要安装几个whl包，千卡集群：/public/home/sglang/gbj/whl/glm_whl~~

~~curl -f -C - -o glm_whl.tar.gz ~~~~https://wuzh01.hpccube.com:65015/efile/s/d/Z3VvYmo=/53e5ae4200dc606d~~~~  ~~

~~乌镇本地：/public/home/guobj/fasttrans/sgl0510rc0_torch290_v4.tar.gz【同环境乌镇报错跑不通】~~

<image token="MygQbAWNboTGn1xNJeEcLQX4n3g" width="2186" height="1440" align="center"/>

## P节点【tp8-radix-cache】

p节点需要打开radix-cache
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

# export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
# export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
# export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1

# tvm_ffi
# export USE_ROCM=1
# export TLE_BACKEND=ROCM
# export TLE_USE_CUDA=OFF
# export TLE_USE_ROCM=ON
# export TVM_FFI_DISABLE_TORCH_DLPACK=1

export TVM_HOME=/workspace/tilelang/3rdparty/tvm
export TVM_LIBRARY_PATH=$TVM_HOME/build
export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH

model_path=/module2/GLM-5-W8A8
model=${model_path##*/}
tp=8 
pp=1
dp=1
ep=1
nodes=1
rank=0
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


option="--numa-node 3 1 1 0 7 5 5 4"
#option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"

option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
#option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
# option+=" --ep-size $ep "
#option+=" --cuda-graph-max-bs 512 "
#option+=" --page-size 64" --nsa-prefill-backend flashmla_sparse
#option+=" --context-length 6000"
option+=" --disable-cuda-graph  " 
option+=" --quantization slimquant_marlin  "
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

option+=" --disaggregation-mode prefill "



python3 -m sglang.launch_server --model-path $model_path  ${option} \
                                --trust-remote-code \
                             --kv-cache-dtype fp8_e4m3  --dtype bfloat16 --mem-fraction-static $gpu_mem \
                                --host $host_ip --port 30000 --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --tp-size $tp --dp-size $dp 2>&1 | tee  ${logpath}-test--tensor$hostname-$time.log
                        

```

## P节点【tp8cp8】
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

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
#export TVM_HOME=/workspace/tilelang/3rdparty/tvm
#export TVM_LIBRARY_PATH=$TVM_HOME/build
#export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH

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
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size -1 "
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
option+=" --quantization slimquant_marlin "
option+=" --attn-cp-size ${attn_cp_size} "
option+=" --enable-nsa-prefill-context-parallel  --nsa-prefill-cp-mode round-robin-split "
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

## D节点【tp8-mtp3】：
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

# export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
# export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
# export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1

# tvm_ffi
# export USE_ROCM=1
# export TLE_BACKEND=ROCM
# export TLE_USE_CUDA=OFF
# export TLE_USE_ROCM=ON
# export TVM_FFI_DISABLE_TORCH_DLPACK=1

export TVM_HOME=/workspace/tilelang/3rdparty/tvm
export TVM_LIBRARY_PATH=$TVM_HOME/build
export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH
#export PYTHONPATH=/workspace/tilelang:$PYTHONPATH

# export CUDA_HOME=/opt/dtk
# export CPLUS_INCLUDE_PATH=/opt/dtk/include:$CPLUS_INCLUDE_PATH

#/models/GLM-5-FP8 /model/GLM5-Channelwise-FP8-quantized /model/ZhipuAI/GLM-5-FP8 #
#model_path=/module2/GLM-5-W8A8
model_path=/module2/GLM-5-W8A8
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

option="--numa-node 3 1 1 0 7 5 5 4"
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size -1"
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
#option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
#option+=" --pp-max-micro-batch-size 2"
option+=" --cuda-graph-max-bs 128 "
#option+=" --context-length 6000"
#option+=" --disable-cuda-graph  " #--skip-server-warmup
option+=" --quantization slimquant_marlin  "
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode decode "

python3 -m sglang.launch_server --model-path $model_path  ${option} \
                                --trust-remote-code \
                             --kv-cache-dtype fp8_e4m3  --dtype bfloat16 --mem-fraction-static $gpu_mem \
                                --host $host_ip --port 30000 --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --tp-size $tp --dp-size $dp 2>&1 | tee  ${logpath}-test--tensor$hostname-$time.log
```

## D节点【ep16dp16】：
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export ROCSHMEM_MAX_NUM_CONTEXTS=48
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export ROCSHMEM_TOPO_FILE_FORCE="/home/topo_400g.config"


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

#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072


export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16


export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0

# export SGLANG_USE_FP8_W8A8_MOE=1
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export MC_TOPO_FILE_FORCE=/mnt/gbj/mc_topo_400g.config
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1

export TVM_HOME=/workspace/tilelang/3rdparty/tvm
export TVM_LIBRARY_PATH=$TVM_HOME/build
export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH


export W8A8_SUPPORT_METHODS=3

model_path=/module/GLM-5-W8A8
model=${model_path##*/}
tp=16 #K100AI为8
pp=1
dp=16
ep=16
attn_cp_size=1
nodes=2
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
#master_ip=
#max_model_len=6000
gpu_mem=0.85
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tpp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi


option="--numa-node 3 1 1 0 7 5 5 4"
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size -1 "
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 3 --speculative-eagle-topk 1 --speculative-num-draft-tokens 4 "
option+=" --quantization slimquant_marlin "
option+=" --moe-a2a-backend deepep "
option+=" --enable-dp-attention --moe-dense-tp-size 1  --enable-dp-lm-head"
option+=" --attn-cp-size ${attn_cp_size} "
option+=" --cuda-graph-max-bs 32 "
# option+=" --disable-cuda-graph "
option+=" --moe-dense-tp-size 1 "
option+=" --deepep-mode low_latency"
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
option+=" --disaggregation-mode decode "

python3 -m sglang.launch_server --model-path $model_path  ${option} \
                                --trust-remote-code \
                                --kv-cache-dtype fp8_e4m3  --dtype bfloat16 --mem-fraction-static $gpu_mem \
                                --host $host_ip --port 30000 --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --tp-size $tp --dp-size $dp 2>&1 | tee  ${logpath}-test--tensor$hostname-$time.log


```

简单精度验证正常：

<image token="W7SQbuCiZoNHhIxnIAoc2Jtknsd" width="2395" height="402" align="center"/>

## router：
```bash {wrap}
python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://10.16.1.36:30000 --decode http://10.16.1.46:30000 --policy round_robin --port 30010
```

## Decode

decode_throughput_generate.py
```python {wrap}
import asyncio
import aiohttp
import json
import time
import random
import argparse
from datetime import datetime
import numpy as np

TIMESTAMP_FORMAT = "%Y-%m-%d %H:%M:%S"

time_stamp = datetime.now().strftime(TIMESTAMP_FORMAT)

tokenizer = None

global_prefix_ids: list[int] | None = None


def create_prompts(input_len: int, num_req: int=1, prefix_len: int=0, prefix_ids: list[int] | None = None) -> list[list[int]]:
    if prefix_len > 0:
        assert input_len >= prefix_len
        if prefix_ids is None:
            prefix_ids = [random.randint(42, 10000) for _ in range(prefix_len)]
        else:
            assert len(prefix_ids) == prefix_len
    else:
        prefix_ids = []

    return [prefix_ids + [random.randint(42, 10000) for _ in range(input_len-prefix_len)] for _ in range(num_req)]


def detokenize_to_str(token_id_list: list[int]) -> str:
    from transformers import AutoTokenizer
    global tokenizer
    if tokenizer == None:
        assert args.tokenizer_path != ''
        tokenizer = AutoTokenizer.from_pretrained(args.tokenizer_path)
    return tokenizer.decode(token_id_list)


def generate_data(base_url: str, prompt: str | list[int], output_len: int=10):
    prompt_key = "input_ids"
    if args.str_input:
        prompt = detokenize_to_str(prompt)
        prompt_key = "text"
    data_template = {
        "stream": True,
        prompt_key: prompt,
        "sampling_params": {
            "max_new_tokens": output_len,
            "temperature": 0.6,
            "top_p": 0.95,
            "ignore_eos": True,
        },
    }
    url = f"{base_url}/generate"
    return data_template, url


async def iter_sse_events(response: aiohttp.ClientResponse):
    buffer = ""
    async for chunk in response.content:
        buffer += chunk.decode("utf-8")
        while "\n\n" in buffer:
            raw_event, buffer = buffer.split("\n\n", 1)
            data_lines = []
            for line in raw_event.splitlines():
                if line.startswith("data:"):
                    data_lines.append(line[5:].lstrip())
            if data_lines:
                yield "\n".join(data_lines)

    trailing = buffer.strip()
    if trailing:
        data_lines = []
        for line in trailing.splitlines():
            if line.startswith("data:"):
                data_lines.append(line[5:].lstrip())
        if data_lines:
            yield "\n".join(data_lines)


async def single_task(prompt_ids: list[int], output_len: int=10, sleep_time: float=0, if_print: bool=False):

    if sleep_time > 0:
        await asyncio.sleep(sleep_time)

    data_template, url = generate_data(args.base_url, prompt=prompt_ids, output_len=output_len)

    t_start = time.time()
    connector = aiohttp.TCPConnector(limit=0)  # 连接数
    first_token = True
    first_token_time = t_start
    last_token_time = t_start
    ttft = 0
    response_interval = []
    completion_tokens = 0
    async with aiohttp.ClientSession(connector=connector) as session:
        async with session.post(url, json=data_template) as response:
            if response.status != 200:
                error_body = await response.text()
                raise RuntimeError(f"HTTP {response.status}: {error_body}")

            async for event in iter_sse_events(response):
                if event == "[DONE]":
                    last_token_time = time.time()
                    continue

                chunk_dict = json.loads(event)
                if "error" in chunk_dict:
                    raise RuntimeError(json.dumps(chunk_dict["error"], ensure_ascii=False))

                meta_info = chunk_dict.get("meta_info") or {}
                completion_tokens = meta_info.get("completion_tokens", completion_tokens)

                has_output = bool(chunk_dict.get("text")) or bool(chunk_dict.get("output_ids"))
                if not has_output:
                    continue

                if first_token:
                    first_token_time = time.time()
                    first_token = False
                    last_token_time = first_token_time
                else:
                    current_time = time.time()
                    response_interval_time = round((current_time - last_token_time)*1000, 2)
                    response_interval.append(response_interval_time)
                    last_token_time = current_time

    ttft = first_token_time - t_start
    tpot = np.median(np.array(response_interval)).item()/1000 if response_interval else 0.0
    if completion_tokens == 0 and not first_token:
        completion_tokens = len(response_interval) + 1

    if if_print:
        print(f"ttft: {ttft*1000:.2f} ms, step time: {tpot * 1000:.2f} ms, "
              f"output_len: {completion_tokens}, "
              f"len_response_interval = {len(response_interval)}, "
              f"response_interval = {response_interval}",
              flush=True)
    return ttft, tpot

async def decode_benchmark():

    bs_list = []
    max_bs_step = args.max_batch_size_step
    cur_bs = args.min_batch_size
    while cur_bs <= args.max_batch_size:
        bs_list.append(cur_bs)
        if cur_bs < max_bs_step:
            cur_bs *= 2
        else:
            cur_bs += max_bs_step

    max_kv_cache_len = args.max_total_kvcache_length_k*1024
    min_input_len = int(args.min_input_length_k*1024)
    max_input_len = args.max_input_length_k*1024
    max_step_len = args.max_input_length_step_k*1024

    num_repeats = args.num_repeats
    dp_size = args.dp_size

    assert args.other_request_output_length > args.target_request_output_length
    reserve_output_length = args.other_request_output_length + 20

    global global_prefix_ids
    global_prefix_ids = create_prompts(max_input_len)[0]

    save_path = f"{args.save_path}/{time_stamp}_decode_tpot_results.json"

    # bench
    tpot_var_bs_list = []
    for i, bs in enumerate(bs_list):
        print(f"\n--- Local BS={bs} ---")
        max_local_input_len = min(
            max_kv_cache_len // bs,
            max_input_len
        ) - reserve_output_length
        assert max_local_input_len > 0
        prompt_len_list = []
        prompt_len = min_input_len
        while prompt_len < max_local_input_len:
            prompt_len_list.append(prompt_len)
            if prompt_len <= max_step_len:
                prompt_len *= 2
            else:
                prompt_len += max_step_len
        prompt_len_list.append(max_local_input_len)

        tpot_var_len_list = []
        for prompt_len in prompt_len_list:
            tmpt_tpot_list = []
            print(f"\nPrompt length: {prompt_len}")
            for _ in range(num_repeats):
                prompt_ids_list = create_prompts(
                    prompt_len,
                    num_req=bs*dp_size,
                    prefix_len=prompt_len,
                    prefix_ids=global_prefix_ids[:prompt_len]
                )

                tasks = []
                for i, prompt_ids in enumerate(prompt_ids_list):
                    if i == bs*dp_size-1:
                        tasks.append(
                            asyncio.create_task(
                                single_task(
                                    prompt_ids,
                                    args.target_request_output_length,
                                    args.target_request_wait_time,
                                    if_print=True
                                )
                            )
                        )
                    else:
                        tasks.append(
                            asyncio.create_task(
                                single_task(
                                    prompt_ids,
                                    args.other_request_output_length,
                                )
                            )
                        )
                results = await asyncio.gather(*tasks)
                tpot = results[-1][1]
                tmpt_tpot_list.append(tpot)
            avg_tpot = sum(tmpt_tpot_list) / len(tmpt_tpot_list)
            tpot_var_len_list.append(avg_tpot)

        tpot_var_bs_list.append(
            {
                "bs": bs,
                "prompt_len_list": prompt_len_list,
                "tpot_var_len_list": tpot_var_len_list
            }
        )

        result_dict = {
            "bs_list": bs_list[:len(tpot_var_bs_list)],
            "tpot_var_bs_list": tpot_var_bs_list
        }

        with open(save_path, "w") as f:
            json.dump(result_dict, f, indent=4)
        print(f"save result dict to {save_path}")

        if args.plot:
            plot_results(result_dict)


def plot_results(result_dict: dict, save_path: str | None = None):
    import matplotlib.pyplot as plt
    import numpy as np

    bs_list = result_dict['bs_list']
    tpot_var_bs_list = result_dict['tpot_var_bs_list']

    plt.figure(figsize=(10, 5))
    for i, bs in enumerate(bs_list):
        prompt_len_list = tpot_var_bs_list[i]["prompt_len_list"]
        tpot_var_len_list = np.array(tpot_var_bs_list[i]["tpot_var_len_list"]) * 1000
        plt.plot(prompt_len_list, tpot_var_len_list, marker='o', label=f'bs={bs}')


    if args.x_log_scale:
        plt.xscale('log')
    plt.ylim(bottom=0)
    plt.xlabel('Context Length')
    plt.ylabel('Step Time (ms)')
    plt.title('Decode TPOT Benchmark')
    plt.legend(bbox_to_anchor=(0.5, 1.05), loc='lower center', ncol=len(bs_list))

    plt.grid(True)
    if save_path == None:
        save_path = f"{args.save_path}/{time_stamp}_decode_tpot.png"
    print(f"save result figure to {save_path}")
    plt.savefig(save_path)


def replot():
    assert args.result_path != ''
    with open(args.result_path, "r") as f:
        result_dict = json.load(f)

    suffix = '_decode_tpot_results.json'
    if args.result_path.endswith(suffix):
        prefix = args.result_path.rstrip(suffix)
    else:
        prefix = f"{args.save_path}/{time_stamp}"
    new_save_path = f"{prefix}_decode_tpot.png"
    plot_results(result_dict=result_dict, save_path=new_save_path)


if __name__ == '__main__':
    argparser = argparse.ArgumentParser(description='Decode Step Time Benchmark')

    argparser.add_argument(
        '--base-url',
        type=str,
        default='http://127.0.0.1:8080',
    )

    argparser.add_argument(
        '--model',
        type=str,
        default='DeepSeek-V3',
    )

    argparser.add_argument(
        '--min-input-length-k',
        type=float,
        default=1,
        help='min input length test, '
        '--min-input-length-k 1 means 1024.'
    )

    argparser.add_argument(
        '--max-input-length-k',
        type=int,
        default=0,
        help='max input length test, '
        '--max-input-length-k 128 means 128k.'
    )

    argparser.add_argument(
        '--max-input-length-step-k',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--max-total-kvcache-length-k',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--min-batch-size',
        type=int,
        default=1,
    )

    argparser.add_argument(
        '--max-batch-size',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--max-batch-size-step',
        type=int,
        default=16,
    )

    argparser.add_argument(
        '--other-request-output-length',
        type=int,
        default=256,
    )

    argparser.add_argument(
        '--target-request-output-length',
        type=int,
        default=64,
    )

    argparser.add_argument(
        '--target-request-wait-time',
        type=float,
        default=0.1,
    )

    argparser.add_argument(
        '--num-repeats',
        type=int,
        default=3,
    )

    argparser.add_argument(
        '--dp-size',
        type=int,
        default=1,
    )

    argparser.add_argument(
        '--str-input',
        action=argparse.BooleanOptionalAction,
        default=False,
        help='Default we use token_ids_list as prompt. '
        'If inference engine does not support token_ids_list as input, '
        'we can set --str-input and we will detokenizer token_ids_list to str.'
    )

    argparser.add_argument(
        '--tokenizer-path',
        type=str,
        default='',
    )

    argparser.add_argument(
        '--plot',
        action=argparse.BooleanOptionalAction,
        default=True
    )

    argparser.add_argument(
        '--x-log-scale',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--save_path',
        type=str,
        default="./",
    )

    argparser.add_argument(
        '--replot',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--result-path',
        type=str,
        default="",
    )

    args = argparser.parse_args()

    if args.replot:
        replot()
    else:
        asyncio.run(decode_benchmark())


# python3 decode_throughput.py --base-url http://127.0.0.1:8080 --model DeepSeek-V3 --max-input-length-k 128 --max-total-kvcache-length-k 300 --max-batch-size 32

```


```bash {wrap}
python decode_throughput_generate.py     --base-url http://10.16.1.36:30010     --model /module/GLM-5-W8A8     --max-input-length-k 50    --max-total-kvcache-length-k 500     --max-batch-size 32
```

## prefill

prefill_throughput_generate.py
```python {wrap}
import asyncio
import aiohttp
import json
import time
import random
import argparse
from datetime import datetime
import tqdm
import math
from tqdm.asyncio import tqdm_asyncio

TIMESTAMP_FORMAT = "%Y-%m-%d-%H-%M-%S"

time_stamp = datetime.now().strftime(TIMESTAMP_FORMAT)

tokenizer = None

global_prefix_ids: list[int] | None = None

def create_prompts(input_len: int, num_req: int=1, prefix_len: int=0, prefix_ids: list[int] | None = None) -> list[list[int]]:
    if prefix_len > 0:
        assert input_len >= prefix_len
        if prefix_ids is None:
            prefix_ids = [random.randint(42, 10000) for _ in range(prefix_len)]
        else:
            assert len(prefix_ids) == prefix_len
    else:
        prefix_ids = []

    return [prefix_ids + [random.randint(42, 10000) for _ in range(input_len-prefix_len)] for _ in range(num_req)]


def detokenize_to_str(token_id_list: list[int]) -> str:
    from transformers import AutoTokenizer
    global tokenizer
    if tokenizer == None:
        assert args.tokenizer_path != ''
        tokenizer = AutoTokenizer.from_pretrained(args.tokenizer_path)
    return tokenizer.decode(token_id_list)


def completion_data(base_url: str, prompt: list[int], output_len: int=10):
    """
    修改为 /generate 接口格式 (类似SGLang)
    """
    prompt_key = "input_ids"
    if args.str_input:
        prompt = detokenize_to_str(prompt)
        prompt_key = "text"
    
    data_template = {
        "stream": True,
        prompt_key: prompt,
        "sampling_params": {
            "max_new_tokens": output_len,
            "temperature": 0.6,
            "top_p": 0.95,
            "ignore_eos": True,
        },
    }
    url = f"{base_url}/generate"
    return data_template, url


async def iter_sse_events(response: aiohttp.ClientResponse):
    """
    从文件2复制：解析SSE事件流
    """
    buffer = ""
    async for chunk in response.content:
        buffer += chunk.decode("utf-8")
        while "\n\n" in buffer:
            raw_event, buffer = buffer.split("\n\n", 1)
            data_lines = []
            for line in raw_event.splitlines():
                if line.startswith("data:"):
                    data_lines.append(line[5:].lstrip())
            if data_lines:
                yield "\n".join(data_lines)

    trailing = buffer.strip()
    if trailing:
        data_lines = []
        for line in trailing.splitlines():
            if line.startswith("data:"):
                data_lines.append(line[5:].lstrip())
        if data_lines:
            yield "\n".join(data_lines)


async def single_task(prompt_ids: list[int], output_len: int = 10, if_print: bool = True):

    data_template, url = completion_data(args.base_url, prompt=prompt_ids, output_len=output_len)

    t_start = time.time()    
    connector = aiohttp.TCPConnector(limit=0)  # 连接数
    first_token = True
    last_time = t_start
    ttft = 0
    tpot = []
    
    async with aiohttp.ClientSession(connector=connector) as session:
        async with session.post(url, json=data_template) as response:
            # 从文件2添加：检查HTTP状态码
            if response.status != 200:
                error_body = await response.text()
                raise RuntimeError(f"HTTP {response.status}: {error_body}")

            # 从文件2添加：使用SSE事件流解析
            async for event in iter_sse_events(response):
                if event == "[DONE]":
                    continue

                chunk_dict = json.loads(event)
                # 从文件2添加：检查error字段
                if "error" in chunk_dict:
                    raise RuntimeError(json.dumps(chunk_dict["error"], ensure_ascii=False))

                # 从文件2添加：检查是否有输出
                has_output = bool(chunk_dict.get("text")) or bool(chunk_dict.get("output_ids"))
                if not has_output:
                    continue

                if first_token:
                    last_time = time.time()
                    ttft = last_time - t_start
                    first_token = False
                else:
                    current_time = time.time()
                    tpot.append(current_time - last_time)
                    last_time = current_time
    
    if if_print:
        print(f"prompt_len: {len(prompt_ids)}, ttft: {ttft*1000:.6f} ms", flush=True)
    
    return ttft, tpot


async def prefill_benchmark():

    # bench
    prompt_len_list = []
    assert args.max_input_length_k != 0
    min_input_len = int(args.min_input_length_k*1024)
    max_input_len = args.max_input_length_k*1024
    max_step_len = args.max_input_length_step_k*1024

    prompt_len = min_input_len
    while prompt_len < max_input_len-100:
        prompt_len_list.append(prompt_len)
        if prompt_len <= max_step_len:
            prompt_len *= 2
        else:
            prompt_len += max_step_len
    prompt_len_list.append(max_input_len-100)

    global global_prefix_ids
    global_prefix_ids = create_prompts(max_input_len)[0]

    if args.prefix_cache:
        prefix_ratio_list = [0, 0.2, 0.4, 0.6, 0.8]
        # warmup prefix
        prompt_len = max_input_len - 100
        print(f"warmup prefix start ...", flush=True)
        tasks = [
            asyncio.create_task(
                single_task(
                    global_prefix_ids[:prompt_len], 
                    1,
                    if_print=False,
                )
            ) for _ in range(args.dp_size*2)
        ]
        results = await asyncio.gather(*tasks)
        print(f"warmup prefix finished", flush=True)
    else:
        prefix_ratio_list = [0]

    chunk_size = args.chunk_size_k * 1024

    save_path = f"{args.save_path}/{time_stamp}_prefill_throughput_results.json"
    result_dict = {
        "single_req": {},
        "concurrency": {},
    }
    

    # test single concurrency (only for dp_size == 1)
    for prefix_ratio in prefix_ratio_list:
        print(f"\nPrefix Ratio: {prefix_ratio:.2f}", flush=True)
        if args.dp_size == 1:
            throughput_list = []
            for prompt_len in prompt_len_list:
                throughput = await eval_normal(prompt_len=prompt_len, prefix_ratio=prefix_ratio)
                throughput_list.append(throughput)

            result_dict["single_req"][prefix_ratio] = {
                "prompt_len_list": prompt_len_list, 
                "throughput_list": throughput_list
            }

            with open(save_path, "w") as f:
                json.dump(result_dict, f, indent=4)
            print(f"save result dict to {save_path}")

            if args.plot:
                plot_results(result_dict)

        if args.concurrency:
            # test concurrency (all dp_size)
            finished_prompt_len_list = []
            if len(prompt_len_list) > 0:
                throughput_list = []
                for prompt_len in prompt_len_list:
                    concurrency = math.ceil(chunk_size / (prompt_len*(1-prefix_ratio)))
                    throughput = await eval_concurrency(
                        prompt_len=prompt_len,
                        concurrency=concurrency,
                        workload=5 if concurrency > 1 else 2,
                        prefix_ratio=prefix_ratio,
                    )
                    throughput_list.append(throughput)
                    finished_prompt_len_list.append(prompt_len)
                
                    result_dict['concurrency'][prefix_ratio] = {
                        "prompt_len_list": finished_prompt_len_list, 
                        "throughput_list": throughput_list
                    }

                    with open(save_path, "w") as f:
                        json.dump(result_dict, f, indent=4)
                    print(f"save result dict to {save_path}")

                    if args.plot:
                        plot_results(result_dict)


async def eval_normal(prompt_len: int, prefix_ratio: float) -> float:
    print(f"prompt_len: {prompt_len} start eval single request throughput")
    prefix_len = int(prompt_len * prefix_ratio)
    num_repeats = args.num_repeats
    tmp_throughput_list = []

    prefix_prompt = global_prefix_ids[:prefix_len]

    prompt_ids_list = create_prompts(
        prompt_len, 
        num_req=num_repeats+1, 
        prefix_len=prefix_len,
        prefix_ids=prefix_prompt,
    )
    #warmup
    ttft, _ = await single_task(prompt_ids_list[-1], if_print=False)
    for prompt_ids in prompt_ids_list[:-1]:
        ttft, _ = await single_task(prompt_ids)
        throughput = (prompt_len-prefix_len)/ttft
        tmp_throughput_list.append(throughput)
    avg_throughput = sum(tmp_throughput_list) / len(tmp_throughput_list)
    return avg_throughput


async def eval_concurrency(
    prompt_len: int, 
    concurrency: int, 
    workload: int=5,
    prefix_ratio: float=0,
) -> float:
    print(f"prompt_len: {prompt_len} start eval concurrency throughput")
    dp_size = args.dp_size
    num_reqs = concurrency * workload * dp_size

    prefix_len = int(prompt_len * prefix_ratio)
    prefix_prompt = global_prefix_ids[:prefix_len]

    prompt_ids_list = create_prompts(
        prompt_len, 
        num_req=num_reqs,
        prefix_len=prefix_len,
        prefix_ids=prefix_prompt,
    )

    t_start = time.time()
    tasks = [
        asyncio.create_task(
            single_task(prompt_ids, output_len=1, if_print=False)
        ) for prompt_ids in prompt_ids_list
    ]
    results = await tqdm_asyncio.gather(*tasks, desc=f"eval concurrency throughput, prompt_len: {prompt_len}")
    t_end = time.time()
    t_duration = t_end - t_start

    throughput = (prompt_len-prefix_len) * num_reqs / t_duration
    
    print(f"prompt_len: {prompt_len}, throughput: {throughput:.2f} tokens / s", flush = True)
    return throughput


def plot_results(result_dict: dict, save_path: str | None = None):
    import matplotlib.pyplot as plt
    import numpy as np

    plt.figure(figsize=(10, 5))
    for prefix_ratio, results in result_dict["single_req"].items():
        prompt_len_list = results["prompt_len_list"]
        throughput_list = results["throughput_list"]
        compute_len_list = np.array(prompt_len_list) * (1-float(prefix_ratio))

        plt.plot(compute_len_list, throughput_list, marker='o', label=f"Single Request, Prefix={prefix_ratio}")

    for prefix_ratio, results in result_dict["concurrency"].items():
        prompt_len_list = results["prompt_len_list"]
        throughput_list = results["throughput_list"]
        compute_len_list = np.array(prompt_len_list) * (1-float(prefix_ratio))
        
        plt.plot(compute_len_list, throughput_list, marker='o', label=f"Concurrency, Prefix={prefix_ratio}")

    plt.ylim(bottom=0)
    if args.x_log_scale:
        plt.xscale('log')
    plt.xlabel('New Compute Length')
    plt.ylabel('Throughput (tokens/s)')
    plt.title('Prefill Throughput Benchmark')
    plt.legend(bbox_to_anchor=(0.5, 1.05), loc='lower center', ncol=4)

    plt.tight_layout(pad=0.1)
    plt.grid(True)
    if save_path == None:
        save_path = f"{args.save_path}/{time_stamp}_prefill_throughput.png"
    print(f"save result figure to {save_path}")
    plt.savefig(save_path)


def replot():
    assert args.result_path != ''
    with open(args.result_path, "r") as f:
        result_dict = json.load(f)

    suffix = '_prefill_throughput_results.json'
    if args.result_path.endswith(suffix):
        prefix = args.result_path.rstrip(suffix)
    else:
        prefix = f"{args.save_path}/{time_stamp}"
    new_save_path = f"{prefix}_prefill_throughput.png"
    plot_results(result_dict=result_dict, save_path=new_save_path)
    


if __name__ == '__main__':
    argparser = argparse.ArgumentParser(description='Prefill Throughput Benchmark')

    argparser.add_argument(
        '--base-url',
        type=str,
        default='http://127.0.0.1:8080',
    )

    argparser.add_argument(
        '--model',
        type=str,
        default='DeepSeek-V3',
    )

    argparser.add_argument(
        '--min-input-length-k',
        type=float,
        default=1,
        help='min input length test, '
        '--min-input-length-k 1 means 1024.'
    )

    argparser.add_argument(
        '--max-input-length-k',
        type=int,
        default=0,
        help='max input length test, '
        '--max-input-length-k 128 means 128k.'
    )

    argparser.add_argument(
        '--max-input-length-step-k',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--chunk-size-k',
        type=int,
        default=8,
    )

    argparser.add_argument(
        '--num-repeats',
        type=int,
        default=3,
    )

    argparser.add_argument(
        '--prefix-cache',
        action=argparse.BooleanOptionalAction,
        default=True,
    )

    argparser.add_argument(
        '--concurrency',
        action=argparse.BooleanOptionalAction,
        default=True,
    )

    argparser.add_argument(
        '--dp-size',
        type=int,
        default=1,
    )

    argparser.add_argument(
        '--str-input',
        action=argparse.BooleanOptionalAction,
        default=False,
        help='Default we use token_ids_list as prompt. '
        'If inference engine does not support token_ids_list as input, '
        'we can set --str-input and we will detokenizer token_ids_list to str.'
    )

    argparser.add_argument(
        '--tokenizer-path',
        type=str,
        default='',
    )

    argparser.add_argument(
        '--plot',
        action=argparse.BooleanOptionalAction,
        default=True,
    )

    argparser.add_argument(
        '--x-log-scale',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--save-path',
        type=str,
        default="./",
    )

    argparser.add_argument(
        '--replot',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--result-path',
        type=str,
        default="",
    )

    args = argparser.parse_args()

    if args.replot:
        replot()
    else:
        asyncio.run(prefill_benchmark())


# python3 prefill_throughput.py --base-url http://127.0.0.1:8080 --model DeepSeek-V3 --max-input-length-k 128
```


```bash {wrap}
python prefill_throughput_generate.py \
    --base-url http://10.16.1.55:30010 \
    --model /module/GLM-5-W8A8  \
    --max-input-length-k 50
```

## 测试结果：

D节点会出现类似的log，36节点跑挂了

<image token="RkSvbWBz5orrUlxIegHcy9FPnLg" width="2388" height="1146" align="center"/>

接收率不太稳定

<image token="CxQnbmFvEoysy6xi8QJcjJeqncc" width="2374" height="1450" align="center"/>

<image token="WFAdbrWT7ogF4Jxl0iNcawjZnvc" width="1000" height="500" align="center"/>

Prefill报错：超出32k会vmfault

<image token="Mzp7bDZ2uoSuENxOg2HcM5fqnbb" width="2392" height="1476" align="center"/>

<image token="ORlsbI6alo4YYIx16LIc1VPwn3c" width="1000" height="500" align="center"/>

# kimi2.5

镜像：

千卡集群：/public/das/sglang/Images/sgl0510rc0_torch290_v3.tar.gz【千卡集群】

乌镇本地：/public/home/guobj/fasttrans/sgl0510rc0_torch290_v3.tar.gz

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

模型文件夹需要添加下tokenizer.json，不然PD分离起router时会报错无法识别到tokenizer.json

【千卡集群路径，node036和46节点已存在可直接跑】：/public/home/sglang/gbj/tokenizer.json

乌镇本地没有，可以通过下面脚本直接生成
</callout>

```python {wrap}
from transformers import AutoTokenizer
import json

tok = AutoTokenizer.from_pretrained("/module/Kimi-K2.5", trust_remote_code=True)

# 获取词汇表（如果可访问）
try:
    vocab = tok.get_vocab()  # 或 tok.encoder
    
    # 构建 tokenizer.json 结构
    tokenizer_json = {
        "version": "1.0",
        "truncation": None,
        "padding": None,
        "added_tokens": [],
        "normalizer": None,
        "pre_tokenizer": {
            "type": "ByteLevel",
            "add_prefix_space": False,
            "trim_offsets": True,
            "use_regex": True
        },
        "post_processor": None,
        "decoder": {
            "type": "ByteLevel",
            "add_prefix_space": False,
            "trim_offsets": True
        },
        "model": {
            "type": "BPE",
            "vocab": vocab,
            "merges": [],  # tiktoken 通常不暴露 merges
            "dropout": None,
            "unk_token": None,
            "continuing_subword_prefix": "",
            "end_of_word_suffix": "",
            "fuse_unk": False,
            "byte_fallback": False
        }
    }
    
    with open("./tokenizer.json", "w") as f:
        json.dump(tokenizer_json, f, indent=2)
        
except AttributeError:
    print("无法直接获取词汇表，Kimi tokenizer 可能封装了 tiktoken")
```

## P节点【pp4tp2-radix-cache】

p节点需要打开radix-cache不然在decode测试时prefill性能低router会failed
```bash {wrap}
export SGLANG_USE_LIGHTOP=1
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1
export SGLANG_USE_OPT_CAT=1
# export SGLANG_USE_FUSED_RMS_QUANT=1
# export SGLANG_USE_RMS_QUANT_PATH=1
# export USE_FUSED_RMS_QUANT=1 # lmslim需要 vllm
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=1

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export HIP_GRAPH_ACCUMULATE_DISPATCH=0
export SGLANG_USE_LIGHTOP=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/profiler_0324

export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
# export NCCL_MIN_NCHANNELS=16
export SGLANG_USE_MARLIN_W4A16_MOE=1

export ALLREDUCE_STREAM_WITH_COMPUTE=1

model_path=/module/Kimi-K2.5
model=${model_path##*/}
tp=2
pp=4
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
# host_ip=0.0.0.0
port=30000
hostname=$(hostname)
master_ip=$host_ip
#max_model_len=65536
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
#option+=" --disable-radix-cache "
#option+=" --tokenizer-path $model_path"
#option+=" --reasoning-parser kimi_k2 --tool-call-parser kimi_k2 "

#option+=" --chunked-prefill-size 16384"
#option+=" --max-running-requests 512 "
#option+=" --context-length $max_model_len"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
option+=" --pp-max-micro-batch-size 1"

#option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 2  --cuda-graph-max-bs 512"
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode prefill "



#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port $port --trust-remote-code  --page-size 64 \
                                --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log


```

## D节点【tp8】：

**node1**
```bash {wrap}
export SGLANG_USE_LIGHTOP=1
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1
export SGLANG_USE_OPT_CAT=1
# export SGLANG_USE_FUSED_RMS_QUANT=1
# export SGLANG_USE_RMS_QUANT_PATH=1
# export USE_FUSED_RMS_QUANT=1 # lmslim需要 vllm
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=1

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export HIP_GRAPH_ACCUMULATE_DISPATCH=0
export SGLANG_USE_LIGHTOP=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/profiler_0324

export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
# export NCCL_MIN_NCHANNELS=16
export SGLANG_USE_MARLIN_W4A16_MOE=1

export ALLREDUCE_STREAM_WITH_COMPUTE=1

model_path=/module/Kimi-K2.5
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
# host_ip=0.0.0.0
port=30000
hostname=$(hostname)
master_ip=$host_ip
#max_model_len=65536
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size 16384"
#option+=" --tokenizer-path $model_path"
#option+=" --reasoning-parser kimi_k2 --tool-call-parser kimi_k2 "

#option+=" --max-running-requests 512 "
#option+=" --context-length $max_model_len"

#option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 2  --cuda-graph-max-bs 512"
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode decode "



#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port $port --trust-remote-code  --page-size 64 \
                                --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```

## router：
```bash {wrap}
python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://10.16.1.36:30000 --decode http://10.16.1.46:30000 --policy round_robin --port 30010
```

**初步测试性能并不好**

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

**此脚本为generate接口，腾讯给的v1接口会报错，可以联系中华获取，generate接口性能暂时不好，大致是kimi暂时还没怎么优化吧，会有图片的warning的log，怀疑没有tunning json文件**
</callout>

<image token="Gsl1bt0PGo70UTxJdI0c1eY2nad" width="1891" height="563" align="center"/>

## decode测试：
```bash {wrap}
python decode_throughput_generate.py     --base-url http://10.16.1.46:30010     --model /module/Kimi-K2.5     --max-input-length-k 55    --max-total-kvcache-length-k 500     --max-batch-size 32
```

## prefill：
```bash {wrap}
python prefill_throughput_generate.py \
    --base-url http://10.16.1.46:30010 \
    --model /model/Kimi-K2.5 \
    --max-input-length-k 55
```

## 测试结果：

<image token="B3JfbODrpo08ITxiAPrci2mmnUd" width="792" height="434" align="center"/>

## 关联资源

- [[飞书知识图谱/文档/SGLang-GLM5-W8A8int8-镜像更新记录--cEGyenMf|SGLang-GLM5-W8A8int8-镜像更新记录]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/GnrpwZEeoiHNTRkpwjycEGyenMf)

## 关联知识

- [[飞书知识图谱/实体/精度问题|精度问题]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-V3|DeepSeek-V3]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-FP8|GLM-5-FP8]]（model）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/GLM5-Channelwise-FP8-quantized|GLM5-Channelwise-FP8-quantized]]（model）
- [[飞书知识图谱/实体/GLM5-int8|GLM5-int8]]（model）
- [[飞书知识图谱/实体/glm5-w8a8|glm5-w8a8]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/Kimi-K2.5|Kimi-K2.5]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

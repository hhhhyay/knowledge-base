---
title: Sglang测试指导（QTeam）
tags:
  - AI
  - LLM
  - Benchmark
model:
  - deepseek-r1
  - DeepSeek-R1-0528-W4A8-V2
  - DeepSeek-R1-bf16
  - DeepSeek-R1-Channel-INT8
  - DeepSeek-R1-Distill-Llama-70B
  - DeepSeek-R1-Distill-Llama-70B-main
  - DeepSeek-R1-Distill-Llama-70B-quantized.w8a8
  - DeepSeek-R1-Distill-Llama-8B
  - DeepSeek-R1-Distill-Qwen-32B
  - GLM-5-W8A8
  - Glm5-W8a8
  - Kimi-K2.5
  - minimax-append-think
  - minimax-m2
  - MiniMax-M2.5-W8A8
  - Qwen2.5-VL-72B-Instruct
  - Qwen3-30B-A3B
  - Qwen3-32B
  - Qwen3-32B.w8a8
  - Qwen3-8B
  - Qwen3-8B.w8a8
  - Qwen3-VL-235B-A22B-Instruct
  - Qwen3.5-397B-A17B-W8A8
framework:
  - Mooncake
  - OpenCompass
  - SGLang
  - vLLM
issue_type:
  - 精度问题
source: feishu
feishu_token: Q3YVdFn9Koy0PRxhf5uchGjinEd
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/Q3YVdFn9Koy0PRxhf5uchGjinEd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# Sglang测试指导（QTeam）

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/Q3YVdFn9Koy0PRxhf5uchGjinEd)

（1）模型为二级标题；测试模式为三级标题；加速卡为四级标题；

（2）对应模型需要备注编写人员；

（3）模型名称需要与社区命名一致；如果是量化模型需要明确是社区量化还是内部量化；

（4）如果不同加速卡有测试区别，需要在不同加速卡下明确，相同部分全部放在加速卡上统一编写；

（5）总结出来测试方法，不明确具体镜像；

（6）注意编写格式，如使用代码块，编写整洁；

# 导航

<add-ons component-id="" component-type-id="blk_637dcc698597401c1a8fd711" record="{"ignoreCataLogRecordIds":["FEq5d0eUfo5E2vxzQmEc0Jp1nvf","SDZMdHfPLoggcQxnuzuc8yRin7f"],"isShowAllLevel":true,"showCataLogLevel":3,"viewType":"normal"}"/>

参考sglang使用文档[[飞书知识图谱/资源/Sglang 使用文档--cKpaDnnK|Sglang 使用文档]]

**创建容器：**

<callout emoji="tada" background-color="light-orange" border-color="light-orange">

docker run -it --network=host --name=gbj_sgl --privileged --device=/dev/kfd --device=/dev/dri --ipc=host --shm-size=128G --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -u root --ulimit stack=-1:-1 --ulimit memlock=-1:-1 -v /opt/hyhal:/opt/hyhal:ro -v `pwd`:/mnt -v /public/opendas/DL_DATA/llm-models:/model:ro 镜像 bash
</callout>

## 统一设置

<callout emoji="musical_score" background-color="light-orange" border-color="light-orange">

dpsk系列中非蒸馏模型均使用**dcu_mla**后端，in4/int8在bmz使用fp8_e5m2、nmz使用fp8_e4m3

除此之外的其他模型均使用**fa3**后端

精度测试默认打开radix-cache、chunked-prefill【默认打开】，model-len可以用默认或者32768

性能测试关闭radix-cache【--disable-radix-cache】，【--chunked-prefill-size -1】，model-len可以用40k即40960
</callout>

# sglang精度测试

<text bgcolor="dark-gray">数据集及benchmark脚本获取</text>

<text bgcolor="dark-gray">curl -f -C - -o acc_data.tar.gz </text><text bgcolor="dark-gray">https://wuzh01.hpccube.com:65015/efile/s/d/Z3VvYmo=/0a79098b3acaf452</text><text bgcolor="dark-gray">   </text>

<text bgcolor="dark-gray">benchmark脚本：acc_data/sglang0510rc0</text>

<text bgcolor="dark-gray">暂时先使用sglang自带的benchmark脚本进行精度测试，测试数据集为gsm8k、ceval、mmlu</text>

<text bgcolor="dark-gray">每个脚本在sglang0510rc0/benchmark/*里的shell文件里，统一测2次取个平均值</text>

<text bgcolor="dark-gray">mmlu数据集在无网环境下会有问题，需要cl100k_base.tiktoken文件，并进行软链接下9b5ad71b2ce5302211f9c61530b329a4922fc6a4</text>
```bash {wrap}
tiktoken_cache_dir = "/mnt/gbj"    #文件路径
os.environ["TIKTOKEN_CACHE_DIR"] = tiktoken_cache_dir

# validate
assert os.path.exists(os.path.join(tiktoken_cache_dir,"cl100k_base.tiktoken"))
```

<image token="HkysbYI2KoS1UfxJ1Pkc1jjqnKK" width="1087" height="241" align="center"/>

## 环境准备【是用opencompass测试】

[[飞书知识图谱/文档/opencompass精度测试--cAib8nsm|opencompass精度测试]]

[[飞书知识图谱/文档/opencompass精度测试方法（DCU）--cfV7rn3c|opencompass精度测试方法（DCU）]]

## 环境变量设置

### 统一变量

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

export USE_DCU_CUSTOM_ALLREDUCE=1 （走custom_allreduce解决精度问题）

export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0（在 prefix是否对 chunked 数据做缓存）

export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_KVALLOC_KERNEL=1（kv cache 分配优化，triton改手写）

export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200 

export SGLANG_SET_CPU_AFFINITY=1（绑定cpu核心）

export HIP_KERNEL_BATCH_CEILING=100

export GPU_MAX_HW_QUEUES=3  （dpsk的ifb需要，会影响性能）

sysctl -w kernel.numa_balancing=0（GPU kernel性能优化）
</callout>

### 融合算子环境变量：

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用

export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1 # 大ep下无效

export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat

export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条。 mtp下无法使用

export SGLANG_USE_FUSED_RMSNORM_ROPE=1  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配

export SGLANG_USE_FUSED_RMS_QUANT=1

export SGLANG_USE_RMS_QUANT_PATH=1

export USE_FUSED_RMS_QUANT=1  # lmslim and vllm 

export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  # w8a8下未适配
</callout>

### mtp环境变量

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

export SGLANG_KVALLOC_KERNEL=1

export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1

export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1

export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1

export SGLANG_GET_LAST_LOC=1

export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1

export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

export SGLANG_ENABLE_SPEC_V2=1
</callout>

## 小龙虾模型

### Glm5-W8a8
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

export ALLREDUCE_STREAM_WITH_COMPUTE=1
# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
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

### Qwen3.5-397B-A17B-W8A8
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
export SGLANG_ENABLE_SPEC_V2=1
# triton改写算子
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
export SGLANG_USE_FUSED_TOPK_SOFTMAX=1
#export SGLANG_KV_LAYOUT_DCU_FA=1

model_path=/model/qwen3.5/Qwen3.5-397B-A17B-W8A8

model=${model_path##*/}
tp=8
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

option="--numa-node 0 0 0 0 1 1 1 1  "
#option+=" --mm-attention-backend fa3"
#option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy extra_buffer"
option+=" --quantization  w8a8_int8 "
#option+=" --mamba-scheduler-strategy no_buffer"
#option+=" --chunked-prefill-size 16384"
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

sglang serve --model-path $model_path ${option} \
                                --host $host_ip  --port 30000 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16 --kv-cache fp8_e4m3  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64 2>&1 | tee  ${logpath}/$time.log

```

### MiniMax-M2.5-W8A8
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_USE_LIGHTOP=1 
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof

export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16

export ALLREDUCE_STREAM_WITH_COMPUTE=1
export LMSLIM_USE_LIGHTOP=1
export VLLM_USE_LIGHTOP_MOE_ALIGN=1

model_path=/model/MiniMax-M2.5-W8A8
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
max_model_len=131027
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
# --dist-init-addr 10.16.1.81:5000
option="--numa-node 0 0 0 0 1 1 1 1 "
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size 16384"
#option+=" --max-running-requests 512 "
#option+=" --context-length $max_model_len"
#option+=" --disaggregation-mode decode --prefill-round-robin-balance  --host 12.12.12.116 --port 30001" #根据实际情况更换ip
#option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
option+=" --nnodes 1 --node-rank 0"
#option+=" --disable-radix-cache"
# 移除 --dist-init-addr 以及 nnodes/node-rank 手动指定，让 SGLang 自动处理
python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --quantization slimquant_marlin  --kv-cache-dtype fp8_e4m3  \
                                --trust-remote-code  --page-size 64 \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp   --dp-size $dp \
                    --tool-call-parser minimax-m2 \
                                --reasoning-parser minimax-append-think \
                                --mem-fraction-static $gpu_mem --attention-backend fa3

```

### kimi2.5
```bash {wrap}
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1


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
export SGLANG_USE_MARLIN_W4A16_MOE=1

export ALLREDUCE_STREAM_WITH_COMPUTE=1
#export MC_IB_GID_INDEX=0
export SGLANG_HEALTH_CHECK_TIMEOUT=60
export SGLANG_ROCM_USE_AITER_MOE=1

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
max_model_len=81920
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 1 1 2 2 3 3 "
#option+=" --disable-radix-cache "
#option+=" --tokenizer-path $model_path"
#option+=" --reasoning-parser kimi_k2 --tool-call-parser kimi_k2 "

#option+=" --chunked-prefill-size -1"
#option+=" --max-running-requests 512 "
#option+=" --context-length $max_model_len"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
# option+=" --pp-max-micro-batch-size 1"

#option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 2  --cuda-graph-max-bs 512"
#option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9"
#option+=" --disaggregation-ib-device mlx5_9"
#option+=" --disaggregation-mode prefill "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "

#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port $port --trust-remote-code  --page-size 64 \
                                --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```

## dcu_mla后端【dpsk】

### int4

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

NMZ：--quantization slimquant_w4a8_marlin --kv-cache-dtype <text bgcolor="light-yellow">fp8_e4m3</text>

BMZ：--quantization slimquant_w4a8_marlin --kv-cache-dtype <text bgcolor="light-yellow">fp8_e5m2</text>
</callout>

```shell {wrap}

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_RMS_QUANT=1  
export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT=1
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0
#mtp
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1
# export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）

export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
#export W8A8_SUPPORT_METHODS=1
export ALLREDUCE_STREAM_WITH_COMPUTE=1

model_path=/model/vllm-w8a8-models/DeepSeek-R1-0528-W4A8-V2
model=${model_path##*/}
tp=4 
pp=1
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
host_ip=0.0.0.0
hostname=$(hostname)
master_ip=$host_ip
max_model_len=40960
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"

#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
sglang serve --model-path $model_path ${option}  \
                                --quantization slimquant_w4a8_marlin --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30000 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```

### int8

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

NMZ：--quantization slimquant_marlin --kv-cache-dtype <text bgcolor="light-yellow">fp8_e4m3</text>

BMZ：--quantization slimquant_marlin --kv-cache-dtype <text bgcolor="light-yellow">fp8_e5m2</text>
</callout>

```shell {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_RMS_QUANT=1  
export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT=1
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1  
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0
#mtp
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#export SGLANG_USE_FA_FP8=1

#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
#export W8A8_SUPPORT_METHODS=1
#export SGLANG_NUMA_BIND_V2=0
export ALLREDUCE_STREAM_WITH_COMPUTE=1

model_path=/model/vllm-w8a8-models/DeepSeek-R1-Channel-INT8
model=${model_path##*/}
tp=8 
pp=1
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
host_ip=0.0.0.0
port=30000
hostname=$(hostname)
master_ip=$host_ip
max_model_len=40960
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4 "
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size  16384"
option+=" --max-running-requests 512 "
option+=" --context-length $max_model_len"
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 512"
#option+=" --enable-torch-compile --torch-compile-max-bs 128"
#option+=" --pp-max-micro-batch-size 2"
#option+=" --schedule-conservativeness 0.3"


#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
sglang serve --model-path $model_path ${option}  \
                                --quantization slimquant_marlin --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port $port --trust-remote-code  \
                                --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log
                                
```

### fp8

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

NMZ：--quantization slimquant_w4a8_marlin --kv-cache-dtype <text bgcolor="light-yellow">fp8_e4m3</text>

BMZ：--quantization slimquant_w4a8_marlin --kv-cache-dtype <text bgcolor="light-yellow">fp8_e5m2</text>
</callout>

### bf16【双机脚本参考，不是环境变量】

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

NMZ：--kv-cache-dtype <text bgcolor="light-yellow">fp8_e4m3</text>

BMZ：--kv-cache-dtype <text bgcolor="light-yellow">fp8_e5m2</text>
</callout>

node1：
```shell {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0 
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_RMS_QUANT=0  
export SGLANG_USE_RMS_QUANT_PATH=0
export USE_FUSED_RMS_QUANT=1
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0
#mtp
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1
# export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#export SGLANG_USE_FA_FP8=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
#export W8A8_SUPPORT_METHODS=1
export ALLREDUCE_STREAM_WITH_COMPUTE=1

export GLOO_SOCKET_IFNAME=ens14f0   #根据实际情况
export NCCL_SOCKET_IFNAME=ens14f0   #根据实际情况

model_path=/model/deepseek-r1/DeepSeek-R1-bf16

model=${model_path##*/}
tp=16 
pp=1
dp=1
ep=1
nodes=2
rank=0

host_ip=$(hostname -I | awk '{print $1}')
#host_ip=0.0.0.0
port=30000
hostname=$(hostname)
master_ip=$host_ip
max_model_len=40960
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4 "
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size  16384"
#option+=" --max-running-requests 1024 "
#option+=" --context-length $max_model_len"
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 1024"
#option+=" --enable-torch-compile --torch-compile-max-bs 128"
#option+=" --pp-max-micro-batch-size 2"
#option+=" --schedule-conservativeness 0.3"


#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
sglang serve --model-path $model_path ${option} --kv-cache-dtype fp8_e4m3 --page-size 64 \
                                --host $host_ip  --port $port --trust-remote-code  \
                                --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```

node2：与node1脚本一致
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0 
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_RMS_QUANT=0  
export SGLANG_USE_RMS_QUANT_PATH=0
export USE_FUSED_RMS_QUANT=1
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0
#mtp
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1
# export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#export SGLANG_USE_FA_FP8=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
#export W8A8_SUPPORT_METHODS=1
export ALLREDUCE_STREAM_WITH_COMPUTE=1

export GLOO_SOCKET_IFNAME=ens14f0   #根据实际情况
export NCCL_SOCKET_IFNAME=ens14f0   #根据实际情况

model_path=/model/deepseek-r1/DeepSeek-R1-bf16

model=${model_path##*/}
tp=16 
pp=1
dp=1
ep=1
nodes=2
rank=1

host_ip=$(hostname -I | awk '{print $1}')
#host_ip=0.0.0.0
port=30000
hostname=$(hostname)
master_ip=$node1_ip
max_model_len=40960
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4 "
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size  16384"
#option+=" --max-running-requests 1024 "
#option+=" --context-length $max_model_len"
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 1024"
#option+=" --enable-torch-compile --torch-compile-max-bs 128"
#option+=" --pp-max-micro-batch-size 2"
#option+=" --schedule-conservativeness 0.3"


#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --kv-cache-dtype fp8_e4m3 --page-size 64 \
                                --host $host_ip  --port $port --trust-remote-code  \
                                --dist-init-addr $master_ip:5001 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```

## fa3后端

### 一般模型

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

NMZ：kv-cache-dtype fp8_e4m3、attention_backend fa3

BMZ：kv-cache-dtype fp8_e5m2、attention_backend fa3

K100AI：attention_backend fa3
</callout>

**normal.sh**
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0

#mtp overlap
export SGLANG_ENABLE_SPEC_V2=1
# triton改写算子
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
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  # 大ep下无效
# export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat
# export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条
# export SGLANG_USE_FUSED_RMSNORM_ROPE=1  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
# export SGLANG_USE_FUSED_RMS_QUANT=1
# export USE_FUSED_RMS_QUANT=1  # lmslim and vllm
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  # w8a8下未适配
# export SGLANG_USE_RMS_QUANT_PATH=1
# export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
# export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
# export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1
export SGLANG_USE_MARLIN_W16A16_MOE=1 #int8/int4不需要

#model_path=/model/qwen3/Qwen3-30B-A3B
#model_path=/model/qwen3/Qwen3-32B
#model_path=/model/qwen3/Qwen3-8B
#model_path=/model/vllm-w8a8-models/Qwen3-32B.w8a8
#model_path=/model/vllm-w8a8-models/Qwen3-8B.w8a8
#model_path=/model/deepseek-r1/DeepSeek-R1-Distill-Llama-70B-main
#model_path=/model/deepseek-r1/DeepSeek-R1-Distill-Qwen-32B
#model_path=/model/vllm-w8a8-models/DeepSeek-R1-Distill-Llama-70B-quantized.w8a8
model_path=$1
model=${model_path##*/}
tp=$2
pp=1
dp=1
ep=1
time=$(date "+%m%d-%H%M")

mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$(hostname)"
#max_model_len=32768
gpu_mem=0.9
hostname=$(hostname)
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$(hostname -I | awk '{print $1}')
#host_ip=0.0.0.0
port=$3
port1=$4
rank=0
nodes=1

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4 "
option+=" --dtype bfloat16"
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size 16384"
#option+=" --max-running-requests 360 "
#option+=" --context-length $max_model_len "
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="16,15,15,15"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 --enable-beta-spec --cuda-graph-max-bs 32 "
#option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 4"
#option+=" --schedule-conservativeness 0.3"
#option+=" --disaggregation-decode-dp 8 --disaggregation-decode-tp 8"
#option+=" --disaggregation-ib-device mlx5_9"
    #--speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 \
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off --moe-a2a-backend deepep  --deepep-mode low_latency  --chunked-prefill-size 8192 --max-prefill-tokens 8192
sglang serve --model-path $model_path --trust-remote-code  ${option} \
      --kv-cache-dtype fp8_e4m3  --tensor-parallel-size $tp  --page-size 64 \
    --mem-fraction-static $gpu_mem --attention-backend fa3 --host $host_ip  --port $port --dist-init-addr $host_ip:$port1 \
                2>&1 | tee ${logpath}-model_len$max_model_len-mem$gpu_mem-$time-$hostname.log

```

**测试命令执行：**
```bash {wrap}
bash normal.sh /model/qwen3/Qwen3-30B-A3B 2 30000 5000
```

### 多模态模型

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

NMZ：kv-cache-dtype fp8_e4m3、attention_backend fa3、<text bgcolor="yellow">--mm-attention-backend fa3</text>

BMZ：kv-cache-dtype fp8_e5m2、attention_backend fa3、<text bgcolor="yellow">--mm-attention-backend fa3</text>
</callout>

**vl.sh**
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0

#mtp overlap
export SGLANG_ENABLE_SPEC_V2=1
# triton改写算子
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
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  # 大ep下无效
# export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat
# export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条
# export SGLANG_USE_FUSED_RMSNORM_ROPE=1  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
# export SGLANG_USE_FUSED_RMS_QUANT=1
# export USE_FUSED_RMS_QUANT=1  # lmslim and vllm
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  # w8a8下未适配
# export SGLANG_USE_RMS_QUANT_PATH=1
# export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
# export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
# export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1
export SGLANG_USE_MARLIN_W16A16_MOE=1 #int8/int4不需要

#model_path=/model/qwen3/Qwen3-VL-235B-A22B-Instruct
#model_path=/model/qwen2.5/Qwen2.5-VL-72B-Instruct
model_path=$1
model=${model_path##*/}
tp=4
pp=1
dp=1
ep=1
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$(hostname)"
#max_model_len=32768
gpu_mem=0.9
hostname=$(hostname)
host_ip=$(hostname -I | awk '{print $1}')
#host_ip=0.0.0.0
port=30000
master_ip=$(hostname -I | awk '{print $1}')
rank=0
nodes=1

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --mm-attention-backend fa3"
option+=" --dtype bfloat16"
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size 16384"
#option+=" --max-running-requests 360 "
#option+=" --context-length $max_model_len "
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="16,15,15,15"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 --enable-beta-spec --cuda-graph-max-bs 32 "
#option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 4"
#option+=" --schedule-conservativeness 0.3"
#option+=" --disaggregation-decode-dp 8 --disaggregation-decode-tp 8"
#option+=" --disaggregation-ib-device mlx5_9"
    #--speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 \
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off --moe-a2a-backend deepep  --deepep-mode low_latency  --chunked-prefill-size 8192 --max-prefill-tokens 8192
sglang serve --model-path $model_path --trust-remote-code  ${option} \
      --kv-cache-dtype fp8_e4m3  --tensor-parallel-size $tp  --page-size 64 --nnodes $nodes --node-rank $rank \
    --mem-fraction-static $gpu_mem --attention-backend fa3 --host $host_ip  --port $port --dist-init-addr $master_ip:5000 \
                2>&1 | tee  ${logpath}-model_len$max_model_len-mem$gpu_mem-$time-$hostname.log

```

**测试命令执行：**
```bash {wrap}
bash vl.sh /model/qwen2.5/Qwen2.5-VL-72B-Instruct
```

### next模型

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

NMZ：kv-cache-dtype fp8_e4m3、attention_backend fa3、--mamba-scheduler-strategy extra_buffer

BMZ：kv-cache-dtype fp8_e5m2、attention_backend fa3、--mamba-scheduler-strategy extra_buffer
</callout>

next.sh
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_TORCH_PROFILER_DIR=/home/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0

#mtp overlap
export SGLANG_ENABLE_SPEC_V2=1
# triton改写算子
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
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  # 大ep下无效
# export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat
# export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条
# export SGLANG_USE_FUSED_RMSNORM_ROPE=1  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
# export SGLANG_USE_FUSED_RMS_QUANT=1
# export USE_FUSED_RMS_QUANT=1  # lmslim and vllm
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  # w8a8下未适配
# export SGLANG_USE_RMS_QUANT_PATH=1
# export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
# export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
# export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1
export SGLANG_USE_MARLIN_W16A16_MOE=1 #int8/int4不需要

#model_path=/model/qwen3/Qwen3-30B-A3B
#model_path=/model/qwen3/Qwen3-32B
#model_path=/model/qwen3/Qwen3-8B
#model_path=/model/vllm-w8a8-models/Qwen3-32B.w8a8
#model_path=/model/vllm-w8a8-models/Qwen3-8B.w8a8
#model_path=/model/deepseek-r1/DeepSeek-R1-Distill-Llama-70B-main
#model_path=/model/deepseek-r1/DeepSeek-R1-Distill-Qwen-32B
#model_path=/model/vllm-w8a8-models/DeepSeek-R1-Distill-Llama-70B-quantized.w8a8
model_path=$1
model=${model_path##*/}
tp=$2
pp=1
dp=1
ep=1
time=$(date "+%m%d-%H%M")

mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$(hostname)"
#max_model_len=32768
gpu_mem=0.9
hostname=$(hostname)
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$(hostname -I | awk '{print $1}')
#host_ip=0.0.0.0
port=$3
port1=$4
rank=0
nodes=1

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4 "
option+=" --dtype bfloat16"
option+="  --mamba-scheduler-strategy extra_buffer"
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size 16384"
#option+=" --max-running-requests 360 "
#option+=" --context-length $max_model_len "
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="16,15,15,15"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 --enable-beta-spec --cuda-graph-max-bs 32 "
#option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 4"
#option+=" --schedule-conservativeness 0.3"
#option+=" --disaggregation-decode-dp 8 --disaggregation-decode-tp 8"
#option+=" --disaggregation-ib-device mlx5_9"
    #--speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 \
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off --moe-a2a-backend deepep  --deepep-mode low_latency  --chunked-prefill-size 8192 --max-prefill-tokens 8192
sglang serve --model-path $model_path --trust-remote-code  ${option} \
      --kv-cache-dtype fp8_e4m3  --tensor-parallel-size $tp  --page-size 64 \
    --mem-fraction-static $gpu_mem --attention-backend fa3 --host $host_ip  --port $port --dist-init-addr $host_ip:$port1 \
                2>&1 | tee ${logpath}-model_len$max_model_len-mem$gpu_mem-$time-$hostname.log

```

# 性能测试

<callout emoji="star2" background-color="light-orange" border-color="light-orange">

性能测试关闭radix-cache【--disable-radix-cache】，【--chunked-prefill-size -1】，model-len可以用40k即40960
</callout>

**客户端脚本：**
```bash {wrap}
model_path=/model/vllm-w8a8-models/DeepSeek-R1-0528-W4A8-V2

model=${model_path##*/}
tp=8
time=$(date "+%m%d-%H%M")

mode="cuda"
logpath="./client-das18/${model}-${mode}-$time-tp$tp"

data_type="w4a8"
port=30003
csv_name="${model}-tp${tp}-$time"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

pairs=("4000 1500")

echo "input,output,request_rate,num_prompts,max_concurrency,concurrency,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log

batch_seq=$({
  seq 1 1 64
  seq 66 2 256
  seq 260 4 512
  seq 520 8 1024
})

pairs=(
  "512 512 1024"
  "1024 1024 512"
  "4096 1024 256"
  "16384 1024 64"
  "4096 4096 256"
  "512 4096 512"
  "32768 1024 32"
  "16384 1024 64"
)


for pair in "${pairs[@]}"; do
  prompt_tokens=$(echo "$pair" | awk '{print $1}')
  completion_tokens=$(echo "$pair" | awk '{print $2}')
  max_bs=$(echo "$pair" | awk '{print $3}')

       for batch in $batch_seq;do
        [[ $batch -gt $max_bs ]] && break

        python -m sglang.bench_serving --backend sglang-oai \
        --model ${model_path} \
        --port $port --host 10.16.6.82 \
        --dataset-name random \
        --random-range-ratio 1 \
        --dataset-path ShareGPT_V3_unfiltered_cleaned_split.json \
        --random-input-len $prompt_tokens \
        --random-output-len $completion_tokens \
        --num-prompts $batch --max-concurrency $batch  2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log

        #Total token throughput
        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        request_rate=`grep -a "^Traffic request rate" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $2}'`


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


        echo "$prompt_tokens,$completion_tokens,$request_rate,$batch,$batch,$Concurrency,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log


     done
done

```

## dcu_mla后端：

### DeepSeek-R1-0528-W4A8-V2

### nmz-tp4
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
#export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
#export ROCSHMEM_HEAP_SIZE=1900000000
#export ROCSHMEM_MAX_NUM_CONTEXTS=32
#export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384
#export ROCSHMEM_TOPO_FILE="/workspace/built-in-BW-topo-input.xml"
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  # disable in w8a8
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_RMS_QUANT=1  # disable in w8a8
export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT=1
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  # disable in w8a8
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0
#mtp
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1
# export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
export SGLANG_USE_FA_FP8=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=1
#export SGLANG_NUMA_BIND_V2=0

export ALLREDUCE_STREAM_WITH_COMPUTE=1

#model_path=/model/vllm-w8a8-models/DeepSeek-R1-Channel-INT8
model_path=/model/vllm-w8a8-models/DeepSeek-R1-0528-W4A8-V2
#model_path=/model/deepseek-r1/DeepSeek-R1-Distill-Llama-8B
model=${model_path##*/}
tp=4 #K100AI为8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
max_model_len=40960
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size  16384"
option+=" --max-running-requests 1024 "
option+=" --context-length $max_model_len"
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 256 "
#option+=" --enable-torch-compile --torch-compile-max-bs 128"
#option+=" --pp-max-micro-batch-size 2"

python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --quantization slimquant_w4a8_marlin --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30003 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```


### bmz-tp8
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
#export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
#export ROCSHMEM_HEAP_SIZE=1900000000
#export ROCSHMEM_MAX_NUM_CONTEXTS=32
#export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384
#export ROCSHMEM_TOPO_FILE="/workspace/built-in-BW-topo-input.xml"
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=1  # disable in w8a8
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_RMS_QUANT=1  # disable in w8a8
export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT=1
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  # disable in w8a8
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0
#mtp
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1
# export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#export SGLANG_USE_FA_FP8=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=1
#export SGLANG_NUMA_BIND_V2=0

export ALLREDUCE_STREAM_WITH_COMPUTE=1

#model_path=/model/vllm-w8a8-models/DeepSeek-R1-Channel-INT8
model_path=/model/vllm-w8a8-models/DeepSeek-R1-0528-W4A8-V2
#model_path=/model/deepseek-r1/DeepSeek-R1-Distill-Llama-8B
model=${model_path##*/}
tp=4 #K100AI为8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
max_model_len=40960
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size  16384"
option+=" --max-running-requests 1024 "
option+=" --context-length $max_model_len"
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 256 "
#option+=" --enable-torch-compile --torch-compile-max-bs 128"
#option+=" --pp-max-micro-batch-size 2"

python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --quantization slimquant_w4a8_marlin --kv-cache-dtype fp8_e5m2 \
                                --host $host_ip  --port 30003 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```


## fa3后端：

### DeepSeek-R1-Distill-Llama-70B

### nmz-tp2
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
#export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
#export ROCSHMEM_HEAP_SIZE=1900000000
#export ROCSHMEM_MAX_NUM_CONTEXTS=32
#export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384
#export ROCSHMEM_TOPO_FILE="/workspace/built-in-BW-topo-input.xml"
#export SGLANG_USE_LIGHTOP=1
#export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # disable in w8a8
#export SGLANG_USE_OPT_CAT=1
#export SGLANG_USE_FUSED_RMS_QUANT=1  # disable in w8a8
#export SGLANG_USE_RMS_QUANT_PATH=1
#export USE_FUSED_RMS_QUANT=1
#export SGLANG_USE_FUSED_SILU_MUL_QUANT=1  # disable in w8a8
#export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
sysctl -w kernel.numa_balancing=0
#mtp
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）

#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
#export W8A8_SUPPORT_METHODS=1
#export SGLANG_NUMA_BIND_V2=0

export ALLREDUCE_STREAM_WITH_COMPUTE=1

model_path=/model/deepseek-r1/DeepSeek-R1-Distill-Llama-70B
model=${model_path##*/}
tp=2 #K100AI为8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
max_model_len=32768
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size  -1"
option+=" --max-running-requests 1024 "
option+=" --context-length $max_model_len"
#option+=" --disable-cuda-graph "
#option+=" --chunked-prefill-size -1"
#export SGLANG_PP_LAYER_PARTITION="10,8,7,7,8,7,7,7"
#option+=" --moe-dense-tp-size 1 "
#option+=" --ep-size $ep "
#option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 256 "
#option+=" --enable-torch-compile --torch-compile-max-bs 128"
#option+=" --pp-max-micro-batch-size 2"

python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30003 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 2>&1 | tee  ${logpath}-$time.log

```


# PD分离测试流程

[[飞书知识图谱/资源/PD 分离.md--ckWQSnfd|PD 分离.md]]

## 关联资源

- [[飞书知识图谱/资源/Sglang 使用文档--cKpaDnnK|Sglang 使用文档]] · 飞书源链接不可用
- [[飞书知识图谱/文档/opencompass精度测试--cAib8nsm|opencompass精度测试]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/MNxwwIk2HizPgfkyxbIcAib8nsm)
- [[飞书知识图谱/文档/opencompass精度测试方法（DCU）--cfV7rn3c|opencompass精度测试方法（DCU）]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/VADRwyctUiLkvykzRN6cfV7rn3c)
- [[飞书知识图谱/资源/PD 分离.md--ckWQSnfd|PD 分离.md]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/精度问题|精度问题]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/deepseek-r1|deepseek-r1]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-0528-W4A8-V2|DeepSeek-R1-0528-W4A8-V2]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-bf16|DeepSeek-R1-bf16]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-INT8|DeepSeek-R1-Channel-INT8]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B|DeepSeek-R1-Distill-Llama-70B]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B-main|DeepSeek-R1-Distill-Llama-70B-main]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B-quantized.w8a8|DeepSeek-R1-Distill-Llama-70B-quantized.w8a8]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-8B|DeepSeek-R1-Distill-Llama-8B]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Qwen-32B|DeepSeek-R1-Distill-Qwen-32B]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/Glm5-W8a8|Glm5-W8a8]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/Kimi-K2.5|Kimi-K2.5]]（model）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/minimax-append-think|minimax-append-think]]（model）
- [[飞书知识图谱/实体/minimax-m2|minimax-m2]]（model）
- [[飞书知识图谱/实体/MiniMax-M2.5-W8A8|MiniMax-M2.5-W8A8]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/NVSHMEM|NVSHMEM]]（concept）
- [[飞书知识图谱/实体/OpenCompass|OpenCompass]]（framework）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PD 分离|PD 分离]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/Qwen2.5-VL-72B-Instruct|Qwen2.5-VL-72B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen3-30B-A3B|Qwen3-30B-A3B]]（model）
- [[飞书知识图谱/实体/Qwen3-32B|Qwen3-32B]]（model）
- [[飞书知识图谱/实体/Qwen3-32B.w8a8|Qwen3-32B.w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-8B|Qwen3-8B]]（model）
- [[飞书知识图谱/实体/Qwen3-8B.w8a8|Qwen3-8B.w8a8]]（model）
- [[飞书知识图谱/实体/Qwen3-VL-235B-A22B-Instruct|Qwen3-VL-235B-A22B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen3.5-397B-A17B-W8A8|Qwen3.5-397B-A17B-W8A8]]（model）
- [[飞书知识图谱/实体/RPS|RPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

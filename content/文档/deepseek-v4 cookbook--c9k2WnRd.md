---
title: deepseek-v4 cookbook
tags:
  - AI
  - LLM
  - Benchmark
model:
  - deepseek-v4
  - Deepseek-V4
  - DeepSeek-V4
  - deepseek-v4-dev
  - DeepSeek-V4-Flash-Channel-F8-w8a8
  - DeepSeek-V4-Flash-FP8
  - DeepSeek-V4-Flash-FP8-Channel
  - DeepSeek-V4-Flash-INT8-Channel
  - DeepSeek-V4-Pro-Channel-FP8-w8a8
  - DeepSeek-V4-Pro-FP8
  - DeepSeek-V4-Pro-FP8-Channel
  - deepseekv4-v2.image.tar.gz
framework:
  - SGLang
issue_type:
  - OOM
source: feishu
feishu_token: PpYXdSuOSov8qexgnxBc9k2WnRd
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/PpYXdSuOSov8qexgnxBc9k2WnRd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# deepseek-v4 cookbook

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/PpYXdSuOSov8qexgnxBc9k2WnRd)

## DeepSeek-V4-Flash-FP8-Channel

### 环境搭载

镜像：http://42.228.13.241:18000/docker-images/deepseekv4-v2.image.tar.gz

补充地址：[<text underline="true">harbor.sourcefind.cn:5443/dcu/admin/base/custom:</text>](http%3A%2F%2Fharbor.sourcefind.cn%3A5443%2Fdcu%2Fadmin%2Fbase%2Fcustom%3A)<text underline="true">sglang-deepseek-v4-dev</text>

1. **/public/home/wangaq/images/deepseekv4.image.tar.gz**
1. /public/home/wangaq/images/deepseekv4-v2.image.tar.gz

代码仓库：https://developer.sourcefind.cn/codes/OpenDAS/sglang/-/tree/v0.5.10_dpsk_v4

模型权重：https://www.modelscope.cn/models/hygon/DeepSeek-V4-Flash-Channel-F8-w8a8

依赖包:

目录：/public/home/wangaq/whl/

Tilelang: tilelang-0.1.9+cpu.git9561fceb-cp38-abi3-linux_x86_64.whl

transfomers:transformers-5.7.0.dev0-py3-none-any.whl

lightop:lightop-0.6.0+das.dtk2604.torch290.2604301745.gd338bc-cp310-cp310-manylinux_2_28_x86_64.whl

使用镜像1需要手动更新依赖包

<text bgcolor="light-red">推荐</text><text bgcolor="light-red">镜像2</text>，已经安装了相关包，目前处于开发阶段，如果后续有更新相关包，再安装

容器：
```shell {wrap}
# 容器
docker run  -it \
--name "wangaq-deepseekv4" \
--privileged \
--shm-size=64G \
--device=/dev/kfd \
--device=/dev/dri/ \
--cap-add=SYS_PTRACE \
--security-opt seccomp=unconfined \
--ulimit memlock=-1:-1 \
--ipc=host \
--network host \
--group-add video \
--privileged \
-v /usr/local/hyhal:/usr/local/hyhal:ro \
-v /opt/hyhal:/opt/hyhal:ro \
-v /data:/nvme:ro \
-v /parastor:/parastor:ro \
-v /module:/root/module \
-v /module1:/root/module1 \
-v /public/opendas/DL_DATA/llm-models:/root/llm-models:ro \
-v /public/home/wangaq/sglang:/root/sglang \
-v /public/home/wangaq/dtk:/root/dtk \
-v /public/home/wangaq/whl:/root/whl \
wangaq-deepseekv4:v2  /bin/bash
```

### SGLang代码拉取

需更新最新的SGLang仓库代码，如没有权限请联系孙毅老师解决。
```plaintext
git clone -b v0.5.10_dpsk_v4 https://developer.sourcefind.cn/codes/OpenDAS/sglang.git
```

### **需更新****必要的package**

**aiter/deep_ep/deepgemm/flash_mla/tilelang  拉取最新das出的包，性能以这个为准，后续拉取端到端trace对比看看是否kernel性能满足要求**

### 启动命令

<callout emoji="white_check_mark" background-color="light-green" border-color="light-green">

该配置用于 DeepSeek-V4-Flash-FP8-Channel，已验证路径为单机 CP8EP8。EP/MegaMoE 路径当前未作为跑通配置写入本 cookbook。
</callout>

**需要更新的包**

lightop

<view type="1">

  <file token="UNl1bSB26oKzybxfSq7cxlnTnOh" name="lightop-0.6.0+das1.1git9442d31.abi1.dtk2604.torch2.9-cp310-cp310-linux_x86_64.whl"/>

</view>

flash_mla:
```bash
http://pypi.sourcefind.cn:666/customer/dtk2604-dcc2604-0422-aicc/+f/00a/bc5d57c9e1d2d/flash_mla-1.2.0+das.opt1.dtk2604.torch290.2606032016.g3de4f9.aicc-cp310-cp310-manylinux_2_28_x86_64.whl#sha256=00abc5d57c9e1d2dd1afba992d54d71c8ca4289f2f6612f44213db53926b2ebf 
```

#### 单机CP8EP8（Prefill较优并行实践，已验证 split prefill/decode）

Deepep_config.json
```json
{
  "normal_dispatch": {
    "num_sms": 48,
    "num_max_nvl_chunked_send_tokens": 6,
    "num_max_nvl_chunked_recv_tokens": 256,
    "num_max_rdma_chunked_send_tokens": 6,
    "num_max_rdma_chunked_recv_tokens": 128
  },
  "normal_combine": {
    "num_sms": 48,
    "num_max_nvl_chunked_send_tokens": 4,
    "num_max_nvl_chunked_recv_tokens": 256,
    "num_max_rdma_chunked_send_tokens": 6,
    "num_max_rdma_chunked_recv_tokens": 128
  }
}
```

```bash
export NCCL_SOCKET_IFNAME=ens19f0 # ip自查
export GLOO_SOCKET_IFNAME=ens19f0 # ip自查
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_OPT_USE_FUSED_STORE_CACHE=false
export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0
export SGLANG_ENABLE_SPEC_V2=1
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FP8_W8A8_MOE=1
export SGLANG_USE_DEEPGEMM_MOE=1
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export SGLANG_ROCM_USE_AITER_MOE=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_MLA_CAT=1
export SGLANG_USE_LIGHTOP_GROUP_FP8_QUANT=1
export SGLANG_USE_FUSED_DPSKV4_SILU_MUL_FP8_QUANT=1
export SGLANG_USE_LINEAR_BF16_FP32_USE_BLASLT=1
export SGLANG_APPLY_CONFIG_BACKUP=none
export SGLANG_ROCM_USE_AITER_TILELANG_MHC=1
export SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA=1
export SGLANG_OPT_FLASHMLA_SPARSE_PREFILL=1
export SGLANG_USE_DPSKV4_LIGHTOP_QUANT_K_CACHE=1
export SGLANG_USE_DPSKV4_LIGHTOP_RMSNORM=1
export SGLANG_USE_FUSED_DPSKV4_QNORM_ROPE_KV_ROPE_QUANT=1
export SGLANG_USE_LIGHTOP_EP_MOE_ALIGN=1
export SGLANG_USE_LIGHTOP_EP_SCATTER=1
export SGLANG_USE_LIGHTOP_EP_GATHER=1
export SGLANG_USE_LIGHTOP_TOPK_IDS_POSTPROCESS=1

sglang serve \
 --tp-size 8 \
 --dist-timeout 10000 \
 --watchdog-timeout 3600 \
 --port 10015 \
 --host 0.0.0.0 \
 --model-path /home/model/DeepSeek-V4-Flash-FP8-Channel \
 --disable-radix-cache \
 --model-loader-extra-config '{"enable_multithread_load": "true","num_threads": 64}' \
 --trust-remote-code \
 --chunked-prefill-size 32768 \
 --disable-flashinfer-autotune \
 --skip-server-warmup \
 --enable-nsa-prefill-context-parallel \
 --nsa-prefill-cp-mode round-robin-split \
 --moe-a2a-backend deepep \
 --deepep-mode auto \
 --deepep-config deepep_config_path \
 --cuda-graph-max-bs 128 \
 --max-total-tokens 1048576
```

<callout emoji="writing_hand" background-color="light-orange" border-color="light-orange">

Tips

如果碰到rocshemem init问题，需要指定ROCSHMEM_IB_GID_INDEX，具体的值需要和通信库对齐，每一台机器不一样
</callout>

#### 单机TP4PP2

**启动命令**
```bash
#!/bin/bash
set -euo pipefail

cd /root/sglang/sglang-gitlab/python

# 更新 flash_mla/lightop 或 JIT kernel 后建议清理缓存，避免继续加载旧 kernel。
rm -rf ~/.cache/

export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export PYTHONPATH=/root/sglang/sglang-gitlab/python:${PYTHONPATH:-}

# DeepSeek-V4 Flash Channel 常用优化开关。
export SGLANG_APPLY_CONFIG_BACKUP=none
export SGLANG_DSV4_CHANNEL_FP8_SCALE=1
export SGLANG_ROCM_USE_AITER_MOE=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_MLA_CAT=1
export SGLANG_USE_LIGHTOP_GROUP_FP8_QUANT=1
export SGLANG_USE_FUSED_DPSKV4_SILU_MUL_FP8_QUANT=1
export SGLANG_USE_LINEAR_BF16_FP32_USE_BLASLT=1
export SGLANG_ROCM_USE_AITER_TILELANG_MHC=1
export SGLANG_USE_DPSKV4_LIGHTOP_QUANT_K_CACHE=1
export SGLANG_USE_DPSKV4_LIGHTOP_RMSNORM=1
export SGLANG_USE_FUSED_DPSKV4_QNORM_ROPE_KV_ROPE_QUANT=1
export SGLANG_USE_LIGHTOP_EP_MOE_ALIGN=1
export SGLANG_USE_LIGHTOP_EP_SCATTER=1
export SGLANG_USE_LIGHTOP_EP_GATHER=1
export SGLANG_USE_LIGHTOP_TOPK_IDS_POSTPROCESS=1
export SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
export USE_DCU_CUSTOM_ALLREDUCE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1
export SGLANG_USE_LIGHTOP=1
export SGLANG_OPT_USE_FUSED_STORE_CACHE=false
export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0

# split prefill/decode MLA 路径。
export SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA=1
export SGLANG_OPT_FLASHMLA_SPARSE_PREFILL=1
unset SGLANG_FLASH_MLA_ENTRYPOINT_DEBUG

MODEL_PATH=/model/DeepSeek-V4-Flash-FP8-Channel
PORT=30000

python3 -m sglang.launch_server \
  --trust-remote-code \
  --model-path "${MODEL_PATH}" \
  --tp 4 \
  --pipeline-parallel-size 2 \
  --disable-radix-cache \
  --chunked-prefill-size 3072 \
  --disable-flashinfer-autotune \
  --host 0.0.0.0 \
  --port "${PORT}" \
  2>&1 | tee serve.log
```

**验证点**

- 启动日志中确认 `SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA=1` 和 `SGLANG_OPT_FLASHMLA_SPARSE_PREFILL=1` 生效。
- 该路径用于 TP4PP2；不要把尚未跑通的 EP/MegaMoE 参数混入该命令。
- 如遇 JIT kernel 或 VMFault 类问题，先清理缓存并重新启动服务，再从日志中第一处 `KERNEL VMFault` 和第一个 `kernel name` 开始定位。

#### 单机TP8

拉取最新仓库代码后，到sglang/python/目录下启动如下命令
```bash
#!/bin/bash

# rm -rf ~/.cache/
# rm -rf ./dsv4_nan_dumps

hy-smi

# export SGLANG_TORCH_PROFILER_DIR="/root/sglang/prof-dpsk-v4"
######## 20260519更新 ##########
rm -rf ~/.cache/ # 清理jit kernel缓存，解决开启cudagraph可能出现的卡住现象
export SGLANG_ROCM_USE_AITER_MOE=1 # 开启aiter moe算子优化
export SGLANG_USE_OPT_CAT=1 
export SGLANG_USE_FUSED_MLA_CAT=1 
export SGLANG_USE_LIGHTOP_GROUP_FP8_QUANT=1 # 开启act_quant算子优化
export SGLANG_USE_FUSED_DPSKV4_SILU_MUL_FP8_QUANT=1 # 开启silu_mul_fp8_quant融合算子优化
export SGLANG_USE_LINEAR_BF16_FP32_USE_BLASLT=1 # 开启inear_bf16_fp32算子优化
######## 20260519更新 ##########


######## 20260511更新 ##########
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
# 算子优化
export USE_DCU_CUSTOM_ALLREDUCE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1 
export SGLANG_USE_LIGHTOP=1 # 开启 lightop的 topK算子(fused_moe_gate)/moe_align算子/moe_sum算子【开启aite moe算子时，只使用到topK算子】
#export SGLANG_USE_FP8_W8A8_MOE=1 # 开启 lightop的 Marlin MOE W8A8 算子
#export SGLANG_ROCM_USE_AITER_MOE=0  # 关闭aiter moe算子(默认开启)
######## 20260511更新 ##########

export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
# export TVM_FFI_GPU_BACKEND=hip
# export TVM_FFI_DISABLE_TORCH_C_DLPACK=0 

# export SGLANG_OPT_USE_TILELANG_MHC_PRE=0
export SGLANG_OPT_USE_FUSED_STORE_CACHE=false
export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0

# export GPU_FLUSH_ON_EXECUTION=true
# export SGLANG_DEBUG_DECODE_BATCH_TRACE=1
# export SGLANG_DEBUG_STREAM_TRACE=1
# export SGLANG_DEBUG_MOE_TRACE=1
# export SGLANG_HACK_FLASHMLA_BACKEND=torch
# MODEL_PATH=/root/module/DeepSeek-V4-Flash-FP8
export SGLANG_APPLY_CONFIG_BACKUP=none
MODEL_PATH=/model/DeepSeek-V4-Flash-FP8-Channel
# MODEL_PATH=/parastor/home/public_user/zhangbin/DeepSeek-V4-Flash-FP8/
port=30000

# export SGLANG_FLASH_MLA_ENTRYPOINT_DEBUG=1
sglang serve \
  --trust-remote-code \
  --model-path $MODEL_PATH \
  --tp 8 \
  --disable-radix-cache  \
  --chunked-prefill-size 3072 \
  --disable-flashinfer-autotune \
  --host 0.0.0.0 \
  --port $port \
  2>&1 | tee serve.log 
```

#### 单机TP8DP8（Decodel较优并行实践）/单机TP4DP4(tp参数修改为4）
```markdown
#!/bin/bash
rm -rf ~/.cache
export SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA=1
unset SGLANG_FLASH_MLA_ENTRYPOINT_DEBUG
export SGLANG_ENABLE_SPEC_V2=1
export SGLANG_ROCM_USE_AITER_MOE=1 
export SGLANG_USE_OPT_CAT=1 
export SGLANG_USE_FUSED_MLA_CAT=1 
export SGLANG_USE_LIGHTOP_GROUP_FP8_QUANT=1 
export SGLANG_USE_FUSED_DPSKV4_SILU_MUL_FP8_QUANT=1 
export SGLANG_USE_LINEAR_BF16_FP32_USE_BLASLT=1
export SGLANG_ROCM_USE_AITER_TILELANG_MHC=1
export SGLANG_USE_DPSKV4_LIGHTOP_QUANT_K_CACHE=1
export SGLANG_USE_DPSKV4_LIGHTOP_RMSNORM=1
export SGLANG_USE_FUSED_DPSKV4_QNORM_ROPE_KV_ROPE_QUANT=1
export SGLANG_USE_LIGHTOP_EP_MOE_ALIGN=1
export SGLANG_USE_LIGHTOP_EP_SCATTER=1
export SGLANG_USE_LIGHTOP_EP_GATHER=1
export SGLANG_USE_LIGHTOP_TOPK_IDS_POSTPROCESS=1 
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 
export HIP_D2H_DISABLE_COPY_BUFFER=0 
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 
export USE_DCU_CUSTOM_ALLREDUCE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1 
export SGLANG_USE_LIGHTOP=1 
export SGLANG_OPT_USE_FUSED_STORE_CACHE=false
export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0

export SGLANG_APPLY_CONFIG_BACKUP=none
MODEL_PATH=/module/DeepSeek-V4-Flash-FP8-Channel

port=30000

sglang serve \
  --trust-remote-code \
  --model-path $MODEL_PATH \
  --tp 8 \
  --chunked-prefill-size 2048 \
  --disable-flashinfer-autotune \
  --speculative-algorithm EAGLE \
  --speculative-num-steps 1 \
  --speculative-eagle-topk 1 \
  --speculative-num-draft-tokens 2 \
  --dp-size 8 \
  --enable-dp-attention --moe-dense-tp-size 1  --enable-dp-lm-head \
  --host 0.0.0.0 \
  --port $port \
  2>&1 | tee serve.log 
```

#### 单机TP8DP8EP8 MegaMoE（开发阶段，Prefill调试路径）

<callout emoji="💡" background-color="light-yellow" border-color="light-yellow">

当前 MegaMoE 仍处于开发调试阶段，暂时不进行前端测试。该配置优先用于单机 8 卡 prefill、profiler、VMFault 和 barrier 长尾问题定位；对外性能数据发布前需要重新跑完整精度和性能回归。
</callout>

- 框架分支：`v0.5.10_dpsk_v4_megamoe`，本地调试时需确保工作区与该分支提交对齐。
- DeepGEMM：使用 GitHub 最新代码；当前已验证更新点为 `f54cbc2 Fix staged MegaMoE rank barrier ticketing`。
- 算子要求：`K3_USE_ASM_TAIL_REDUCE` 保持 `0`，不要默认切到 asm tail reduce 路径。
- MegaMoE 目前只按 EP8 路径调试，即 `tp=8`、`dp=8`、`--enable-dp-attention`、`--moe-a2a-backend megamoe`。
```bash
#!/bin/bash
set -euo pipefail

cd /root/sglang/sglang-gitlab/python

export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export MODEL_PATH=/root/llm-models/DeepSeek-V4-Flash-FP8-Channel
export PORT=30000
export PYTHONPATH=/root/sglang/sglang-gitlab/python:/root/sglang/DeepGEMM:${PYTHONPATH:-}
export TORCH_EXTENSIONS_DIR=/root/sglang/hygon_tmp/torch_extensions_megamoe

export K3_USE_ASM_TAIL_REDUCE=0
export SGLANG_DSV4_CHANNEL_FP8_SCALE=1
export SGLANG_APPLY_CONFIG_BACKUP=none
export SGLANG_OPT_USE_DEEPGEMM_MEGA_MOE=1
export SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK=2048
export SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA=0
export SGLANG_OPT_FLASHMLA_SPARSE_PREFILL=0
export SGLANG_ROCM_USE_AITER_MOE=1
unset SGLANG_USE_AITER
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_MLA_CAT=1
export SGLANG_USE_LIGHTOP_GROUP_FP8_QUANT=1
export SGLANG_USE_FUSED_DPSKV4_SILU_MUL_FP8_QUANT=1
export SGLANG_USE_LINEAR_BF16_FP32_USE_BLASLT=1
export SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
export USE_DCU_CUSTOM_ALLREDUCE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1
export SGLANG_USE_LIGHTOP=1
export SGLANG_OPT_USE_FUSED_STORE_CACHE=false
export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0

python3 -m sglang.launch_server \
  --trust-remote-code \
  --model-path "${MODEL_PATH}" \
  --tp 8 \
  --dp 8 \
  --enable-dp-attention \
  --moe-a2a-backend megamoe \
  --chunked-prefill-size 2048 \
  --page-size 256 \
  --max-running-requests 64 \
  --mem-fraction-static 0.88 \
  --disable-flashinfer-autotune \
  --disable-radix-cache \
  --host 0.0.0.0 \
  --port "${PORT}" \
  2>&1 | tee serve_megamoe.log
```

**Prefill profiler建议：**当前只关注 prefill 时，先用 `/start_profile` 指定 `profile_by_stage=true`、`profile_stages=["prefill"]`、`num_steps=5`，压测 size 建议从 `isl4096-osl1-bs64` 开始。
```bash
curl -X POST "http://127.0.0.1:${PORT}/start_profile" \
  -H "Content-Type: application/json" \
  -d '{
    "output_dir": "/root/sglang/hygon_tmp/megamoe_profile/profile",
    "num_steps": 5,
    "activities": ["CPU", "GPU"],
    "profile_by_stage": true,
    "profile_stages": ["prefill"],
    "merge_profiles": false,
    "profile_prefix": "megamoe_isl4096_osl1_bs64"
  }'
```

- 如果只定位 barrier 或 VMFault，建议先关闭 cudagraph 或降低变量数，确认 prefill 本体无 VMFault 后再恢复 cudagraph 场景。
- prof 文件重点看各 rank 的 `rank_barrier_kernel`、MegaMoE persistent kernel、prefill stage 总耗时以及 rank 间空泡。
- 结束后执行 `/stop_profile`，并保留 `profile/*.trace.json.gz`、`serve.log`、`env.txt`、`sglang_commit.txt`、`deepgemm_commit.txt`。

#### PD分离脚本

##### P节点启动命令（CP8EP8）
```bash
rm -rf ~/.cache/
rm -rf ~/.triton/

# 健康检查环境变量
export SGLANG_HEALTH_CHECK_TIMEOUT=180

export NCCL_SOCKET_IFNAME=enp113s0f0np0 
export GLOO_SOCKET_IFNAME=enp113s0f0np0
export MC_ENABLE_DEST_DEVICE_AFFINITY=1 
export UCX_NET_DEVICES=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1                                                                                                                                                                                         
export NCCL_IB_HCA=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16

export SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA=1
export SGLANG_OPT_FLASHMLA_SPARSE_PREFILL=1
export SGLANG_OPT_USE_FUSED_STORE_CACHE=false
export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0
export SGLANG_ENABLE_SPEC_V2=1
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FP8_W8A8_MOE=1
export SGLANG_USE_DEEPGEMM_MOE=1
export SGLANG_ROCM_USE_AITER_MOE=1
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_MLA_CAT=1
export SGLANG_USE_LIGHTOP_GROUP_FP8_QUANT=1
export SGLANG_USE_FUSED_DPSKV4_SILU_MUL_FP8_QUANT=1
export SGLANG_USE_LINEAR_BF16_FP32_USE_BLASLT=1
export SGLANG_APPLY_CONFIG_BACKUP=none
export SGLANG_ROCM_USE_AITER_TILELANG_MHC=1
export SGLANG_USE_DPSKV4_LIGHTOP_QUANT_K_CACHE=1
export SGLANG_USE_DPSKV4_LIGHTOP_RMSNORM=1
export SGLANG_USE_FUSED_DPSKV4_QNORM_ROPE_KV_ROPE_QUANT=1
export SGLANG_USE_LIGHTOP_EP_MOE_ALIGN=1
export SGLANG_USE_LIGHTOP_EP_SCATTER=1
export SGLANG_USE_LIGHTOP_EP_GATHER=1
export SGLANG_USE_LIGHTOP_TOPK_IDS_POSTPROCESS=1

option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9 "
option+=" --disaggregation-mode prefill "

time=$(date +"%Y%m%d-%H%M%S")
nodes=1
rank=0
host_ip=12.12.12.20
master_ip=$host_ip

sglang serve ${option} \
 --tp-size 8 \
 --dist-timeout 10000 \
 --watchdog-timeout 3600 \
 --model-path /model/DeepSeek-V4-Flash-FP8-Channel \
 --disable-radix-cache \
 --trust-remote-code \
 --chunked-prefill-size 16384 \
 --disable-flashinfer-autotune \
 --disable-cuda-graph \
 --enable-nsa-prefill-context-parallel \
 --nsa-prefill-cp-mode round-robin-split \
 --moe-a2a-backend deepep \
 --deepep-mode auto \
 --deepep-config deepep-config.json \
 --max-total-tokens 1048576 \
 --host $host_ip  --port 30123  \
 --dist-init-addr $master_ip:5123 --nnodes $nodes --node-rank $rank \
 2>&1 | tee ./serve-p_${time}.log

```

##### D节点启动命令(TP8DP8)
```bash
#!/bin/bash
rm -rf ~/.cache/
rm -rf ~/.triton/

# 健康检查环境变量
export SGLANG_HEALTH_CHECK_TIMEOUT=180

export NCCL_SOCKET_IFNAME=enp113s0f0np0
export GLOO_SOCKET_IFNAME=enp113s0f0np0
export MC_ENABLE_DEST_DEVICE_AFFINITY=1 
export UCX_NET_DEVICES=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1                                                                                                                                                                                         
export NCCL_IB_HCA=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16

export SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA=1
export SGLANG_ENABLE_SPEC_V2=1
export SGLANG_ROCM_USE_AITER_MOE=1 
export SGLANG_USE_OPT_CAT=1 
export SGLANG_USE_FUSED_MLA_CAT=1 
export SGLANG_USE_LIGHTOP_GROUP_FP8_QUANT=1 
export SGLANG_USE_FUSED_DPSKV4_SILU_MUL_FP8_QUANT=1 
export SGLANG_USE_LINEAR_BF16_FP32_USE_BLASLT=1
export SGLANG_ROCM_USE_AITER_TILELANG_MHC=1
export SGLANG_USE_DPSKV4_LIGHTOP_QUANT_K_CACHE=1
export SGLANG_USE_DPSKV4_LIGHTOP_RMSNORM=1
export SGLANG_USE_FUSED_DPSKV4_QNORM_ROPE_KV_ROPE_QUANT=1
export SGLANG_USE_LIGHTOP_EP_MOE_ALIGN=1
export SGLANG_USE_LIGHTOP_EP_SCATTER=1
export SGLANG_USE_LIGHTOP_EP_GATHER=1
export SGLANG_USE_LIGHTOP_TOPK_IDS_POSTPROCESS=1 
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
export HIP_H2D_DISABLE_COPY_BUFFER=0 
export HIP_D2H_DISABLE_COPY_BUFFER=0 
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 
export USE_DCU_CUSTOM_ALLREDUCE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1 
export SGLANG_USE_LIGHTOP=1 
export SGLANG_OPT_USE_FUSED_STORE_CACHE=false
export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0
export SGLANG_APPLY_CONFIG_BACKUP=none

option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9 "
option+=" --disaggregation-mode decode "

time=$(date +"%Y%m%d-%H%M%S")
nodes=1
rank=0
host_ip=12.12.12.18
master_ip=$host_ip

# --speculative-algorithm EAGLE \
# --speculative-num-steps 1 \
# --speculative-eagle-topk 1 \
# --speculative-num-draft-tokens 2 \
sglang serve ${option} \
  --trust-remote-code \
  --model-path /model/DeepSeek-V4-Flash-FP8-Channel \
  --tp-size 8 \
  --chunked-prefill-size 16384 \
  --disable-flashinfer-autotune \
  --dp-size 8 \
  --enable-dp-attention --moe-dense-tp-size 1  --enable-dp-lm-head \
  --host $host_ip  --port 30125  \
  --dist-init-addr $master_ip:5123 --nnodes $nodes --node-rank $rank \
  2>&1 | tee ./serve-d_${time}.log
```

##### 路由命令
```plaintext
python -m sglang_router.launch_router --pd-disaggregation --prefill http://12.12.12.20:30123 --decode http://12.12.12.18:30125 --host 0.0.0.0 --port 10015
```


### 简单请求命令
```javascript
PORT="${PORT:-30000}"
BASE_URL="${BASE_URL:-http://127.0.0.1:${PORT}}"

curl "${BASE_URL}/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d @- <<JSON
{
  "model": "${MODEL_PATH}",
  "messages": [
    {
      "role": "user",
      "content": "甲乙两班共有学生98人，甲班比乙班多6人，求两班各有多少人？"
    }
  ]
}
JSON
```


## DeepSeek-V4-Flash-INT8-Channel

<callout emoji="camping" background-color="light-orange" border-color="light-orange">

**更新日志：**

- **[2026-05-29]** 
	- 支持int8 aiter moe，开启方法如下，**开启aitermoe后可支持PP并行，moe_marlin仍然不支持PP并行**
		- 按照文档更新代码；
		- 更新aiter组件 
      pip install aiter==0.1.3+das.opt1.dtk2604.torch290.2605231554.gc701ac -i http://pypi.sourcefind.cn:666/nightly/dtk2604/ --trusted-host pypi.sourcefind.cn
    - 启动命令去掉 `--quantization slimquant_marlin`
</callout>

- **镜像：**
	- 郑州集群 `/public/home/wangaq/images/deepseekv4-v2.image.tar.gz`
	- Harbor：[<text color="purple" underline="true">harbor.sourcefind.cn:5443/dcu/admin/base/custom:</text>](http%3A%2F%2Fharbor.sourcefind.cn%3A5443%2Fdcu%2Fadmin%2Fbase%2Fcustom%3A)<text color="purple" underline="true">sglang-deepseek-v4-dev</text>
- **权重：**
	- 郑州集群 `/parastor/home/public_user/wanglong/DeepSeek-V4-Flash-INT8-Channel`
	- modelscope：https://modelscope.cn/collections/hygon/Deepseek-V4
- **代码：** https://developer.sourcefind.cn/codes/OpenDAS/sglang/-/tree/v0.5.10_dpsk_v4
- **脚本：**
```bash {wrap}
#!/bin/bash

rm -rf ~/.cache/
# rm -rf ./dsv4_nan_dumps
# export SGLANG_TORCH_PROFILER_DIR="/root/sglang/prof-dpsk-v4"

######## 20260511更新 ##########
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
# 算子优化
export USE_DCU_CUSTOM_ALLREDUCE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1
export SGLANG_USE_FP8_W8A8_MOE=1 # 开启 lightop的 Marlin MOE W8A8 算子
export SGLANG_USE_LIGHTOP=1 # 开启 lightop的 topK算子(fused_moe_gate)/moe_align算子/moe_sum算子
export SGLANG_ROCM_USE_AITER_MOE=0  # 关闭aiter moe算子(默认开启)
######## 20260511更新 ##########

export SGLANG_OPT_USE_FUSED_HASH_TOPK=true
export SGLANG_OPT_SWIGLU_CLAMP_FUSION=false
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=true
export SGLANG_NSA_FUSE_TOPK=false  # temp: bypass lightop fused topk to isolate crash
export SGLANG_JIT_DEEPGEMM_PRECOMPILE=0

export SGLANG_APPLY_CONFIG_BACKUP=none
MODEL_PATH=$PWD/DeepSeek-V4-Flash-INT8-Channel/

export SGLANG_TOPK_TRANSFORM_512_TORCH=1
export SGLANG_DSV4_MODE=2604
export PYTHONPATH=$PWD/code/sglang/python:$PYTHONPATH

sglang serve \
  --port 30001 \
  --trust-remote-code \
  --model-path $MODEL_PATH \
  --tp $1 \
  --disable-radix-cache  \
  --chunked-prefill-size 3072 \
  --quantization slimquant_marlin \
  --chat-template $PWD/code/sglang_sg/examples/chat_template/tool_chat_template_deepseekv3.jinja \
  --kv-cache-dtype auto  --disable-flashinfer-autotune
```

## DeepSeek-V4-Pro-FP8 / DeepSeek-V4-Pro-FP8-Channel

- **镜像：**
	- 郑州集群 `/public/home/wangaq/images/deepseekv4-v2.image.tar.gz`
	- 容器名：`wangaq-deepseekv4`
	- 代码仓库：https://developer.sourcefind.cn/codes/OpenDAS/sglang/-/tree/v0.5.10_dpsk_v4
- **权重：**
	- Pro blockwise：`/root/module1/DeepSeek-V4-Pro-FP8`
	- Pro Channelwise：https://www.modelscope.cn/models/hygon/DeepSeek-V4-Pro-Channel-FP8-w8a8
- **机器：**
	- `10.16.1.26` 作为 rank0
	- `10.16.1.28` 作为 rank1
	- 两机各使用 8 张卡，整体 `pp2 tp8`
- **脚本：**
	- 进入容器后执行以下启动脚本
	- rank1 需要先启动，rank0 后启动
	- Pro Channel 使用 Channel 模型目录自己的 config 和 tokenizer，不需要额外设置 `SGLANG_DSV4_CHANNEL_FP8_SCALE`
```bash

#!/usr/bin/env bash
set -euo pipefail

NODE_RANK="${NODE_RANK:-0}"
if [[ $# -gt 0 ]]; then
  NODE_RANK="$1"
  shift
fi

WORKDIR="${SGLANG_WORKDIR:-/root/sglang}"
if [[ ! -r "${WORKDIR}/sglang-gitlab/python" ]]; then
  WORKDIR="/public/home/wangaq/sglang"
fi
SCRIPT_DIR="${WORKDIR}/sglang-gitlab/python"

export MODEL_PATH="${MODEL_PATH:-/root/module1/DeepSeek-V4-Pro-FP8}"
export TOKENIZER_PATH="${TOKENIZER_PATH:-$MODEL_PATH}"
export HIP_VISIBLE_DEVICES="${HIP_VISIBLE_DEVICES:-0,1,2,3,4,5,6,7}"
PYTHONPATH="${PYTHONPATH:-}"
export PYTHONPATH="$SCRIPT_DIR:$PYTHONPATH"
export SGLANG_TORCH_PROFILER_DIR="${SGLANG_TORCH_PROFILER_DIR:-$WORKDIR/prof-dpsk-v4}"

export SGLANG_OPT_USE_FUSED_STORE_CACHE="${SGLANG_OPT_USE_FUSED_STORE_CACHE:-false}"
export SGLANG_OPT_USE_FUSED_HASH_TOPK="${SGLANG_OPT_USE_FUSED_HASH_TOPK:-true}"
export SGLANG_OPT_SWIGLU_CLAMP_FUSION="${SGLANG_OPT_SWIGLU_CLAMP_FUSION:-false}"
export SGLANG_TOPK_TRANSFORM_512_TORCH="${SGLANG_TOPK_TRANSFORM_512_TORCH:-false}"
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK="${SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK:-false}"
export SGLANG_JIT_DEEPGEMM_PRECOMPILE="${SGLANG_JIT_DEEPGEMM_PRECOMPILE:-0}"
export SGLANG_ROCM_USE_AITER_MOE="${SGLANG_ROCM_USE_AITER_MOE:-false}"

export SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA="${SGLANG_DSV4_SPLIT_PREFILL_DECODE_MLA:-0}"
export SGLANG_DSV4_SPLIT_HCA_NONSPARSE_MLA="${SGLANG_DSV4_SPLIT_HCA_NONSPARSE_MLA:-false}"
export SGLANG_DSV4_SPARSE_PREFILL_SINGLE_CALL="${SGLANG_DSV4_SPARSE_PREFILL_SINGLE_CALL:-false}"
export SGLANG_DSV4_SPARSE_PREFILL_TRITON_GATHER="${SGLANG_DSV4_SPARSE_PREFILL_TRITON_GATHER:-false}"
export SGLANG_DISABLED_MODEL_ARCHS="${SGLANG_DISABLED_MODEL_ARCHS:-midashenglm}"
export SGLANG_DEBUG_DSV4_LOAD="${SGLANG_DEBUG_DSV4_LOAD:-0}"
export SGLANG_APPLY_CONFIG_BACKUP="${SGLANG_APPLY_CONFIG_BACKUP:-none}"

export TP="${TP:-8}"
export PP="${PP:-2}"
export EP_SIZE="${EP_SIZE:-1}"
export NNODES="${NNODES:-2}"
export DIST_INIT_ADDR="${DIST_INIT_ADDR:-10.16.1.26:20000}"
export HOST="${HOST:-0.0.0.0}"
export PORT="${PORT:-30000}"
export CHUNKED_PREFILL_SIZE="${CHUNKED_PREFILL_SIZE:-3072}"
export MEM_FRACTION_STATIC="${MEM_FRACTION_STATIC:-0.80}"
export MAX_RUNNING_REQUESTS="${MAX_RUNNING_REQUESTS:-64}"
if [[ -z "${SERVE_LOG:-}" ]]; then
  export SERVE_LOG="$WORKDIR/hygon_tmp/pro_multinode_rank$NODE_RANK.log"
else
  export SERVE_LOG
fi

cd "${SCRIPT_DIR}"
SERVE_LOG_DIR="${SERVE_LOG%/*}"
mkdir -p "$SERVE_LOG_DIR" "$SGLANG_TORCH_PROFILER_DIR"
: > "${SERVE_LOG}"
exec > >(tee -a "${SERVE_LOG}") 2>&1

echo "== Starting DeepSeek V4 Pro multi-node =="
echo "host: $(hostname)"
echo "node_rank: ${NODE_RANK}"
echo "nnodes: ${NNODES}"
echo "dist_init_addr: ${DIST_INIT_ADDR}"
echo "model: ${MODEL_PATH}"
echo "tokenizer: ${TOKENIZER_PATH}"
echo "tp: ${TP}"
echo "pp: ${PP}"
echo "devices: ${HIP_VISIBLE_DEVICES}"
echo "port: ${PORT}"
echo "log: ${SERVE_LOG}"

python -m sglang.launch_server \
  --trust-remote-code \
  --model-path "${MODEL_PATH}" \
  --tokenizer-path "${TOKENIZER_PATH}" \
  --tp-size "${TP}" \
  --pp-size "${PP}" \
  --ep-size "${EP_SIZE}" \
  --nnodes "${NNODES}" \
  --node-rank "${NODE_RANK}" \
  --dist-init-addr "${DIST_INIT_ADDR}" \
  --dist-timeout 1800 \
  --chunked-prefill-size "${CHUNKED_PREFILL_SIZE}" \
  --mem-fraction-static "${MEM_FRACTION_STATIC}" \
  --max-running-requests "${MAX_RUNNING_REQUESTS}" \
  --disable-overlap-schedule \
  --disable-flashinfer-autotune \
  --disable-radix-cache \
  --host "${HOST}" \
  --port "${PORT}" \
  "$@"

```

### 容器内 bash 启动方式（不重启容器）

如果容器 `wangaq-deepseekv4` 已经正常运行，不需要重启或重建容器。直接进入容器 bash 后设置环境变量，再执行 `/root/sglang/hygon_tmp/start_pro_multinode_node.sh`。启动顺序仍然是先 `10.16.1.28` 的 rank1，再启动 `10.16.1.26` 的 rank0。

在 `10.16.1.28` 进入容器并启动 Pro Channel rank1：
```bash
docker exec -it wangaq-deepseekv4 /bin/bash

export MODEL_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel
export TOKENIZER_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel
export DIST_INIT_ADDR=10.16.1.26:20000
export SGLANG_APPLY_CONFIG_BACKUP=none
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=false

bash /root/sglang/hygon_tmp/start_pro_multinode_node.sh 1
```

在 `10.16.1.26` 进入容器并启动 Pro Channel rank0：
```bash
docker exec -it wangaq-deepseekv4 /bin/bash

export MODEL_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel
export TOKENIZER_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel
export DIST_INIT_ADDR=10.16.1.26:20000
export SGLANG_APPLY_CONFIG_BACKUP=none
export SGLANG_TOPK_TRANSFORM_512_TORCH=false
export SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=false

bash /root/sglang/hygon_tmp/start_pro_multinode_node.sh 0
```

如果希望命令在容器 bash 内后台运行，可以使用 `nohup`：
```bash
nohup bash /root/sglang/hygon_tmp/start_pro_multinode_node.sh 1 > /root/sglang/hygon_tmp/start_rank1.out 2>&1 &

nohup bash /root/sglang/hygon_tmp/start_pro_multinode_node.sh 0 > /root/sglang/hygon_tmp/start_rank0.out 2>&1 &
```

### Pro blockwise启动方式

在 `10.16.1.28` 启动 rank1：
```bash
docker exec \
  -e MODEL_PATH=/root/module1/DeepSeek-V4-Pro-FP8 \
  -e TOKENIZER_PATH=/root/module1/DeepSeek-V4-Pro-FP8 \
  -e DIST_INIT_ADDR=10.16.1.26:20000 \
  -e SGLANG_APPLY_CONFIG_BACKUP=large \
  -e SGLANG_TOPK_TRANSFORM_512_TORCH=false \
  -e SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=false \
  -d wangaq-deepseekv4 \
  /root/sglang/hygon_tmp/start_pro_multinode_node.sh 1

```

在 `10.16.1.26` 启动 rank0：
```bash
docker exec \
  -e MODEL_PATH=/root/module1/DeepSeek-V4-Pro-FP8 \
  -e TOKENIZER_PATH=/root/module1/DeepSeek-V4-Pro-FP8 \
  -e DIST_INIT_ADDR=10.16.1.26:20000 \
  -e SGLANG_APPLY_CONFIG_BACKUP=large \
  -e SGLANG_TOPK_TRANSFORM_512_TORCH=false \
  -e SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=false \
  -d wangaq-deepseekv4 \
  /root/sglang/hygon_tmp/start_pro_multinode_node.sh 0

```

### Pro Channel 启动方式

当前 Pro Channel 的 `config.json` 和 tokenizer 已更新，启动时使用 Channel 模型目录自己的 tokenizer，并保持 `SGLANG_APPLY_CONFIG_BACKUP=none`。当前代码已经按 compressed-tensors 的 channelwise scale 路径加载权重，不需要再设置 `SGLANG_DSV4_CHANNEL_FP8_SCALE`。

在 `10.16.1.28` 启动 rank1：
```bash
docker exec \
  -e MODEL_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel \
  -e TOKENIZER_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel \
  -e DIST_INIT_ADDR=10.16.1.26:20000 \
  -e SGLANG_APPLY_CONFIG_BACKUP=none \
  -e SGLANG_TOPK_TRANSFORM_512_TORCH=false \
  -e SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=false \
  -d wangaq-deepseekv4 \
  /root/sglang/hygon_tmp/start_pro_multinode_node.sh 1
```

在 `10.16.1.26` 启动 rank0：
```bash
docker exec \
  -e MODEL_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel \
  -e TOKENIZER_PATH=/root/module1/DeepSeek-V4-Pro-FP8-Channel \
  -e DIST_INIT_ADDR=10.16.1.26:20000 \
  -e SGLANG_APPLY_CONFIG_BACKUP=none \
  -e SGLANG_TOPK_TRANSFORM_512_TORCH=false \
  -e SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK=false \
  -d wangaq-deepseekv4 \
  /root/sglang/hygon_tmp/start_pro_multinode_node.sh 0
```

### 启动状态检查
```bash
grep -nE 'Load weight|Memory pool end|Capture cuda graph|The server is fired up|Traceback|Exception|Killed|VMFault|memory leak|ValueError' \
  /root/sglang/hygon_tmp/pro_multinode_rank0.log \
  /root/sglang/hygon_tmp/pro_multinode_rank1.log | tail -n 300

```

正常启动顺序：
```plaintext
Load weight begin
Load weight end
Memory pool end
Capture cuda graph begin
Capture cuda graph end
The server is fired up and ready to roll!

```

### 简单请求
```bash
PORT="${PORT:-30000}"
BASE_URL="${BASE_URL:-http://127.0.0.1:$PORT}"

curl "${BASE_URL}/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d @- <<JSON
{
  "model": "default",
  "messages": [
    {
      "role": "user",
      "content": "甲乙两班共有学生98人，甲班比乙班多6人，求两班各有多少人？"
    }
  ],
  "max_tokens": 256,
  "temperature": 0
}
JSON

```

### 数据集验证
```bash
cd /root/sglang/sglang-gitlab/python

PYTHONPATH=/root/sglang/sglang-gitlab/python \
python3 -m sglang.test.few_shot_gsm8k \
  --num-questions 200 \
  --port 30000

```

当前验证结果：
```plaintext
Pro 非 Channel cudagraph，GSM8K 200 题：
Accuracy: 0.970
Invalid: 0.000

Pro Channel cudagraph，GSM8K 200 题：
Accuracy: 0.985
Invalid: 0.000
Latency: 282.293 s
Output throughput: 63.168 token/s

```

### 注意事项

- Pro 模型必须使用两机，每机 8 卡，否则容易 OOM。
- 当前验证的 cudagraph 路径需要关闭 `SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK`。
- 当前 Pro Channel 代码不需要设置 `SGLANG_DSV4_CHANNEL_FP8_SCALE`；该变量是旧调试路径遗留项。
- Pro Channel tokenizer 使用 Channel 模型目录，即 `/root/module1/DeepSeek-V4-Pro-FP8-Channel`。
- Channel FP8 scale 已经是 channelwise 形状，不能再用 per-tensor scale 的方式展开，否则会导致精度异常。

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/CUDA Graph|CUDA Graph]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/deepseek-v4|deepseek-v4]]（model）
- [[飞书知识图谱/实体/Deepseek-V4|Deepseek-V4]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4|DeepSeek-V4]]（model）
- [[飞书知识图谱/实体/deepseek-v4-dev|deepseek-v4-dev]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4-Flash-Channel-F8-w8a8|DeepSeek-V4-Flash-Channel-F8-w8a8]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4-Flash-FP8|DeepSeek-V4-Flash-FP8]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4-Flash-FP8-Channel|DeepSeek-V4-Flash-FP8-Channel]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4-Flash-INT8-Channel|DeepSeek-V4-Flash-INT8-Channel]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4-Pro-Channel-FP8-w8a8|DeepSeek-V4-Pro-Channel-FP8-w8a8]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4-Pro-FP8|DeepSeek-V4-Pro-FP8]]（model）
- [[飞书知识图谱/实体/DeepSeek-V4-Pro-FP8-Channel|DeepSeek-V4-Pro-FP8-Channel]]（model）
- [[飞书知识图谱/实体/deepseekv4-v2.image.tar.gz|deepseekv4-v2.image.tar.gz]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OOM|OOM]]（issue）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/RDMA|RDMA]]（concept）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）

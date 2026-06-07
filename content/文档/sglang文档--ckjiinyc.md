---
title: sglang文档
tags:
  - AI
  - LLM
  - Benchmark
model:
  - GLM-5-W8A8
  - glm-int8-tp16-dp16-ep16
  - glm-int8-tp8cp8
framework:
  - Mooncake
  - SGLang
  - vLLM
issue_type: []
source: feishu
feishu_token: Dz6ndtV3poID3Ox82dhckjiinyc
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/Dz6ndtV3poID3Ox82dhckjiinyc
last_synced_at: 2026-06-07T17:07:53.393Z
---

# sglang文档

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/Dz6ndtV3poID3Ox82dhckjiinyc)


<lark-table rows="8" cols="6" column-widths="226,454,465,220,100,100">

  <lark-tr>
    <lark-td>
      切分方式
    </lark-td>
    <lark-td>
      测试命令
    </lark-td>
    <lark-td>
      环境变量
    </lark-td>
    <lark-td>
      备注
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      tp
    </lark-td>
    <lark-td>
      sglang serve \
        --model-path /model/vllm-w8a8-models/GLM-5-W8A8 ${option} \
        --trust-remote-code \
        --host $HOST \
        --port 30000 \
        --dist-init-addr $HOST:5000 \
        --tp-size $tp \
        --kv-cache-dtype fp8_e4m3 \
        --dtype bfloat16 \
        --mem-fraction-static 0.9 \
        --page-size 64 \
        --nsa-prefill-backend flashmla_auto \
        --nsa-decode-backend flashmla_kv \
        --quantization slimquant_marlin
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      pp
    </lark-td>
    <lark-td>
      option+=" --pp-size  $pp"
    </lark-td>
    <lark-td>
      export SGLANG_PP_LAYER_PARTITION="x,x,x,x,x,x,x,x"
    </lark-td>
    <lark-td>
      根据模型的moe专家数，一般是均分；或者按照专家和卡数分配均匀
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      cp
    </lark-td>
    <lark-td>
      option+="--attn-cp-size $cp --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split"
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      dp-tpmoe
    </lark-td>
    <lark-td>
      option+="  --dp-size  $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head"
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      deepep
    </lark-td>
    <lark-td>
      option+=" --ep-size $ep --moe-a2a-backend deepep --deepep-mode low_latency --moe-dense-tp-size 1 --enable-dp-lm-head"
    </lark-td>
    <lark-td>
      export ROCSHMEM_DISABLE_HDP_FLUSH=1
      export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
      export ROCSHMEM_HEAP_SIZE=3173741824
      export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
      export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      mtp3
    </lark-td>
    <lark-td>
      option+=" --speculative-algorithm EAGLE --speculative-num-steps 3   --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      hicache
    </lark-td>
    <lark-td>
      option+="  --enable-hierarchical-cache   --hicache-ratio 1   --hicache-mem-layout page_first   --hicache-io-backend kernel  --hicache-write-policy write_through  --hicache-storage-backend mooncake   --hicache-storage-prefetch-policy best_effort "
    </lark-td>
    <lark-td>
      export MOONCAKE_TE_META_DATA_SERVER="http://xxx:8080/metadata"
      export MOONCAKE_GLOBAL_SEGMENT_SIZE="64GB"
      export MOONCAKE_PROTOCOL="tcp"
      export MOONCAKE_DEVICE="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
      export MOONCAKE_MASTER=xxx:50051
    </lark-td>
    <lark-td>
      注意：
      MHA、GQA类注意力需要设置export SGLANG_KV_LAYOUT_DCU_FA=1、--hicache-mem-layout dcu_layout
      pd分离时
      在p/d的主节点起服务前需要先执行
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
</lark-table>

PD分离额外加参数

<lark-table rows="4" cols="5" column-widths="100,430,531,374,100">

  <lark-tr>
    <lark-td>
      角色
    </lark-td>
    <lark-td>
      测试命令
    </lark-td>
    <lark-td>
      环境变量
    </lark-td>
    <lark-td>
      备注
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      P节点
    </lark-td>
    <lark-td>
      option+="    --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9,mlx5_6,mlx5_7 
        --disaggregation-mode prefill "
    </lark-td>
    <lark-td>
      export MC_IB_GID_INDEX=0
      export ROCSHMEM_IB_GID_INDEX=0
      export MC_ENABLE_DEST_DEVICE_AFFINITY=1
      export SGLANG_HOST_IP=$(hostname -I 2>/dev/null | awk '{print $1}')
    </lark-td>
    <lark-td>
      PD分离时，P需要额外加参数：
      option+="    --enable-hierarchical-cache "
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      D节点
    </lark-td>
    <lark-td>
      option+="    --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9,mlx5_6,mlx5_7 
        --disaggregation-mode decode "
    </lark-td>
    <lark-td>
      export MC_IB_GID_INDEX=0
      export ROCSHMEM_IB_GID_INDEX=0
      export MC_ENABLE_DEST_DEVICE_AFFINITY=1
      export SGLANG_HOST_IP=$(hostname -I 2>/dev/null | awk '{print $1}')
    </lark-td>
    <lark-td>
      D节点需要加下：
      option+="  --disaggregation-decode-enable-offload-kvcache "
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      router
    </lark-td>
    <lark-td>
      python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://10.16.1.20:30000 --decode http://10.16.1.16:30000 --policy round_robin --port 30020
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
</lark-table>

```sql
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_ENABLE_SPEC_V2=1
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
#export SGLANG_USE_FUSED_RMS_QUANT=1
#export SGLANG_USE_RMS_QUANT_PATH=1
#export SGLANG_USE_FUSED_SILU_MUL_QUANT=1
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
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
#export ROCSHMEM_TOPO_FILE_FORCE=/home/topo.config
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_HEALTH_CHECK_TIMEOUT=600
#export SGLANG_ENABLE_HEALTH_ENDPOINT_GENERATION=0
#export MC_TOPO_FILE_FORCE=/home/mc_topo.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=ens61f1np1
export GLOO_SOCKET_IFNAME=ens61f1np1
#export ROCBLAS_TENSILE_LIBPATH=/mnt11/task/sgl/40-16/pd/yunyi/library_gpu6_glm5_int8
export ROCBLAS_TENSILE_LIBPATH=/mnt11/nmz/sgl/auto_select_tools/optimization_configs/new/config/library_gpu6
export HIPBLASLT_TUNING_OVERRIDE_FILE=hipblaslt.config
#export SGLANG_USE_MODELSCOPE=1
export W8A8_SUPPORT_METHODS=3
export MC_IB_GID_INDEX=0
export ROCSHMEM_IB_GID_INDEX=0
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
export SGLANG_HOST_IP=$(hostname -I 2>/dev/null | awk '{print $1}')
#export ROCBLAS_LAYER=3
export ROCSHMEM_TOPO_FILE_FORCE=/mnt11/lijian/hg_deep_topo.config


master_ip=$1
#hicache
export MOONCAKE_TE_META_DATA_SERVER="http://$master_ip:8080/metadata"
export MOONCAKE_GLOBAL_SEGMENT_SIZE="64GB"
export MOONCAKE_PROTOCOL="tcp"
export MOONCAKE_DEVICE="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
export MOONCAKE_MASTER="$master_ip:50051"


time=$(date "+%m%d-%H%M")
logpath="dserver/glm-int8-tp16-dp16-ep16-$(hostname)"
logfile="${logpath}/glm5_int8_${time}.log"
mkdir -p "${logpath}"
HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
  HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi


sglang serve \
  --model-path /model/vllm-w8a8-models/GLM-5-W8A8 \
  --trust-remote-code \
  --host $HOST \
  --port 30000 \
  --dist-init-addr $master_ip:5000  \
  --nnodes 2 \
  --node-rank $2 \
  --tp-size 16 \
  --dp-size 16 \
  --ep-size 16 \
  --moe-dense-tp-size 1 \
  --enable-dp-attention \
  --moe-a2a-backend deepep \
  --deepep-mode low_latency \
  --enable-dp-lm-head \
  --nsa-prefill-backend flashmla_auto \
  --nsa-decode-backend flashmla_kv \
  --context-length 131072 \
  --dtype bfloat16 \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --page-size 64 \
  --kv-cache-dtype fp8_e4m3 \
  --mem-fraction-static 0.8 \
  --chunked-prefill-size -1 \
  --quantization slimquant_marlin \
  --cuda-graph-max-bs 32 \
  --max-running-requests 512 \
  --speculative-algorithm EAGLE \
  --speculative-num-steps 3 \
  --speculative-eagle-topk 1 \
  --speculative-num-draft-tokens 4 \
  --disaggregation-mode decode \
  --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9 \
  --hicache-ratio 1 \
  --hicache-mem-layout page_first \
  --hicache-io-backend kernel \
  --hicache-write-policy write_through \
  --hicache-storage-backend mooncake \
  --hicache-storage-prefetch-policy best_effort \
  --disaggregation-decode-enable-offload-kvcache  2>&1 | tee "${logfile}"

```

```bash
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_HEALTH_CHECK_TIMEOUT=600
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/profile
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
export SGLANG_USE_MODELSCOPE=1
export HIP_GRAPH_ACCUMULATE_DISPATCH=0
export HIP_H2D_DISABLE_COPY_BUFFER=0
export HIP_D2H_DISABLE_COPY_BUFFER=0
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768
export HIP_D2H_DIRECT_COPY_THRESHOLD=512
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512
export HIP_GRAPH_USE_CMD_CACHE=0
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
#export MC_TOPO_FILE_FORCE=/home/mc_topo_400g.config
#export MC_TOPO_FILE_FORCE=/
#export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export SGLANG_USE_LIGHTOP=1
export SGLANG_ROCM_USE_AITER_MOE=0
export W8A8_SUPPORT_METHODS=3
export MC_IB_GID_INDEX=0
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
export SGLANG_HOST_IP=$(hostname -I 2>/dev/null | awk '{print $1}')
#export SGLANG_ENABLE_HEALTH_ENDPOINT_GENERATION=0

#hicache
export SGLANG_KV_LAYOUT_DCU_FA=1
export MOONCAKE_TE_META_DATA_SERVER="http://$SGLANG_HOST_IP:8080/metadata"
export MOONCAKE_GLOBAL_SEGMENT_SIZE="64GB"
export MOONCAKE_PROTOCOL="tcp"
export MOONCAKE_DEVICE="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
export MOONCAKE_MASTER="$SGLANG_HOST_IP:50051"

#export GPU_FLUSH_ON_EXECUTION=1 #调试用

time=$(date "+%m%d-%H%M")
logpath="pserver/glm-int8-tp8cp8-$(hostname)"
logfile="${logpath}/glm5_int8_${time}.log"
mkdir -p "${logpath}"

HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
  HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi


option="--attn-cp-size 8 --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split"

sglang serve \
  --model-path /model/vllm-w8a8-models/GLM-5-W8A8 ${option} \
  --trust-remote-code \
  --host $HOST \
  --port 30000 \
  --dist-init-addr $HOST:5000 \
  --tp-size 8 \
  --pp-size 1 \
  --kv-cache-dtype fp8_e4m3 \
  --dtype bfloat16 \
  --mem-fraction-static 0.9 \
  --page-size 64 \
  --nsa-prefill-backend flashmla_auto \
  --nsa-decode-backend flashmla_kv \
  --quantization slimquant_marlin \
  --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_8,mlx5_9,mlx5_6,mlx5_7 \
  --disaggregation-mode prefill \
  --enable-hierarchical-cache \
  --hicache-ratio 1 \
  --hicache-mem-layout page_first \
  --hicache-io-backend kernel \
  --hicache-write-policy write_through \
  --hicache-storage-backend mooncake \
  --hicache-storage-prefetch-policy best_effort 2>&1 | tee "${logfile}"

```

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/glm-int8-tp16-dp16-ep16|glm-int8-tp16-dp16-ep16]]（model）
- [[飞书知识图谱/实体/glm-int8-tp8cp8|glm-int8-tp8cp8]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

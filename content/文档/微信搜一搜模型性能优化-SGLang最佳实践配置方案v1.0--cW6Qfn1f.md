---
title: 微信搜一搜模型性能优化-SGLang最佳实践配置方案v1.0
tags:
  - AI
  - LLM
  - Benchmark
model:
  - Qwen3-Next-80B-A3B-Instruct
  - Qwen3-Next-80B-A3B-Instruct.log
  - Qwen3.5-27B
  - Qwen3.5-27B.log
  - Qwen3.5-35B-A3B
  - Qwen3.5-35B-A3B-CHANNEL-FP8
  - Qwen3.5-35B-A3B-CHANNEL-FP8.log
  - Qwen3.5-35B-A3B-FP8
  - Qwen3.5-35B-A3B-FP8.log
  - Qwen3.5-35B-A3B.log
framework:
  - SGLang
issue_type: []
source: feishu
feishu_token: ZfxrwHQHVi06gOkKcoHcW6Qfn1f
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/ZfxrwHQHVi06gOkKcoHcW6Qfn1f
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 微信搜一搜模型性能优化-SGLang最佳实践配置方案v1.0

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/ZfxrwHQHVi06gOkKcoHcW6Qfn1f)

# QwQ-32B

## 启动服务
```bash {wrap}
option="--numa-node 3 1 1 0 7 5 5 4"
# option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy extra_buffer" 
option+=" --chunked-prefill-size -1"
option+=" --cuda-graph-max-bs 256"
host_ip=$(hostname -I | awk '{print $1}')
# ==========================================
model_path="/model/QwQ-32B"
# 使用融合算子
export SGLANG_USE_FUSED_SPLIT_QKV_RMS_ROTARY_EMBEDDING=1
nohup python3 -u -m sglang.launch_server  ${option} \
  --model-path  $model_path \
  --host $host_ip \
  --port 30003 \
  --tp-size 8  --pp-size 1 \
  --attention-backend fa3 \
  --page-size 64 --pp-size 1  \
  --mem-fraction-static 0.8 \
  --kv-cache-dtype fp8_e4m3 \
 >./nohup/QwQ-32B.log 2>&1 & 
```

## 性能测试
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

MODEL_PATH="/model/QwQ-32B"
SERVER_IP=$(hostname -I | awk '{print $1}')
SERVER_PORT="30003"
LOG_DIR="./benchserveing_test/"
mkdir -p "${LOG_DIR}"
LOG_FILE="${LOG_DIR}/test.log"
python3 -m sglang.bench_serving \
  --backend sglang \
  --model "${MODEL_PATH}" \
  --host "${SERVER_IP}" \
  --port "${SERVER_PORT}" \
  --dataset-name random-ids \
  --num-prompts 100 \
  --max-concurrency 20 \
  --random-input-len 10730 \
  --random-output-len 410 \
  --random-range-ratio 1 \
  2>&1 | tee "${LOG_FILE}"
```

# Qwen3-Next-80B-A3B-Instruct

## 启动服务
```bash {wrap}
option="--numa-node 3 1 1 0 7 5 5 4"
# option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy extra_buffer" 
option+=" --chunked-prefill-size -1"
option+=" --cuda-graph-max-bs 256"
host_ip=$(hostname -I | awk '{print $1}')
# ==========================================
model_path=/model/Qwen3-Next-80B-A3B-Instruct
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FUSED_TOPK_SOFTMAX=1
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1
export SGLANG_USE_MARLIN_W16A16_MOE=1
nohup python3 -u -m sglang.launch_server  ${option} \
  --model-path  $model_path \
  --host $host_ip \
  --port 30003 \
  --tp-size 8  --pp-size 1 \
  --attention-backend fa3 \
  --page-size 64 --pp-size 1  \
  --mem-fraction-static 0.8 \
  --kv-cache-dtype fp8_e4m3 \
 >./nohup/Qwen3-Next-80B-A3B-Instruct.log 2>&1 & 
```

## 性能测试
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

MODEL_PATH=/model/Qwen3-Next-80B-A3B-Instruct
SERVER_IP=$(hostname -I | awk '{print $1}')
SERVER_PORT="30003"
LOG_DIR="./benchserveing_test/"
mkdir -p "${LOG_DIR}"
LOG_FILE="${LOG_DIR}/test.log"
python3 -m sglang.bench_serving \
  --backend sglang \
  --model "${MODEL_PATH}" \
  --host "${SERVER_IP}" \
  --port "${SERVER_PORT}" \
  --dataset-name random-ids \
  --num-prompts 100 \
  --max-concurrency 20 \
  --random-input-len 10718 \
  --random-output-len 315 \
  --random-range-ratio 1 \
  2>&1 | tee "${LOG_FILE}"
```

# Qwen3.5-35B-A3B

## 启动服务
```bash {wrap}
option="--numa-node 3 1 1 0 7 5 5 4"
# option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy extra_buffer" 
option+=" --chunked-prefill-size -1"
option+=" --cuda-graph-max-bs 256"
host_ip=$(hostname -I | awk '{print $1}')
# ==========================================
model_path=/model/Qwen3.5-35B-A3B
# 使用融合算子
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FUSED_TOPK_SOFTMAX=1
export SGLANG_ROCM_USE_AITER_MOE=False
export SGLANG_USE_CAUSAL_CONV1D=1
export SGLANG_USE_CUDA_IPC_TRANSPORT=1
export SGLANG_USE_AITER_LINEAR_ATTN=1
nohup python3 -u -m sglang.launch_server  ${option} \
  --model-path  $model_path \
  --host $host_ip \
  --port 30003 \
  --tp-size 8  --pp-size 1 \
  --attention-backend fa3 \
  --page-size 64 --pp-size 1  \
  --mem-fraction-static 0.8 \
  --kv-cache-dtype fp8_e4m3 \
 >./nohup/Qwen3.5-35B-A3B.log 2>&1 & 
```

## 性能测试
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

MODEL_PATH=/model/Qwen3.5-35B-A3B
SERVER_IP=$(hostname -I | awk '{print $1}')
SERVER_PORT="30003"
LOG_DIR="./benchserveing_test/"
mkdir -p "${LOG_DIR}"
LOG_FILE="${LOG_DIR}/test.log"
python3 -m sglang.bench_serving \
  --backend sglang \
  --model "${MODEL_PATH}" \
  --host "${SERVER_IP}" \
  --port "${SERVER_PORT}" \
  --dataset-name random-ids \
  --num-prompts 100 \
  --max-concurrency 20 \
  --random-input-len 10730 \
  --random-output-len 410 \
  --random-range-ratio 1 \
  2>&1 | tee "${LOG_FILE}"
```

# Qwen3.5-35B-A3B-FP8(blockwise)

## 启动服务
```bash {wrap}
option="--numa-node 3 1 1 0 7 5 5 4"
# option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy extra_buffer" 
option+=" --chunked-prefill-size -1"
option+=" --cuda-graph-max-bs 256"
host_ip=$(hostname -I | awk '{print $1}')
# ==========================================
model_path=/model/Qwen3.5-35B-A3B-FP8
# 使用融合算子
export SGLANG_USE_FUSED_TOPK_SOFTMAX=1
export SGLANG_USE_FP8_W8A8_MOE=1
export SGLANG_USE_FUSED_TOPK_SOFTMAX=1
export SGLANG_ROCM_USE_AITER_MOE=False
export SGLANG_USE_CAUSAL_CONV1D=1
nohup python3 -u -m sglang.launch_server  ${option} \
  --model-path  $model_path \
  --host $host_ip \
  --port 30003 \
  --tp-size 4  --pp-size 1 \
  --attention-backend fa3 \
  --page-size 64 --pp-size 1  \
  --mem-fraction-static 0.8 \
  --kv-cache-dtype fp8_e4m3 \
 >./nohup/Qwen3.5-35B-A3B-FP8.log 2>&1 & 
```

<quote-container>

Qwen3.5-35B-A3B-FP8启动服务时，--tp-size建议配置为4，否则会出现报错。
</quote-container>

## 性能测试
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

MODEL_PATH=/model/Qwen3.5-35B-A3B-FP8
SERVER_IP=$(hostname -I | awk '{print $1}')
SERVER_PORT="30003"
LOG_DIR="./benchserveing_test/"
mkdir -p "${LOG_DIR}"
LOG_FILE="${LOG_DIR}/test.log"
python3 -m sglang.bench_serving \
  --backend sglang \
  --model "${MODEL_PATH}" \
  --host "${SERVER_IP}" \
  --port "${SERVER_PORT}" \
  --dataset-name random-ids \
  --num-prompts 100 \
  --max-concurrency 20 \
  --random-input-len 10730 \
  --random-output-len 410 \
  --random-range-ratio 1 \
  2>&1 | tee "${LOG_FILE}"
```

# Qwen3.5-35B-A3B-CHANNEL-FP8(channelwise)

## 启动服务
```bash {wrap}
option=" --numa-node 3 1 1 0 7 5 5 4"
option=" --chunked-prefill-size -1"
option+=" --mamba-scheduler-strategy extra_buffer"
option+=" --cuda-graph-max-bs 256"
# =================================================
model_path=/model/DL_DATA/llm-models/Qwen3.5-35B-A3B-CHANNEL-FP8
option+=" --kv-cache-dtype fp8_e4m3"
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FP8_W8A8_MOE=1
export SGLANG_USE_FUSED_TOPK_SOFTMAX=1
export SGLANG_ROCM_USE_AITER_MOE=False
export SGLANG_USE_CAUSAL_CONV1D=1
export SGLANG_USE_CUDA_IPC_TRANSPORT=1
export SGLANG_USE_AITER_LINEAR_ATTN=1
export ROCBLAS_TENSILE_LIBPATH=/sgl0510rc0/1test_script/library_gpu6
export SGLANG_ENABLE_SPEC_V2=1
export HIP_VISIBLE_DEVICES=0,1
nohup  python3 -u -m sglang.launch_server  ${option} \
  --model-path  $model_path \
  --host 0.0.0.0 \
  --port 30004 \
  --mm-attention-backend fa3 \
  --speculative-algorithm EAGLE \
  --enable-piecewise-cuda-graph \
  --speculative-num-steps 3 \
  --speculative-eagle-topk 1 \
  --speculative-num-draft-tokens 4 \
  --tp-size 2 --pp-size 1 \
  --attention-backend fa3 \
  --page-size 64 --pp-size 1  \
  --mem-fraction-static 0.7 \
>./nohup/Qwen3.5-35B-A3B-CHANNEL-FP8.log 2>&1 &

```

Qwen3.5-35B-A3B-CHANNEL-FP8权重获取请联系<mention-user id="ou_92b88ddfe003133bdf4f063d1d210845"/>

## 性能测试
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

MODEL_PATH=/model/Qwen3.5-35B-A3B-CHANNEL-FP8
SERVER_IP=$(hostname -I | awk '{print $1}')
SERVER_PORT="30003"
LOG_DIR="./benchserveing_test/"
mkdir -p "${LOG_DIR}"
LOG_FILE="${LOG_DIR}/test.log"
python3 -m sglang.bench_serving \
  --backend sglang \
  --model "${MODEL_PATH}" \
  --host "${SERVER_IP}" \
  --port "${SERVER_PORT}" \
  --dataset-name random-ids \
  --num-prompts 100 \
  --max-concurrency 20 \
  --random-input-len 10730 \
  --random-output-len 410 \
  --random-range-ratio 1 \
  2>&1 | tee "${LOG_FILE}"
```

# Qwen3.5-27B

## 启动服务
```bash {wrap}
option="--numa-node 3 1 1 0 7 5 5 4"
# option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy extra_buffer" 
option+=" --chunked-prefill-size -1"
option+=" --cuda-graph-max-bs 256"
host_ip=$(hostname -I | awk '{print $1}')
# ==========================================
model_path=/model/Qwen3.5-27B
# 使用融合算子
export SGLANG_USE_FUSED_TOPK_SOFTMAX=1
export SGLANG_USE_CAUSAL_CONV1D=1
export SGLANG_USE_AITER_LINEAR_ATTN=1
nohup python3 -u -m sglang.launch_server  ${option} \
  --model-path  $model_path \
  --host $host_ip \
  --port 30003 \
  --tp-size 8  --pp-size 1 \
  --attention-backend fa3 \
  --page-size 64 --pp-size 1  \
  --mem-fraction-static 0.8 \
  --kv-cache-dtype fp8_e4m3 \
 >./nohup/Qwen3.5-27B.log 2>&1 & 
```

## 性能测试
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

MODEL_PATH=/model/Qwen3.5-27B
SERVER_IP=$(hostname -I | awk '{print $1}')
SERVER_PORT="30003"
LOG_DIR="./benchserveing_test/"
mkdir -p "${LOG_DIR}"
LOG_FILE="${LOG_DIR}/test.log"
python3 -m sglang.bench_serving \
  --backend sglang \
  --model "${MODEL_PATH}" \
  --host "${SERVER_IP}" \
  --port "${SERVER_PORT}" \
  --dataset-name random-ids \
  --num-prompts 100 \
  --max-concurrency 20 \
  --random-input-len 10730 \
  --random-output-len 410 \
  --random-range-ratio 1 \
  2>&1 | tee "${LOG_FILE}"
```

# Seed-OSS-36B-Instruct
```bash {wrap}
option="--numa-node 3 1 1 0 7 5 5 4"
# option+=" --disable-radix-cache "
# option+=" --mamba-scheduler-strategy extra_buffer" 
option+=" --chunked-prefill-size -1"
option+=" --cuda-graph-max-bs 256"
host_ip=$(hostname -I | awk '{print $1}')
# ==========================================
model_path=/model/Seed-OSS-36B-Instruct
option+=" --kv-cache-dtype fp8_e4m3"
nohup python3 -u -m sglang.launch_server  ${option} \
  --model-path  $model_path \
  --host $host_ip \
  --port 30003 \
  --tp-size 8  --pp-size 1 \
  --attention-backend fa3 \
  --page-size 64 --pp-size 1  \
  --mem-fraction-static 0.8 \
  --kv-cache-dtype fp8_e4m3 \
 >./nohup/Seed-OSS-36B-Instruct.log 2>&1 & 
```

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/Qwen3-Next-80B-A3B-Instruct|Qwen3-Next-80B-A3B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen3-Next-80B-A3B-Instruct.log|Qwen3-Next-80B-A3B-Instruct.log]]（model）
- [[飞书知识图谱/实体/Qwen3.5-27B|Qwen3.5-27B]]（model）
- [[飞书知识图谱/实体/Qwen3.5-27B.log|Qwen3.5-27B.log]]（model）
- [[飞书知识图谱/实体/Qwen3.5-35B-A3B|Qwen3.5-35B-A3B]]（model）
- [[飞书知识图谱/实体/Qwen3.5-35B-A3B-CHANNEL-FP8|Qwen3.5-35B-A3B-CHANNEL-FP8]]（model）
- [[飞书知识图谱/实体/Qwen3.5-35B-A3B-CHANNEL-FP8.log|Qwen3.5-35B-A3B-CHANNEL-FP8.log]]（model）
- [[飞书知识图谱/实体/Qwen3.5-35B-A3B-FP8|Qwen3.5-35B-A3B-FP8]]（model）
- [[飞书知识图谱/实体/Qwen3.5-35B-A3B-FP8.log|Qwen3.5-35B-A3B-FP8.log]]（model）
- [[飞书知识图谱/实体/Qwen3.5-35B-A3B.log|Qwen3.5-35B-A3B.log]]（model）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）

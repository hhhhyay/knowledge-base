---
title: H20的PD分离测试
tags:
  - AI
  - LLM
  - Benchmark
model:
  - GLM-5
  - GLM-5-FP8
  - GLM5-Channelwise-FP8-quantized
  - Kimi-K2.5
  - minimax-append-think
  - minimax-m2
  - MiniMax-M2.5
  - minimax-test.py
  - Qwen-3.5-397B
  - Qwen3.5-122B
  - Qwen3.5-122B-A10B-FP8
  - Qwen3.5-397B-A17B-FP8
framework:
  - SGLang
issue_type: []
source: feishu
feishu_token: RE69wHKweiF1oQkoA5hcDaVpnSb
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/RE69wHKweiF1oQkoA5hcDaVpnSb
last_synced_at: 2026-06-07T17:07:53.393Z
---

# H20的PD分离测试

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/RE69wHKweiF1oQkoA5hcDaVpnSb)

测试需求

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

**H20 PD分离测试配置（用SGLang跑，优化选项比如TBO/MTP打开）** **：**

PD 配平方式：满足 SLA 条件下，P端 qps = D端 qps

**GLM-5**

Prefix cache=0, 3P4D, 2k/1k，SLA TTFT 5s, TPOT 50ms

Prefix cache=0, 3P4D, 8k/1k, SLA TTFT 5s, TPOT 50ms

~~Prefix cache=80%,~~ PD需要配平(D大概率用EP32), 50k/200, SLA TTFT 5s, TPOT 50ms

测试方法可联系

**Kimi-K2.5**

4k/1k, SLA TTFT 5s, TPOT 100ms/25ms

P端用 PCP8 / TP8

D端用 如果 DP16EP16 / DP32EP32 / DP64EP64 不通就用 TP8

测试方法可联系

**MiniMax-M2.5**

4k/1k, SLA TTFT 5s, TPOT 50ms

<text bgcolor="yellow">50k/256, SLA TTFT 8s，TPOT25ms</text>

模拟P和D，都用TP8，人肉配平

测试方法可联系 

**Qwen-3.5-397B**

4k/1k, SLA TTFT 5s, TPOT 50ms

P端用 PCP8 / TP8

D端用 如果 DP16EP16 / DP32EP32 / DP64EP64 不通就用 TP8

测试方法可联系
</callout>

测试过程数据[[飞书知识图谱/资源/H20的PD分离测试过程数据--cejufnSt|H20的PD分离测试过程数据]]

镜像路径：/public/home/public_user/sglangv0.5.10rc0.tar.gz

创建容器：
```bash {wrap}
docker run -it --network=host --name=gbj-sgl --privileged --device=/dev/kfd --gpus all --device=/dev/dri --ipc=host --shm-size=128G  --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -u root --ulimit stack=-1:-1 --ulimit memlock=-1:-1 --entrypoint bash -v /public/opendas/DL_DATA/llm-models:/model -v `pwd`:/mnt lmsysorg/sglang:latest
```

【补充p95输出】/public/home/public_user/gbj/bench_serving.py 

docker  cp /public/home/public_user/gbj/bench_serving.py /sgl-workspace/sglang/python/sglang

# PD分离文档【[[飞书知识图谱/资源/PD 分离.md--ckWQSnfd|PD 分离.md]]-dcu】

# GLM-5

镜像：docker pull lmsysorg/sglang:v0.5.10rc0
```sql {wrap}
docker run -it \
--network=host \
--gpus all \
--name xxxx \
--privileged \
--device=/dev/kfd --device=/dev/dri \
--shm-size=500G \
--group-add video \
--cap-add=SYS_PTRACE \
--security-opt seccomp=unconfined \
-u root --ulimit stack=-1:-1 \
--ulimit memlock=-1:-1 \
--entrypoint bash \
-v /xxx:/mnt \
lmsysorg/sglang:v0.5.10rc0
```

## pd分离：

<callout emoji="skull" background-color="light-orange" border-color="light-orange">

执行流程：先执行pd各自客户端脚本，等待服务起完；

然后在执行代理脚本；

最后在代理脚本所在节点执行客户端命令测试。
</callout>

### P【tp8pp2cp8】-d【tp16ep16dp16】【输入输出：8k/1k、2K/1k】

P【tp8pp2cp8】
```bash {wrap}
export CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NVSHMEM_DISABLE_MNNVL=1
export NVSHMEM_SYMMETRIC_SIZE=10737418240
export NVSHMEM_ENABLE_NIC_PE_MAPPING=1
export NVSHMEM_HCA_PE_MAPPING=mlx5_0:1:2,mlx5_1:1:2,mlx5_2:1:2,mlx5_4:1:2
export SGLANG_DEEPEE_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
export NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME=ibp39s0
model_path=/module/GLM-5-FP8
model=${model_path##*/}
tp=8
pp=2
dp=1
ep=1
nodes=2
rank=$1
#master_ip=
# host_ip=$(hostname -I | awk '{print $1}')
host_ip="172.16.106.73" # h20-4
# host_ip="172.16.106.54"
master_ip=$host_ip
max_model_len=54272
gpu_mem=0.8
time=$(date "+%m%d-%H%M")
mode="eager"
logpath="server/$model-tp$tp-dp$dp-ep$ep-pp$pp-$host_ip-p-$time"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --page-size 64 "
# option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
# option+=" --attention-backend fa3"
option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
# option+=" --pp-max-micro-batch-size 2"
# option+=" --cuda-graph-max-bs 128 "
option+=" --context-length $max_model_len"
# option+=" --disable-cuda-graph  " #--skip-server-warmup
# option+=" --quantization slimquant_marlin  "
option+=" --disaggregation-ib-device mlx5_0"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode prefill "
option+=" --kv-cache-dtype fp8_e4m3 "
option+=" --attn-cp-size 8 "
# option+=" --max-running-requests 128 "
option+=" --skip-server-warmup "
option+=" --moe-dense-tp-size 1 "

python -m sglang.launch_server \
--model $model_path  ${option} \
--trust-remote-code \
--dtype bfloat16 --mem-fraction-static $gpu_mem \
--tp-size $tp --dp-size $dp --ep-size $ep --pp-size $pp \
--host $host_ip  --port 30000  --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank 2>&1 | tee $logpath/rank$rank-server.log
```

d【tp16ep16dp16】
```bash {wrap}
model_path=/module/GLM-5-FP8
model=${model_path##*/}
tp=16
pp=1
dp=16
ep=16
nodes=2
rank=$1
#master_ip=
# host_ip="172.16.106.73"
host_ip="172.16.106.54"
master_ip=$host_ip
max_model_len=54272
gpu_mem=0.7
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-pp$pp-$host_ip-d-$time"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

# option="--numa-node 3 1 1 0 7 5 5 4"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --page-size 64 "
# option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
# option+=" --attention-backend fa3"
#option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
#option+=" --pp-max-micro-batch-size 2"
option+=" --cuda-graph-max-bs 64 --max-running-requests 1024 "
option+=" --context-length $max_model_len"
# option+=" --disable-cuda-graph  " #--skip-server-warmup
# option+=" --quantization slimquant_marlin  "
option+=" --disaggregation-ib-device mlx5_0"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode decode "
# option+=" --attn-cp-size 8 "
option+=" --kv-cache-dtype fp8_e4m3 "
option+=" --moe-dense-tp-size 1 "
# option+=" --enable-two-batch-overlap "
# --moe-a2a-backend deepep

python -m sglang.launch_server \
--model $model_path  ${option} \
--trust-remote-code \
--dtype bfloat16 --mem-fraction-static $gpu_mem \
--tp-size $tp --dp-size $dp --ep-size $ep --pp-size $pp \
--enable-dp-attention \
--host $host_ip  --port 30000  --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank 2>&1 | tee $logpath/rank$rank-server.log
```

### 

### p【tp8pp3cp8】d【tp32dp32ep32-deepep】

p【tp8pp3cp8】
```bash {wrap}
export NVSHMEM_DISABLE_MNNVL=1
export NVSHMEM_SYMMETRIC_SIZE=10737418240
export NVSHMEM_ENABLE_NIC_PE_MAPPING=1
export NVSHMEM_HCA_PE_MAPPING=mlx5_0:1:2,mlx5_1:1:2,mlx5_2:1:2,mlx5_4:1:2
export SGLANG_DEEPEE_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
export NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME=ibp39s0
# export NVSHMEM_DISABLE_MNNVL=1
# export NVSHMEM_SYMMETRIC_SIZE=10737418240
# export NVSHMEM_ENABLE_NIC_PE_MAPPING=1
# export NVSHMEM_HCA_PE_MAPPING=mlx5_0:1:2,mlx5_1:1:2,mlx5_2:1:2,mlx5_6:1:2
# export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
model_path=/module/GLM-5-FP8
model=${model_path##*/}
tp=8
pp=3
dp=1
ep=1
nodes=3
rank=$1
#master_ip=
host_ip="172.16.106.73"
# host_ip="172.16.106.135"
master_ip=$host_ip
max_model_len=123904
gpu_mem=0.7
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-pp$pp-$host_ip-p-$time"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

# option="--numa-node 3 1 1 0 7 5 5 4"
option+=" --disable-radix-cache "
option+=" --disable-cuda-graph "
option+=" --chunked-prefill-size -1"
option+=" --page-size 64 "
# option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
# option+=" --attention-backend fa3"
option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
#option+=" --pp-max-micro-batch-size 2"
option+=" --cuda-graph-max-bs 32 --max-running-requests 1024 "
option+=" --context-length $max_model_len"
# option+=" --disable-cuda-graph  " #--skip-server-warmup
# option+=" --quantization slimquant_marlin  "
option+=" --disaggregation-ib-device mlx5_0"
option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode prefill "
option+=" --attn-cp-size 8 "
option+=" --kv-cache-dtype fp8_e4m3 "
# option+=" --enable-two-batch-overlap "
# option=" --moe-a2a-backend deepep --moe-runner-backend deep_gemm "
# optiopn=" --enable-eplb true --load-balance-method eplb "
# option=" --ep-num-redundant-experts 8 "

python -m sglang.launch_server \
--model $model_path  ${option} \
--trust-remote-code \
--dtype bfloat16 --mem-fraction-static $gpu_mem \
--tp-size $tp --dp-size $dp --ep-size $ep --pp-size $pp \
--host $host_ip  --port 30000  --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank 2>&1 | tee $logpath/node$nodes-server.log

```

d【tp32dp32ep32-deepep】
```bash {wrap}
#!/bin/bash
set -e
# rm -rf ~/.cache
# rm -rf ~/.triton
# ===================== 你的原有配置 =====================
export NVSHMEM_DISABLE_MNNVL=1
export NVSHMEM_SYMMETRIC_SIZE=10737418240
export NVSHMEM_ENABLE_NIC_PE_MAPPING=1
export NVSHMEM_HCA_PE_MAPPING=mlx5_0:1:2,mlx5_1:1:2,mlx5_2:1:2,mlx5_4:1:2
export SGLANG_DEEPEE_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
export NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME=ibp39s0
model_path=/module/GLM-5-FP8
model=${model_path##*/}
tp=32
pp=1
dp=32
ep=32
nodes=4
rank=$1

# host_ip="172.16.106.73"
host_ip="172.16.106.185"
master_ip=$host_ip
# max_model_len=
max_model_len="123904"
gpu_mem=0.7
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-pp$pp-$host_ip-dd1-$time"

mkdir -p ${logpath}

# ===================== 启动参数（修复错误） =====================
# option=""
# option+=" --disable-radix-cache "
# option+=" --disable-cuda-graph "
option+=" --chunked-prefill-size 16384 "
option+=" --page-size 64 "
option+=" --cuda-graph-max-bs 32 --max-running-requests 1024 "
option+=" --context-length $max_model_len"
option+=" --kv-cache-dtype fp8_e4m3 "
# option+=" --enable-two-batch-overlap "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "

# DeepEP 核心
# option+=" --moe-a2a-backend deepep "
option+=" --moe-a2a-backend deepep --moe-runner-backend deep_gemm "
# option+=" --enable-eplb "
# option+=" --ep-num-redundant-experts 8 "

# 关闭 PP 不兼容功能（必须！否则报错）
# option+=" --disable-overlap-schedule "
# option+=" --disable-speculative-decoding "
# option+=" --disable-mixed-chunked-prefill "

# IB 相关
option+=" --disaggregation-ib-device mlx5_0 "
option+=" --disaggregation-mode decode "

# ===================== 拼接完整命令（关键：打印出来） =====================
CMD="python -m sglang.launch_server \
--model $model_path \
--trust-remote-code \
--dtype bfloat16 --mem-fraction-static $gpu_mem \
--tp-size $tp --dp-size $dp --enable-dp-attention --ep-size $ep --pp-size $pp \
--host $host_ip --port 30000 \
--dist-init-addr $master_ip:5000 \
--nnodes $nodes --node-rank $rank \
$option"

# ===================== 打印最终执行命令（你要的功能） =====================
echo -e "\n==================== 最终执行命令 ===================="
echo "$CMD"
echo -e "========================================================\n"

# 执行并日志输出
$CMD 2>&1 | tee $logpath/rank$rank-server.log
```

### P【tp8pp3cp8ep24-deepep】-d【tp32dp32ep32-deepep】

P【tp8pp3cp8ep24-deepep】
```bash {wrap}
export NVSHMEM_DISABLE_MNNVL=1
export NVSHMEM_SYMMETRIC_SIZE=10737418240
export NVSHMEM_ENABLE_NIC_PE_MAPPING=1
export NVSHMEM_HCA_PE_MAPPING=mlx5_0:1:2,mlx5_1:1:2,mlx5_2:1:2,mlx5_4:1:2
export SGLANG_DEEPEE_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
export NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME=ibp39s0
model_path=/module/GLM-5-FP8
model=${model_path##*/}
tp=8
pp=3
dp=1
ep=24
nodes=3
rank=$1
#master_ip=
host_ip="172.16.106.73"
# host_ip="172.16.106.43"
master_ip=$host_ip
max_model_len=123904
gpu_mem=0.7
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-pp$pp-$host_ip-pp1-$time"
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

# option="--numa-node 3 1 1 0 7 5 5 4"
# option+=" --disable-radix-cache "
option+=" --disable-cuda-graph "
option+=" --chunked-prefill-size -1"
option+=" --page-size 64 "
# option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
# option+=" --attention-backend fa3"
option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
#option+=" --pp-max-micro-batch-size 2"
option+=" --cuda-graph-max-bs 32 --max-running-requests 1024 "
option+=" --context-length $max_model_len"
# option+=" --disable-cuda-graph  " #--skip-server-warmup
# option+=" --quantization slimquant_marlin  "
option+=" --disaggregation-ib-device mlx5_0"
#option+=" --disaggregation-ib-device mlx5_9"
option+=" --disaggregation-mode prefill "

option+=" --attn-cp-size 8 "
option+=" --kv-cache-dtype fp8_e4m3 "
option+=" --enable-dp-lm-head "
# option+=" --enable-two-batch-overlap "
option+=" --moe-a2a-backend deepep --moe-runner-backend deep_gemm "
# optiopn=" --enable-eplb true --load-balance-method eplb "
# option=" --ep-num-redundant-experts 8 "

python -m sglang.launch_server \
--model $model_path  ${option} \
--trust-remote-code \
--dtype bfloat16 --mem-fraction-static $gpu_mem \
--tp-size $tp --dp-size $dp --ep-size $ep --pp-size $pp \
--host $host_ip  --port 30000  --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank 2>&1 | tee $logpath/rank$rank-server.log

```

d【tp32dp32ep32-deepep】
```bash {wrap}
#!/bin/bash
set -e
# rm -rf ~/.cache
# rm -rf ~/.triton
# ===================== 你的原有配置 =====================
export NVSHMEM_DISABLE_MNNVL=1
export NVSHMEM_SYMMETRIC_SIZE=10737418240
export NVSHMEM_ENABLE_NIC_PE_MAPPING=1
export NVSHMEM_HCA_PE_MAPPING=mlx5_0:1:2,mlx5_1:1:2,mlx5_2:1:2,mlx5_4:1:2
export SGLANG_DEEPEE_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
export NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME=ibp39s0
model_path=/module/GLM-5-FP8
model=${model_path##*/}
tp=32
pp=1
dp=32
ep=32
nodes=4
rank=$1

# host_ip="172.16.106.73"
host_ip="172.16.106.185"
master_ip=$host_ip
# max_model_len=
max_model_len="123904"
gpu_mem=0.7
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-pp$pp-$host_ip-dd1-$time"

mkdir -p ${logpath}

# ===================== 启动参数（修复错误） =====================
# option=""
# option+=" --disable-radix-cache "
# option+=" --disable-cuda-graph "
option+=" --chunked-prefill-size 16384 "
option+=" --page-size 64 "
option+=" --cuda-graph-max-bs 32 --max-running-requests 1024 "
option+=" --context-length $max_model_len"
option+=" --kv-cache-dtype fp8_e4m3 "
# option+=" --enable-two-batch-overlap "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "

# DeepEP 核心
# option+=" --moe-a2a-backend deepep "
option+=" --moe-a2a-backend deepep --moe-runner-backend deep_gemm "
# option+=" --enable-eplb "
# option+=" --ep-num-redundant-experts 8 "

# 关闭 PP 不兼容功能（必须！否则报错）
# option+=" --disable-overlap-schedule "
# option+=" --disable-speculative-decoding "
# option+=" --disable-mixed-chunked-prefill "

# IB 相关
option+=" --disaggregation-ib-device mlx5_0 "
option+=" --disaggregation-mode decode "

# ===================== 拼接完整命令（关键：打印出来） =====================
CMD="python -m sglang.launch_server \
--model $model_path \
--trust-remote-code \
--dtype bfloat16 --mem-fraction-static $gpu_mem \
--tp-size $tp --dp-size $dp --enable-dp-attention --ep-size $ep --pp-size $pp \
--host $host_ip --port 30000 \
--dist-init-addr $master_ip:5000 \
--nnodes $nodes --node-rank $rank \
$option"

# ===================== 打印最终执行命令（你要的功能） =====================
echo -e "\n==================== 最终执行命令 ===================="
echo "$CMD"
echo -e "========================================================\n"

# 执行并日志输出
$CMD 2>&1 | tee $logpath/rank$rank-server.log
```

### Router

```plaintext {wrap}

python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://172.16.106.73:30000 --decode http://172.16.106.185:30000 --policy round_robin --port 30020

```

### 客户端

#### 固定pc命中率测试
```bash {wrap}
model_path=/module/GLM-5-FP8
model=${model_path##*/}
time=$(date "+%m%d-%H%M")
mode="cudagraph"
# rr=inf
host_ip="172.16.106.73"
hostname="h20-4"
logpath="client-ali-random/${model}-${mode}-$time-$mode-mem09-${hostname}"
port=30020
#hostname=$hostname
concurrency_multiplier=1
# host_ip=$(hostname -I | awk '{print $1}')

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

# pairs=("8192 1024")
# pairs=("2048 1024")
# pairs=("2048 1")
# pairs=("8192 1024" "2048 1024")
# pairs=("2048 1024" "8192 1024")
total_input_token=51200
gsp_system_prompt_len=38400
gsp_question_len=12800
gsp_output_len=200
# random_rate=$((1.0 - pc_rate))
# random_rate=0.25
echo "input,output,request_rate,num_prompts,max_concurrency,concurrency,Peak_concurrent_requests,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,Peak_output_token_throughput,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log
#--sharegpt-range-ratio 1

#for concurrency_multiplier in 1 ;do
for rr in 0.85; do
    for batch in 1024; do
        # prompt_tokens=${pair%% *}
        # completion_tokens=${pair#* }
        CMD="python3 -m sglang.bench_serving \
        --backend sglang \
        --port $port --host $host_ip \
        --model ${model_path} --request-rate $rr \
        --dataset-name generated-shared-prefix \
        --gsp-num-groups 1 --gsp-prompts-per-group $((batch*concurrency_multiplier)) \
        --gsp-system-prompt-len $gsp_system_prompt_len --gsp-question-len $gsp_question_len --gsp-output-len $gsp_output_len \
        --num-prompts $((batch*concurrency_multiplier))"
        echo "$CMD"


        $CMD 2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log


        # python3 -m sglang.bench_serving --backend sglang-oai \
        # --model ${model_path}   --request-rate $rr --max-concurrency 512 \
        # --port $port --host $host_ip \
        # --dataset-name random-ids \
        # --random-range-ratio 1  \
        # --random-input-len $prompt_tokens \
        # --random-output-len $completion_tokens \
        # --num-prompts $((batch*concurrency_multiplier))  2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log

        #Total token throughput
        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $5}'`
        request_rate=`grep -a "^Traffic request rate" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        mean_Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $2}'`
        Concurrency=`grep -a "^Successful requests:" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $3}'`
        Peak_output_token_throughput=`grep -a "^Peak output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $6}'`
        Peak_concurrent_requests=`grep -a "^Peak concurrent requests" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Mean_TTFT=`grep -a "^Mean TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Median_TTFT=`grep -a "^Median TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Median_TPOT=`grep -a "^Median TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Median_ITL=`grep -a "^Median ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        qps=`grep -a "^Request throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P99_TTFT=`grep -a "^P99 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P99_TPOT=`grep -a "^P99 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P99_ITL=`grep -a "^P99 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P50_TTFT=`grep -a "^P50 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P50_TPOT=`grep -a "^P50 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P50_ITL=`grep -a "^P50 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        Mean_ITL=`grep -a "^Mean ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P95_TTFT=`grep -a "^P95 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P95_TPOT=`grep -a "^P95 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`
        P95_ITL=`grep -a "^P95 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}--total_input_token${total_input_token}.log | awk -F ' ' '{print $4}'`

        echo "$prompt_tokens,$completion_tokens,$request_rate,$((batch*concurrency_multiplier)),$batch,$mean_Concurrency,$Peak_concurrent_requests,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Peak_output_token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log


    done
done
```

#### 调整请求率：
```bash {wrap}
model_path=/module/GLM-5-FP8
model=${model_path##*/}
time=$(date "+%m%d-%H%M")
mode="cudagraph"
# rr=inf
host_ip="172.16.106.43"
logpath="client-ali-random/${model}-${mode}-$time-$mode-mem09-$host_ip"
port=30020
#hostname=$hostname
concurrency_multiplier=1
# host_ip=$(hostname -I | awk '{print $1}')

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

# pairs=("8192 1024")
pairs=("2048 1024")
# pairs=("2048 1")
# pairs=("8192 1024" "2048 1024")
# pairs=("2048 1024" "8192 1024")

echo "input,output,request_rate,num_prompts,max_concurrency,concurrency,Peak_concurrent_requests,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,Peak_output_token_throughput,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log
#--sharegpt-range-ratio 1
for pair in "${pairs[@]}"; do
#for concurrency_multiplier in 1 ;do
    for rr  in 7; do
        for batch in 5000; do
            prompt_tokens=${pair%% *}
            completion_tokens=${pair#* }
            # prompt_tokens=8192
            # completion_tokens=1
            python3 -m sglang.bench_serving --backend sglang-oai \
            --model ${model_path}   --request-rate $rr \
            --port $port --host $host_ip \
            --dataset-name random-ids \
            --random-range-ratio 1  \
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
            mean_Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $2}'`
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

            echo "$prompt_tokens,$completion_tokens,$request_rate,$((batch*concurrency_multiplier)),$batch,$mean_Concurrency,$Peak_concurrent_requests,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Peak_output_token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
            cat $all_log

        done
    done
done

```

# Kimi-K2.5

镜像：docker pull lmsysorg/sglang:v0.5.10rc0
```bash {wrap}
docker run -itd --name=h20-why-sglang \
--privileged \
--network=host \
--device=/dev/kfd --device=/dev/dri \
--ipc=host \
--group-add video \
--cap-add=SYS_PTRACE \
--security-opt seccomp=unconfined \
--ulimit stack=-1:-1 --ulimit memlock=-1:-1 \
--entrypoint bash \
--gpus all \
-v `pwd`:/mnt -v /public/opendas/DL_DATA/llm-models/:/model/ -v /module/:/module -v /modelshare_readonly/:/modelshare_readonly \
 lmsysorg/sglang:latest
```

## p【tp8】
```bash {wrap}
model_path=/module/Kimi-K2.5
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
#--attn-cp-size
host_ip=10.55.55.81
hostname=h20-8
master_ip=$host_ip
max_model_len=81920
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size 8192"
#option+=" --max-running-requests 160 "
option+=" --disable-cuda-graph "
#option+=" --attn-cp-size $cp --enable-prefill-context-parallel"
option+=" --disaggregation-ib-device mlx5_1"
option+=" --disaggregation-mode prefill "


sglang serve --model-path $model_path ${option} \
                                --context-length $max_model_len \
                                --host $host_ip  --port 30123 --trust-remote-code  \
                                --dist-init-addr $master_ip:5123 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16 --kv-cache fp8_e4m3  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64

```

## d【tp8-nomtp】
```bash {wrap}
model_path=/module/Kimi-K2.5
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0

host_ip=10.55.55.80
hostname=h20-7
master_ip=$host_ip
max_model_len=81920
gpu_mem=0.90
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
#option+=" --disable-cuda-graph "
option+=" --disaggregation-ib-device mlx5_1"
option+=" --disaggregation-mode decode "



sglang serve --model-path $model_path ${option} \
                                --context-length $max_model_len \
                                --host $host_ip  --port 30123 --trust-remote-code  \
                                --dist-init-addr $master_ip:5123 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16 --kv-cache fp8_e4m3  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64

```

## Router
```bash {wrap}
python -m sglang_router.launch_router --pd-disaggregation --prefill http://10.55.55.81:30123 --decode http://10.55.55.80:30123 --host 0.0.0.0 --port 8000
```

## 客户端：
```bash {wrap}
model_path=/module/GLM5-Channelwise-FP8-quantized

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
#--max-concurrency $batch 先不加
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

pairs=("4000 20")

echo "input,output,request_rate,num_prompts,concurrency,mean_concurrency,Peak_concurrent_requests,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,Peak_output_token_throughput,mean_ttft_ms,p95_ttft_ms,p99_ttf     t_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log
#--sharegpt-range-ratio 1
#for pair in "${pairs[@]}"; do
#for concurrency_multiplier in 1 ;do
for rr  in 0.8; do
       for batch in 128 ; do

        prompt_tokens=4000
        completion_tokens=1
        python3 -m sglang.bench_serving --backend sglang-oai \
        --model ${model_path}   --request-rate $rr \
        --port $port --host $host_ip \
        --dataset-name random-ids \
        --random-range-ratio 1  \
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
        mean_Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $2}'`
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





        echo "$prompt_tokens,$completion_tokens,$request_rate,$((batch*concurrency_multiplier)),$Concurrency,$mean_Concurrency,$Peak_concurrent_requests,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Peak_output_token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log


     done
done
```

# MiniMax-M2.5

<image token="EpswbA1dhozq47xSwrbcT6JQnMf" width="456" height="182" align="center"/>

```typescript {wrap}
docker run -it --name wen-sglang --gpus all --privileged --shm-size=500G   --device=/dev/kfd --device=/dev/dri/ --cap-add=SYS_PTRACE   --security-opt seccomp=unconfined --ulimit memlock=-1:-1   --ipc=host --network host --group-add video -v /public/home/public_user/wenrh:/mnt -v /public/home/public_user/shiyx/yx_dockerspace:/models --entrypoint /bin/bash lmsysorg/sglang:latest
```

## p服务
```bash {wrap}
export NVSHMEM_ENABLE_NIC_PE_MAPPING=1
export NVSHMEM_HCA_LIST=mlx5_0:1,mlx5_1:1,mlx5_2:1,mlx5_4:1
export NVSHMEM_HCA_PE_MAPPING=mlx5_0:1:2,mlx5_1:1:2,mlx5_2:1:2,mlx5_4:1:2
export NCCL_IB_DISABLE=0
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
export NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME=ibp39s0
#export NCCL_NET_GDR_LEVEL=5
export NVSHMEM_SYMMETRIC_SIZE=536870912

model_path=/models/docker_data/MiniMax-M2.5
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=8
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
echo $host_ip
hostname=$(hostname)
max_model_len=196608
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="p-server/$model-tp$tp-dp$dp-ep$ep-$hostname"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
# --dist-init-addr 10.16.1.52:5000
#option+=" --disable-radix-cache "
option+=" --chunked-prefill-size 8192"
option+=" --max-running-requests 512 "
option+=" --context-length $max_model_len"
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_4"
option+=" --disaggregation-mode prefill --load-balance-method round_robin  --host $host_ip --port 30000" #根据实际情况更换ip
option+=" --nnodes 1 --node-rank 0"
# 移除 --dist-init-addr 以及 nnodes/node-rank 手动指定，让 SGLang 自动处理
python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --kv-cache-dtype fp8_e4m3 \
                                --trust-remote-code  --page-size 64 \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp   --dp-size $dp --ep-size $ep  \
                                --tool-call-parser minimax-m2 \
                                --reasoning-parser minimax-append-think \
                                --enable-dp-attention --moe-a2a-backend deepep  --enable-dp-lm-head --deepep-mode normal \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 \
                                2>&1 | tee  ${logpath}-$time.log

```

## D服务
```bash {wrap}
model_path=/models/docker_data/MiniMax-M2.5
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=8
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
echo $host_ip
max_model_len=131027
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="p-server/$model-tp$tp-dp$dp-ep$ep-$hostname"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
# --dist-init-addr 10.16.1.52:5000
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size 16384"
option+=" --max-running-requests 512 "
option+=" --context-length $max_model_len"
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_4"
option+=" --disaggregation-mode decode  --load-balance-method round_robin  --host $host_ip --port 30001" #根据实际情况更换ip
option+=" --nnodes 1 --node-rank 0"
# 移除 --dist-init-addr 以及 nnodes/node-rank 手动指定，让 SGLang 自动处理
python3 -m sglang.launch_server --model-path $model_path ${option}  \
                                --kv-cache-dtype fp8_e4m3 \
                                --trust-remote-code  --page-size 64 \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp   --dp-size $dp --ep-size $ep  \
                                --tool-call-parser minimax-m2 \
                                --reasoning-parser minimax-append-think \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 \
                                2>&1 | tee  ${logpath}-$time.log

```

## Router
```bash {wrap}
python -m sglang_router.launch_router --pd-disaggregation --prefill http://127.0.0.1:30000 --decode http://127.0.0.1:30001 --host 0.0.0.0 --port 8000
```

## 客户端
```sql {wrap}
curl -X POST http://localhost:30002/v1/chat/completions   -H "Content-Type: application/json"   -d '{
    "model": "default",
    "messages": [
      {
        "role": "user",
        "content": "介绍一下人工智能"
      }
    ]
  }'
```

长输入，75%cache命中率

参考[[飞书知识图谱/资源/locust-50k数据集与性能测试方法--cEyTNnjf|locust-50k数据集与性能测试方法]]
```bash {wrap}
locust -f minimax-test.py --headless -H http://172.16.106.73:30002 --concurrency-levels 14  --custom-model /models/docker_data/MiniMax-M2.5 --level-runs 1  2>&1 | tee  locust-0421.log
```

# Qwen-3.5-397B

## P节点【tp8】
```bash {wrap}
model_path=/model/Qwen3.5-397B-A17B-FP8
model=${model_path##*/}
tp=8
pp=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy no_buffer"
option+=" --chunked-prefill-size -1"
option+=" --disable-cuda-graph "
option+=" --disaggregation-ib-device mlx5_0"
option+=" --disaggregation-mode prefill "


sglang serve --model-path $model_path ${option} \
                                --host $host_ip  --port 30000 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16 --kv-cache fp8_e4m3  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64

```

## D节点【tp8-mtp3】：
```bash {wrap}
export SGLANG_ENABLE_SPEC_V2=1   #mtp overlap

model_path=/model/Qwen3.5-397B-A17B-FP8
model=${model_path##*/}
tp=8
pp=1
nodes=1
rank=0

host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
gpu_mem=0.85
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
#option+=" --mm-attention-backend fa3"
option+=" --disable-radix-cache "
#option+=" --mamba-scheduler-strategy extra_buffer"
option+=" --mamba-scheduler-strategy no_buffer"
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 220 "
option+=" --disaggregation-ib-device mlx5_0"
option+=" --disaggregation-mode decode "

option+=" --cuda-graph-max-bs 220"

option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4"



sglang serve --model-path $model_path ${option} \
                                --host $host_ip  --port 30000 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16 --kv-cache fp8_e4m3  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend fa3 --page-size 64

```

## Router：
```bash {wrap}
python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://10.16.1.36:30000 --decode http://10.16.1.46:30000 --policy round_robin --port 30010
```

## 客户端：
```bash {wrap}
model_path=/model/Qwen3.5-397B-A17B-FP8

model=${model_path##*/}
time=$(date "+%m%d-%H%M")
mode="cudagraph"
rr=inf
logpath="client-ali-random/${model}-${mode}-$time-$mode-rr$rr-mem09-$(hostname)"
port=30010
concurrency_multiplier=1
host_ip=$(hostname -I | awk '{print $1}')

#--max-concurrency $batch 先不加
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

pairs=("4000 20")

echo "input,output,request_rate,num_prompts,concurrency,mean_concurrency,Peak_concurrent_requests,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,Peak_output_token_throughput,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log

for rr  in 5.7; do
       for batch in 3000 ; do

        prompt_tokens=4096
        completion_tokens=1024
        python3 -m sglang.bench_serving --backend sglang-oai \
        --model ${model_path}   --request-rate $rr \
        --port $port --host $host_ip \
        --dataset-name random-ids \
        --random-range-ratio 1  \
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
        mean_Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $2}'`
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





        echo "$prompt_tokens,$completion_tokens,$request_rate,$((batch*concurrency_multiplier)),$Concurrency,$mean_Concurrency,$Peak_concurrent_requests,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Peak_output_token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log


     done
done
```

# Qwen3.5-122B

## P【tp8ep8dp1】
```bash {wrap}
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
model_path=/mnt/gbj/fasttrans/Qwen3.5-122B-A10B-FP8
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=8
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
max_model_len=11000
port=30000
gpu_mem=0.8
time=$(date "+%m%d-%H%M")
logpath="pserver/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
#option+=" --mamba-scheduler-strategy extra_buffer"
option+=" --mamba-scheduler-strategy no_buffer"
option+=" --chunked-prefill-size -1"
#option+=" --max-running-requests 160 "
option+=" --disaggregation-ib-device mlx5_0"
option+=" --disaggregation-mode prefill "
option+=" --disable-cuda-graph"

option+=" --ep-size $ep --deepep-mode normal --moe-a2a-backend deepep "
option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head --deepep-mode normal"


python -m sglang.launch_server --model-path $model_path ${option}  --port $port --tp-size $tp --mem-fraction-static $gpu_mem --reasoning-parser qwen3 --attention-backend fa3 --page-size 64 --kv-cache fp8_e4m3 --dtype bfloat16 --host $host_ip   2>&1 | tee $logpath-$time.log

```

## D【tp8-mtp3】
```bash {wrap}
export NCCL_SOCKET_IFNAME=ibp39s0
export GLOO_SOCKET_IFNAME=ibp39s0
#export MC_IB_GID_INDEX=0
model_path=/mnt/gbj/fasttrans/Qwen3.5-122B-A10B-FP8
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
hostname=$(hostname)
master_ip=$host_ip
max_model_len=22000
port=30000
gpu_mem=0.83
time=$(date "+%m%d-%H%M")
logpath="server/$model-tp$tp-dp$dp-ep$ep-$hostname"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
#option+=" --mamba-scheduler-strategy extra_buffer"
option+=" --disable-radix-cache "
option+=" --mamba-scheduler-strategy no_buffer"
option+=" --max-running-requests 400 "
option+=" --disaggregation-ib-device mlx5_0"
option+=" --disaggregation-mode decode "

option+=" --cuda-graph-max-bs 400"

option+=" --speculative-algo NEXTN --speculative-num-steps 3 --speculative-eagle-topk 1 --speculative-num-draft-tokens 4"

python -m sglang.launch_server --model-path $model_path ${option}  --port $port --tp-size $tp --mem-fraction-static $gpu_mem --context-length $max_model_len --reasoning-parser qwen3 --attention-backend fa3 --page-size 64 --kv-cache fp8_e4m3 --dtype bfloat16 --host $host_ip  2>&1 | tee $logpath-$time.log


```

## Router：
```bash {wrap}
python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://10.16.1.36:30000 --decode http://10.16.1.46:30000 --policy round_robin --port 30010
```

## 客户端：
```bash {wrap}
model_path=/model/Qwen3.5-122B-A10B-FP8

model=${model_path##*/}
time=$(date "+%m%d-%H%M")
mode="cudagraph"
rr=inf
logpath="client-ali-random/${model}-${mode}-$time-$mode-rr$rr-mem09-$(hostname)"
port=30010
concurrency_multiplier=1
host_ip=$(hostname -I | awk '{print $1}')

#--max-concurrency $batch 先不加
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

pairs=("4000 20")

echo "input,output,request_rate,num_prompts,concurrency,mean_concurrency,Peak_concurrent_requests,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,Peak_output_token_throughput,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log

for rr  in 3.8; do
       for batch in 4000 ; do

        prompt_tokens=10240
        completion_tokens=512
        python3 -m sglang.bench_serving --backend sglang-oai \
        --model ${model_path}   --request-rate $rr \
        --port $port --host $host_ip \
        --dataset-name random-ids \
        --random-range-ratio 1  \
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
        mean_Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $2}'`
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





        echo "$prompt_tokens,$completion_tokens,$request_rate,$((batch*concurrency_multiplier)),$Concurrency,$mean_Concurrency,$Peak_concurrent_requests,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Peak_output_token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log


     done
done
```

## 关联资源

- [[飞书知识图谱/资源/H20的PD分离测试过程数据--cejufnSt|H20的PD分离测试过程数据]] · 飞书源链接不可用
- [[飞书知识图谱/资源/PD 分离.md--ckWQSnfd|PD 分离.md]] · 飞书源链接不可用
- [[飞书知识图谱/资源/locust-50k数据集与性能测试方法--cEyTNnjf|locust-50k数据集与性能测试方法]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/EPLB|EPLB]]（concept）
- [[飞书知识图谱/实体/GLM-5|GLM-5]]（model）
- [[飞书知识图谱/实体/GLM-5-FP8|GLM-5-FP8]]（model）
- [[飞书知识图谱/实体/GLM5-Channelwise-FP8-quantized|GLM5-Channelwise-FP8-quantized]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/Kimi-K2.5|Kimi-K2.5]]（model）
- [[飞书知识图谱/实体/Locust|Locust]]（tool）
- [[飞书知识图谱/实体/minimax-append-think|minimax-append-think]]（model）
- [[飞书知识图谱/实体/minimax-m2|minimax-m2]]（model）
- [[飞书知识图谱/实体/MiniMax-M2.5|MiniMax-M2.5]]（model）
- [[飞书知识图谱/实体/minimax-test.py|minimax-test.py]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/NVSHMEM|NVSHMEM]]（concept）
- [[飞书知识图谱/实体/P50|P50]]（metric）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PD 分离|PD 分离]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/Prefix Cache|Prefix Cache]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/Qwen-3.5-397B|Qwen-3.5-397B]]（model）
- [[飞书知识图谱/实体/Qwen3.5-122B|Qwen3.5-122B]]（model）
- [[飞书知识图谱/实体/Qwen3.5-122B-A10B-FP8|Qwen3.5-122B-A10B-FP8]]（model）
- [[飞书知识图谱/实体/Qwen3.5-397B-A17B-FP8|Qwen3.5-397B-A17B-FP8]]（model）
- [[飞书知识图谱/实体/RPS|RPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/SLA|SLA]]（metric）
- [[飞书知识图谱/实体/TBO|TBO]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）

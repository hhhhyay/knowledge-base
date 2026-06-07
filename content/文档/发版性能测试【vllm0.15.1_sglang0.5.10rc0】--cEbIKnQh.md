---
title: 发版性能测试【vllm0.15.1/sglang0.5.10rc0】
tags:
  - AI
  - LLM
  - Benchmark
model:
  - GLM-5-W8A8
  - glm5.1-int8
  - kimi2.5-int4
  - minimax-int8
  - Qwen3.5-397B-A17B-FP8
  - qwen3.5-397b-int8
framework:
  - SGLang
  - vLLM
issue_type: []
source: feishu
feishu_token: E5ewdtRb3o4anwxkslfcEbIKnQh
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/E5ewdtRb3o4anwxkslfcEbIKnQh
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 发版性能测试【vllm0.15.1/sglang0.5.10rc0】

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/E5ewdtRb3o4anwxkslfcEbIKnQh)

# 测试镜像：

<callout emoji="💡" background-color="light-orange" border-color="light-orange">

10.16.1.152:5000/jenkins/model_test_env/vllm:0.15.1-ubuntu22.04-dtk26.04-py3.10-20260515-1239

10.16.1.152:5000/jenkins/model_test_env/sglang:0.5.10rc0-ubuntu22.04-dtk26.04-py3.10-20260522-0959
</callout>

# 测试硬件：

nmz、bmz、H20

# 测试模型：

glm5.1-int8、kimi2.5-int4、minimax-int8、qwen3.5-397b-int8

# 测试参数：

（2k，1k）（64k，1k）命中率90% 

# 测试要求：

tpot 50ms

# 测试结果

见：[[飞书知识图谱/资源/vllm0.15.1发版性能--cI6WnnSb|vllm0.15.1发版性能]][[飞书知识图谱/资源/发版性能测试【sglang】--cM4TmnWe|发版性能测试【sglang】]][[飞书知识图谱/资源/202605发版性能【vllm_sglang】--csOXRnph|202605发版性能【vllm/sglang】]]

# 测试命令：

https://kcnm6g5dkw5p.feishu.cn/wiki/QzOowiQVOiuaRCkbTjecnZVpn1d

---

**弃用**

<text bgcolor="dark-gray">~~vllm客户端脚本~~</text>

<view type="2">

  <file token="KFVbbFjmZoSfhMx3UpkcyIknnYb" name="client.sh"/>

</view>


```plaintext
比如测试pc命中率90%，可第一遍按照95%设置输出长度为1，快速预热，观察到pc命中率稳定在90%左右即可停止预热：
#第一遍： 
bash xx-client.sh 95 #观察serverlog PC命中率稳定在90%之上才算成功
(大输入，1输出，快速预热pc命中率)
#第二遍
bash xx-clinet.sh 90 #正式测试，此时观察server.log 中PC命中率已稳定在90%左右
```

```bash
#!/bin/bash
#
# KV Cache (Prefix Cache) 命中率性能测试脚本
#
# 原理说明：
#   vllm bench serve 的 --dataset-name random 模式支持 --random-prefix-len 参数，
#   该参数指定所有请求共享一个固定前缀的 token 数量。
#   当第一个请求处理完后，这段前缀的 KV cache 会被缓存；后续请求如果包含相同前缀，
#   就会命中 prefix cache，从而跳过前缀部分的 prefill 计算。
#
#   KV Cache 命中率 ≈ random_prefix_len / total_input_len
#
#   例如：total_input_len=10000, 想测 50% 命中率 → random_prefix_len=5000
#
# 用法：
#   ./client_prefix_cache.sh                    # 使用默认参数
#   ./client_prefix_cache.sh 0 25 50 75 90      # 指定要测试的命中率列表（百分比）
#
model_path=/model/vllm-w8a8-models/GLM-5-W8A8
model=${model_path##*/}
tp=8
time=$(date "+%m%d-%H%M")
mode="cudagraph"
data_type="fp16"#不影响
port=9349
#port=9349
# ===================== 可调参数 =====================
# 总输入长度（token 数）
INPUT_LEN=65536
# 输出长度（token 数）
OUTPUT_LEN=256
# 要测试的 KV Cache 命中率列表（百分比），可通过命令行参数覆盖
if [ $# -gt 0 ]; then
    CACHE_HIT_RATES=("$@")
else
    CACHE_HIT_RATES=(90)
fi
# 并发 batch 列表
BATCHES=(1 2 4 8)
# 并发乘数（用于计算总请求数 = batch * multiplier）
CONCURRENCY_MULTIPLIER=4
# 高并发时的乘数阈值
HIGH_BATCH_THRESHOLD=128
HIGH_BATCH_MULTIPLIER=4
# ===================================================
logpath="./client/${model}-${mode}-${time}-prefix-cache-test"
if [ ! -d "${logpath}" ]; then
    mkdir -p "${logpath}"
fi
all_log="${logpath}/all.csv"
# CSV 头部，新增 cache_hit_pct 和 prefix_len 字段
echo "cache_hit_pct,prefix_len,input,output,request_rate,num_prompts,concurrency,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms,QPM,prefill_throughput_tok_s_per_gpu,decode_throughput_tok_s_per_gpu" > "$all_log"
echo "============================================================"
echo " KV Cache 命中率性能测试"
echo " 模型: ${model_path}"
echo " TP: ${tp}"
echo " 总输入长度: ${INPUT_LEN} tokens"
echo " 输出长度: ${OUTPUT_LEN} tokens"
echo " 测试命中率: ${CACHE_HIT_RATES[*]}%"
echo " 并发列表: ${BATCHES[*]}"
echo "============================================================"
for cache_hit_pct in "${CACHE_HIT_RATES[@]}"; do
    # 计算前缀长度 = 总输入长度 × 命中率百分比 / 100
    prefix_len=$(( INPUT_LEN * cache_hit_pct / 100 ))
    echo ""
    echo ">>> 测试 KV Cache 命中率: ${cache_hit_pct}%  (prefix_len=${prefix_len}, total_input=${INPUT_LEN})"
    echo "------------------------------------------------------------"
    for batch in "${BATCHES[@]}"; do
        concurrency_multiplier=${CONCURRENCY_MULTIPLIER}
        if [ "$batch" -gt ${HIGH_BATCH_THRESHOLD} ]; then
            concurrency_multiplier=${HIGH_BATCH_MULTIPLIER}
        fi
        num_prompts=$((batch * concurrency_multiplier))
        log_file="${logpath}/${model}-tp${tp}-${time}-batch${batch}-cache${cache_hit_pct}pct-in${INPUT_LEN}-out${OUTPUT_LEN}.log"
        echo "  → batch=${batch}, num_prompts=${num_prompts}, prefix_len=${prefix_len}"
        # 构建 benchmark 命令
        # 注意：--random-input-len 是除 prefix 外的随机部分长度
        # 实际总输入 = prefix_len + random_input_len
        # 所以 random_input_len = INPUT_LEN - prefix_len
        random_input_len=$(( INPUT_LEN - prefix_len ))
        vllm bench serve \
            --model "${model_path}" \
            --dataset-name random \
            --random-input-len "$random_input_len" \
            --random-output-len "$OUTPUT_LEN" \
            --random-prefix-len "$prefix_len" \
            --num-prompts "$num_prompts" \
            --port "$port" \
            --metric-percentiles 95,99 \
            --max-concurrency "$batch" \
            --ignore-eos \
            --trust-remote-code \
            2>&1 | tee "$log_file"
        # 提取各项指标
        Benchmark_duration=$(grep -a "^Benchmark duration" "$log_file" | awk -F ' ' '{print $4}')
        Total_input_tokens=$(grep -a "^Total input tokens" "$log_file" | awk -F ' ' '{print $4}')
        Total_generated_tokens=$(grep -a "^Total generated tokens" "$log_file" | awk -F ' ' '{print $4}')
        Output_token_throughput=$(grep -a "^Output token throughput" "$log_file" | awk -F ' ' '{print $5}')
        Total_Token_throughput=$(grep -a "^Total token throughput" "$log_file" | awk -F ' ' '{print $5}')
        request_rate=$(grep -a "^Traffic request rate" "$log_file" | awk -F ' ' '{print $4}')
        Mean_TPOT=$(grep -a "^Mean TPOT" "$log_file" | awk -F ' ' '{print $4}')
        Mean_TTFT=$(grep -a "^Mean TTFT" "$log_file" | awk -F ' ' '{print $4}')
        Median_TTFT=$(grep -a "^Median TTFT" "$log_file" | awk -F ' ' '{print $4}')
        Median_TPOT=$(grep -a "^Median TPOT" "$log_file" | awk -F ' ' '{print $4}')
        Median_ITL=$(grep -a "^Median ITL" "$log_file" | awk -F ' ' '{print $4}')
        qps=$(grep -a "^Request throughput" "$log_file" | awk -F ' ' '{print $4}')
        P99_TTFT=$(grep -a "^P99 TTFT" "$log_file" | awk -F ' ' '{print $4}')
        P99_TPOT=$(grep -a "^P99 TPOT" "$log_file" | awk -F ' ' '{print $4}')
        P99_ITL=$(grep -a "^P99 ITL" "$log_file" | awk -F ' ' '{print $4}')
        P95_TTFT=$(grep -a "^P95 TTFT" "$log_file" | awk -F ' ' '{print $4}')
        P95_TPOT=$(grep -a "^P95 TPOT" "$log_file" | awk -F ' ' '{print $4}')
        P95_ITL=$(grep -a "^P95 ITL" "$log_file" | awk -F ' ' '{print $4}')
        Mean_ITL=$(grep -a "^Mean ITL" "$log_file" | awk -F ' ' '{print $4}')
        # 计算派生指标
        QPM=$(awk "BEGIN {print ${qps:-0} * 60}")
        prefill_throughput=$(awk "BEGIN {printf \"%.2f\", ${Total_input_tokens:-0} / ${Benchmark_duration:-1} / ${tp}}")
        decode_throughput=$(awk "BEGIN {printf \"%.2f\", ${Total_generated_tokens:-0} / ${Benchmark_duration:-1} / ${tp}}")
        # 追加到 CSV
        echo "${cache_hit_pct},${prefix_len},${INPUT_LEN},${OUTPUT_LEN},${request_rate},${num_prompts},${batch},${Benchmark_duration},${qps},${Output_token_throughput},${Total_Token_throughput},${Mean_TTFT},${P95_TTFT},${P99_TTFT},${Mean_TPOT},${P95_TPOT},${P99_TPOT},${Mean_ITL},${P95_ITL},${P99_ITL},${QPM},${prefill_throughput},${decode_throughput}" >> "$all_log"
        echo ""
        echo "  ✓ cache_hit=${cache_hit_pct}% batch=${batch} 完成"
        echo ""
    done
done
echo ""
echo "============================================================"
echo " 测试完成！结果汇总："
echo "============================================================"
cat "$all_log"
echo ""
echo "完整结果已保存到: ${all_log}"

```

<text bgcolor="dark-gray">~~sglang的测试脚本~~</text>

<text bgcolor="dark-gray">P95指标需要替换下bench_serving.py文件</text>

[[飞书知识图谱/资源/bench_serving.py--c8W0Yn6S|bench_serving.py]]

<text bgcolor="dark-gray">cp bench_serving.py /usr/local/lib/python3.10/dist-packages/sglang/bench_serving.py</text>
```bash
#!/bin/bash
#
# KV Cache (Prefix Cache) 命中率性能测试脚本
#
model_path=/model/Qwen3.5-397B-A17B-FP8
model=${model_path##*/}
tp=8
time=$(date "+%m%d-%H%M")
mode="cudagraph"
data_type="fp8"
port=30005
# ===================== 可调参数 =====================
INPUT_LEN=2048
OUTPUT_LEN=1024

if [ $# -gt 0 ]; then
    CACHE_HIT_RATES=("$@")
else
        CACHE_HIT_RATES=(90)
fi
#BATCHES=(1 2 4 8 16 32 64 128)
batch_seq=$({
  seq 1 1 64
  seq 66 2 256
  seq 260 4 512
  seq 520 8 1024
})
#BATCHES=(128)
CONCURRENCY_MULTIPLIER=1
# 高并发时的乘数阈值
HIGH_BATCH_THRESHOLD=128
HIGH_BATCH_MULTIPLIER=1
host_ip=$(hostname -I 2>/dev/null | awk '{print $1}')
# ===================================================
logpath="./client/${model}-${mode}-${time}-prefix-cache-test"
#logpath="./client/${model}-${mode}-${time}-prefix-cache-test/0330/45k-250-pc-mtp2"
mkdir -p "${logpath}"
all_log="${logpath}/all.csv"
# ✅ 新表头（已替换）
echo "input,output,request_rate,num_prompts,concurrency,duration_s,\
rps,generate_throughput_tok_s,total_throughput_tok_s,\
mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,\
mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,\
mean_itl_ms,p95_itl_ms,p99_itl_ms" > "$all_log"
echo "============================================================"
echo " KV Cache 命中率性能测试"
echo " 模型: ${model_path}"
echo " TP: ${tp}"
echo " 总输入长度: ${INPUT_LEN} tokens"
echo " 输出长度: ${OUTPUT_LEN} tokens"
echo " 测试命中率: ${CACHE_HIT_RATES[*]}%"
echo " 并发列表: ${BATCHES[*]}"
echo "============================================================"
pairs=(
"2048 1024 200"
"65536 1024 200"
)
cache_hit_pct=90

for pair in "${pairs[@]}"; do
    INPUT_LEN=$(echo "$pair" | awk '{print $1}')
    OUTPUT_LEN=$(echo "$pair" | awk '{print $2}')

    prefix_len=$(( INPUT_LEN * cache_hit_pct / 100 ))
    echo ""
    echo ">>> 测试 KV Cache 命中率: ${cache_hit_pct}%  (prefix_len=${prefix_len})"
    echo "------------------------------------------------------------"
    for batch in $batch_seq;do

        concurrency_multiplier=${CONCURRENCY_MULTIPLIER}
        if [ "$batch" -gt ${HIGH_BATCH_THRESHOLD} ]; then
            concurrency_multiplier=${HIGH_BATCH_MULTIPLIER}
        fi
        num_prompts=$((batch * concurrency_multiplier))
        log_file="${logpath}/${model}-tp${tp}-${time}-batch${batch}-cache${cache_hit_pct}pct-in${INPUT_LEN}-out${OUTPUT_LEN}.log"
        echo "  → batch=${batch}, num_prompts=${num_prompts}, prefix_len=${prefix_len}"
        random_input_len=$(( INPUT_LEN - prefix_len ))
        python3 -m sglang.bench_serving \
        --backend sglang --host $host_ip\
        --port $port \
        --model ${model_path} \
        --dataset-name generated-shared-prefix \
        --gsp-num-groups 1 \
        --gsp-prompts-per-group $num_prompts \
        --gsp-system-prompt-len $prefix_len \
        --gsp-question-len $random_input_len \
        --gsp-output-len $OUTPUT_LEN \
        --num-prompts $num_prompts \
        --max-concurrency $batch \
        2>&1 | tee "$log_file"
        # ================= 指标提取 =================
        duration_s=$(grep -a "^Benchmark duration" "$log_file" | awk '{print $4}')
        rps=$(grep -a "^Request throughput" "$log_file" | awk '{print $4}')
        Total_input_tokens=$(grep -a "^Total input tokens" "$log_file" | awk '{print $4}')
        Total_generated_tokens=$(grep -a "^Total generated tokens" "$log_file" | awk '{print $4}')
        # throughput
        generate_throughput=$(grep -a "^Output token throughput" "$log_file" | awk '{print $5}')
        total_throughput=$(grep -a "^Total token throughput" "$log_file" | awk '{print $5}')
        # TTFT
        mean_ttft_ms=$(grep -a "^Mean TTFT" "$log_file" | awk '{print $4}')
        p95_ttft_ms=$(grep -a "^P95 TTFT" "$log_file" | awk '{print $4}')
        p99_ttft_ms=$(grep -a "^P99 TTFT" "$log_file" | awk '{print $4}')
        # TPOT
        mean_tpot_ms=$(grep -a "^Mean TPOT" "$log_file" | awk '{print $4}')
        p95_tpot_ms=$(grep -a "^P95 TPOT" "$log_file" | awk '{print $4}')
        p99_tpot_ms=$(grep -a "^P99 TPOT" "$log_file" | awk '{print $4}')
        # ITL（兼容不同版本）
        mean_itl_ms=$(grep -a "^Mean ITL" "$log_file" | awk '{print $4}')
        p95_itl_ms=$(grep -a "^P95 ITL" "$log_file" | awk '{print $4}')
        p99_itl_ms=$(grep -a "^P99 ITL" "$log_file" | awk '{print $4}')
        if [ -z "$mean_itl_ms" ]; then
            mean_itl_ms=$(grep -a "Mean Inter-token latency" "$log_file" | awk '{print $5}')
            p95_itl_ms=$(grep -a "P95 Inter-token latency" "$log_file" | awk '{print $5}')
            p99_itl_ms=$(grep -a "P99 Inter-token latency" "$log_file" | awk '{print $5}')
        fi
        # request_rate（无 rate limit，用 rps 代替）
        #request_rate=${rps:-0}
        request_rate=inf
        # ================= 写入 =================
        echo "${INPUT_LEN},${OUTPUT_LEN},${request_rate},${num_prompts},${batch},${duration_s},\
${rps},${generate_throughput},${total_throughput},\
${mean_ttft_ms},${p95_ttft_ms},${p99_ttft_ms},\
${mean_tpot_ms},${p95_tpot_ms},${p99_tpot_ms},\
${mean_itl_ms},${p95_itl_ms},${p99_itl_ms}" >> "$all_log"
        echo "  ✓ cache_hit=${cache_hit_pct}% batch=${batch} 完成"
        echo ""
        cat $all_log
    done
done
echo ""
echo "============================================================"
echo " 测试完成！结果汇总："
echo "============================================================"
cat "$all_log"
echo ""
echo "完整结果已保存到: ${all_log}"

```

## 关联资源

- [[飞书知识图谱/资源/vllm0.15.1发版性能--cI6WnnSb|vllm0.15.1发版性能]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/PHfIwSDfiiJLdukKOrLcI6WnnSb)
- [[飞书知识图谱/资源/发版性能测试【sglang】--cM4TmnWe|发版性能测试【sglang】]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/sheets/FlljsMXgjhqHRktzLWncM4TmnWe)
- [[飞书知识图谱/资源/202605发版性能【vllm_sglang】--csOXRnph|202605发版性能【vllm/sglang】]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/sheets/Qe2xs5C6gh8YI8tWobvcsOXRnph)
- [[飞书知识图谱/资源/bench_serving.py--c8W0Yn6S|bench_serving.py]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/file/Or2ybK7icovPwdxYhSAc8W0Yn6S)

## 关联知识

- [[飞书知识图谱/实体/并发|并发]]（metric）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/glm5.1-int8|glm5.1-int8]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/kimi2.5-int4|kimi2.5-int4]]（model）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/minimax-int8|minimax-int8]]（model）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/Prefix Cache|Prefix Cache]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/Qwen3.5-397B-A17B-FP8|Qwen3.5-397B-A17B-FP8]]（model）
- [[飞书知识图谱/实体/qwen3.5-397b-int8|qwen3.5-397b-int8]]（model）
- [[飞书知识图谱/实体/RPS|RPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

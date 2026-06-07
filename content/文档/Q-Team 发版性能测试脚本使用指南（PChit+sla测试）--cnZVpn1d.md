---
title: Q-Team 发版性能测试脚本使用指南（PChit+sla测试）
tags:
  - AI
  - LLM
  - Benchmark
model:
  - GLM-4.7-W8A8
framework:
  - SGLang
  - vLLM
issue_type: []
source: feishu
feishu_token: QzOowiQVOiuaRCkbTjecnZVpn1d
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/QzOowiQVOiuaRCkbTjecnZVpn1d
last_synced_at: 2026-06-07T17:07:53.393Z
---

# Q-Team 发版性能测试脚本使用指南（PChit+sla测试）

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/QzOowiQVOiuaRCkbTjecnZVpn1d)

http://42.228.13.241:10068/liuzhh8/pchit-perf-benchmark/-/blob/main/README.md gitlib地址

## 1. 快速开始（优先使用最新的）

先安装依赖：
```bash
pip install aiohttp
```

测试脚本：

v4：增加请求率限制功能，请求率默认不限制，不加此参数不影响之前脚本使用

<view type="1">

  <file token="JfV0bD6wqoUgr0xRjOccVPCtnMe" name="prefix_cache_benchmark.py"/>

</view>

```shell
request_rate=3
echo "  request_rate       = ${request_rate}"
--request-rate "${request_rate}" \
```


v3：修复sla模式有请求失败就终止的情况，只以sla条件为终止符（csv保存未汇总）

<view type="1">

  <file token="Az6gbFVx3oUsEKxT0v7c6twJnpb" name="prefix_cache_benchmark.py"/>

</view>

v2:

<view type="1">

  <file token="COKAb2baior5zWxj9pGc4NrlnCN" name="prefix_cache_benchmark.py"/>

</view>

<callout emoji="musical_keyboard" background-color="light-orange" border-color="light-orange">

修复了一点小功能，上个版本可能会出现一点问题，上个版本构造的输入，除了90%的共享前缀，另外10%是随机生成的，但是<text bgcolor="light-yellow">随机种子是固定的</text>，可能会导致，如果server没断，client重复运行了两遍，pc命中率可能会打满（在server log里体现的是命中率超过90%，91%也是出现了部分全命中的）情况，如果脚本只运行一遍不会出现此问题。
</callout>

~~v1:弃用~~

<view type="1">

  <file token="C67jbI93HokZMHxxathcIunDn1d" name="prefix_cache_benchmark.py"/>

</view>

<block-ref id="Z2GydITdZogEOXxylxccxRuxncc"/>（已总结到本文）

<file token="RU3pb1m4xoUohSxlP6rcc6VcnFd" name="README_prefix_cache_benchmark.md"/>

## 2. 测试模式<text bgcolor="light-yellow">(必看)</text>

### 2.1 找最大满足 SLA 并发
```bash
--mode sla-search
```

<text bgcolor="light-yellow">例子</text>

下面命令同时测试2k/ 64k 输入、1k 输出、90% PC 命中率，并按 mean SLA 找最大并发。

<callout emoji="musical_score" background-color="light-orange" border-color="light-orange">

  注意：目前发现sglang- PD分离由于代理Router请求体接受格式问题，使用此脚本测试，client(shell)脚本需增加这两个参数：
  --str-input \
  --tokenizer-path "${model_path}" \
</callout>

```bash
#!/usr/bin/env bash
set -euo pipefail

model_path=/model/GLM-4.7-W8A8
model=${model_path##*/}
base_url="http://127.0.0.1:9349"

# 输入 / 输出长度
input_lengths=(2048 65536)
output_length=1024

# SLA 配置
prefix_cache_hit=0.9
ttft_sla_ms=30000
tpot_sla_ms=75
sla_stat="mean"

# 并发搜索范围
min_concurrency=1
max_concurrency=256

# 每个并发的请求数 = 并发 * request_multiplier
request_multiplier=4

# 每个 bs 预热次数
case_warmup_repeats=1

# 结果输出目录
save_path="./results"

current_time=$(date +"%Y%m%d-%H%M")
logpath="./log/${model}-prefix-cache-sla-${current_time}"
mkdir -p "${logpath}"
mkdir -p "${save_path}"

build_concurrency_list() {
  local min_c="$1"
  local max_c="$2"
  local c="${min_c}"
  local step
  local result=()

  while [ "${c}" -le "${max_c}" ]; do
    result+=("${c}")

    # step 规则：
    # 1-31:    step=1
    # 32-63:   step=2
    # 64-127:  step=4
    # 128-255: step=8
    # 256+:    step=16
    if [ "${c}" -lt 32 ]; then
      step=1
    elif [ "${c}" -lt 64 ]; then
      step=2
    elif [ "${c}" -lt 128 ]; then
      step=4
    elif [ "${c}" -lt 256 ]; then
      step=8
    else
      step=16
    fi

    c=$((c + step))
  done

  printf '%s\n' "${result[@]}"
}

mapfile -t concurrency_list < <(build_concurrency_list "${min_concurrency}" "${max_concurrency}")

if [ "${#concurrency_list[@]}" -eq 0 ]; then
  echo "ERROR: concurrency_list is empty" >&2
  exit 1
fi

last_index=$((${#concurrency_list[@]} - 1))

echo "Generated concurrency list:"
printf ' %s' "${concurrency_list[@]}"
echo
echo "First concurrency: ${concurrency_list[0]}"
echo "Last concurrency: ${concurrency_list[$last_index]}"
echo "Total concurrency count: ${#concurrency_list[@]}"
echo

for input_length in "${input_lengths[@]}"; do
  logfile="${logpath}/${model}-${current_time}-sla-in${input_length}-out${output_length}.log"

  echo "============================================================"
  echo "Running SLA search:"
  echo "  model              = ${model_path}"
  echo "  base_url           = ${base_url}"
  echo "  input_length       = ${input_length}"
  echo "  output_length      = ${output_length}"
  echo "  prefix_cache_hit   = ${prefix_cache_hit}"
  echo "  ttft_sla_ms        = ${ttft_sla_ms}"
  echo "  tpot_sla_ms        = ${tpot_sla_ms}"
  echo "  sla_stat           = ${sla_stat}"
  echo "  request_multiplier = ${request_multiplier}"
  echo "  warmup_repeats     = ${case_warmup_repeats}"
  echo "  concurrency_list   = ${concurrency_list[*]}"
  echo "  log                = ${logfile}"
  echo "============================================================"

  python3 prefix_cache_benchmark.py \
    --mode sla-search \
    --base-url "${base_url}" \
    --model "${model_path}" \
    --input-lengths "${input_length}" \
    --output-length "${output_length}" \
    --prefix-cache-hit "${prefix_cache_hit}" \
    --ttft-sla-ms "${ttft_sla_ms}" \
    --tpot-sla-ms "${tpot_sla_ms}" \
    --sla-stat "${sla_stat}" \
    --concurrency-list "${concurrency_list[@]}" \
    --request-multiplier "${request_multiplier}" \
    --case-warmup-repeats "${case_warmup_repeats}" \
    --save-path "${save_path}" \
    2>&1 | tee "${logfile}"

  echo
done

echo "============================================================"
echo "All SLA search jobs finished."
echo "Logs saved to: ${logpath}"
echo "Results saved to: ${save_path}"
echo "============================================================"
```

#### 输出样例1（单bs预热数据）：

<image token="RvBmb3PIpoYvacx1BG8c3yR8nab" width="455" height="709" align="center"/>

#### 输出样例2（单bs正式测试数据）：

<image token="DjfSbvYt2oZeElxzOKQcIMQXnNe" width="428" height="652" align="center"/>

#### 输出样例3（SLA测试输出）：

<image token="GPOBb12YmoMuLRxcjmRc0snsnjb" width="1722" height="355" align="center"/>

#### 单例：
```bash

python3 prefix_cache_benchmark.py \
  --mode sla-search \
  --base-url http://127.0.0.1:9349 \
  --model /model/GLM-4.7-W8A8 \
  --input-lengths 2048 65536 \
  --output-length 1024 \
  --prefix-cache-hit 0.9 \
  --ttft-sla-ms 10000 \
  --tpot-sla-ms 50 \
  --sla-stat mean \
  --min-concurrency 1 \
  --max-concurrency 256 \
  --concurrency-step 1 \
  --request-multiplier 4 \
  --case-warmup-repeats 1 \
  --save-path ./results
```

<callout emoji="glass_of_milk" background-color="light-orange" border-color="light-orange">

Prefix warmup 一个长度预热一次

--case-warmup-repeats 每个bs预热1次，设置为1，请求数使用4倍并发。  --max-concurrency 256 不影响，这里会找到sla最大并发自停；step并发步长设置为1-32为1；32-64为2；64-128为4；128-256为8；256-更大为16；（2048，1024），（65536，1024）
</callout>

行为：

- 从低并发向高并发测试。
- 某个并发 `sla_pass=FAIL` 后，当前 input length 停止继续向上测。
- 最终输出 `best_sla_concurrency`。

### 2.2 按指定 BS 完整测试

<callout emoji="💡" background-color="light-orange" border-color="light-orange">

注意：如果目标是“到达 SLA fail 就停止”，不要用 `fixed`，应该用 `sla-search`。
</callout>

下面命令只测试 `bs1/bs2/bs3`，不会因为 SLA fail 提前停止。
```bash
python3 prefix_cache_benchmark.py \
  --mode fixed \
  --base-url http://127.0.0.1:9349 \
  --model /model/GLM-4.7-W8A8 \
  --input-lengths 65536 \
  --output-length 1024 \
  --prefix-cache-hit 0.9 \
  --concurrency-list 1 2 3 \
  --request-multiplier 4 \
  --case-warmup-repeats 1 \
  --save-path ./results
```

如果要测试更多指定 BS：
```bash
--concurrency-list 1 2 4 8 16 32 64
```

使用：
```bash
--mode fixed
```

行为：

- 按配置的并发列表完整跑完。
- 即使某个并发 `sla_pass=FAIL`，也会继续测试后续并发。
- 仍然会计算 `sla_pass`，方便后处理筛选。

适用场景：

- 要强制测试指定并发，例如 `1 2 3` 或 `1 2 4 8 16 32 64`。
- 要比较多个 BS 的 TTFT、TPOT、吞吐变化。

## 3.SLA 搜索流程（非必看）

SLA 参数：
```bash
--ttft-sla-ms 5000
--tpot-sla-ms 50
--sla-stat mean
```

`--sla-stat` 可选：
```plaintext
mean
p95
p99
```

如果使用默认：
```bash
--sla-stat mean
```

则通过条件是：
```plaintext
mean_ttft_ms <= ttft_sla_ms
mean_tpot_ms <= tpot_sla_ms
failed_requests == 0
```

任意条件不满足，该 case 就是 `FAIL`。

结果示例：
```plaintext
bs1 PASS
bs3 PASS
bs5 PASS
bs7 FAIL
```

如果测试目标是找最大满足 SLA 并发，那么最大通过并发是：
```plaintext
bs5
```

如果命令使用的是 `--mode sla-search`，脚本会在 `bs7 FAIL` 后停止。

如果命令使用的是 `--mode fixed`，脚本会继续测试后续 BS，这是符合设计的。

## 4.自定义 BS 测试流程（非必看）

自定义 BS 测试使用 `fixed` 模式。

示例：测试 `bs1/bs2/bs3`。
```bash
python3 prefix_cache_benchmark.py \
  --mode fixed \
  --base-url http://127.0.0.1:9349 \
  --model /model/GLM-4.7-W8A8 \
  --input-lengths 65536 \
  --output-length 1024 \
  --prefix-cache-hit 0.9 \
  --concurrency-list 1 2 3 \
  --request-multiplier 8 \
  --case-warmup-repeats 1 \
  --save-path ./results_fixed_bs1_2_3
```

示例：测试常见倍增并发。
```bash
python3 prefix_cache_benchmark.py \
  --mode fixed \
  --base-url http://127.0.0.1:9349 \
  --model /model/GLM-4.7-W8A8 \
  --input-lengths 65536 \
  --output-length 1024 \
  --prefix-cache-hit 0.9 \
  --concurrency-list 1 2 4 8 16 32 64 \
  --request-multiplier 8 \
  --case-warmup-repeats 1 \
  --save-path ./results_fixed_curve
```

`fixed` 模式不会因为 fail 自动停止

## 其他参数说明（非必看）

有两种设置并发列表的方式。

### 显式指定并发列表
```bash
--concurrency-list 1 2 3
```

表示只测试：
```plaintext
bs1, bs2, bs3
```

这是最不容易误解的方式，适合自定义测试。

### 用 min/max/step 自动生成
```bash
--min-concurrency 1
--max-concurrency 32
--concurrency-step 2
```

生成的是：
```plaintext
1, 3, 5, 7, ..., 31
```

不是：
```plaintext
1, 2, 3, ..., 32
```

如果要连续搜索 `1~32`：
```bash
--min-concurrency 1
--max-concurrency 32
--concurrency-step 1
```

如果要测试偶数并发：
```bash
--concurrency-list 2 4 6 8 10 12
```

或者：
```bash
--min-concurrency 2
--max-concurrency 12
--concurrency-step 2
```

## 为什么PC命中率有时候达不到设定的值？

#### 1.累加计算
```shell
`--prefix-cache-hit` 控制的是脚本怎么构造请求，不是强制服务端日志一定显示这个值。

比如设置：

```bash
--prefix-cache-hit 0.9
脚本会尽量构造“约 90% 共享前缀 + 约 10% 随机后缀”的 prompt。也就是说，客户端侧的目标是 90% PC 命中。
但服务端日志里的：Prefix cache hit rate
不一定刚好等于 90%，常见有两个原因。
1. 服务端通常是累计统计
服务端日志里的 PC 命中率很多时候是累计值，不是当前这一个 case、这一条请求的独立值。
压测过程中会有：
HTTP warmup
Prefix warmup
Case warmup
Bench
其中第一条 prefix warmup 长请求通常是 cold miss，它的作用是把 prefix 填进 cache，本身不一定命中。后面的请求才会逐步命中这个 prefix。
所以你可能会看到服务端日志里的命中率从低到高慢慢爬：
41%
56%
74%
82%
85%
这不代表脚本一开始只构造了 41% 或 85% 的共享前缀，而是因为服务端把前面的 cold miss、warmup、bench、甚至历史请求都算进去了。
如果想看某一个 case 的真实命中率，最好看 metrics 的增量，而不是只看日志里的累计值：
本 case 命中率 = 本 case 新增 prefix_cache_hits / 本 case 新增 prefix_cache_queries
本 case 命中率 = 本 case 新增 prefix_cache_hits / 本 case 新增 prefix_cache_queries
```

#### 2. 开启 MTP 后，服务端统计口径可能会变化

如果服务端开了 MTP / speculative decoding，服务端内部的调度和统计路径会更复杂。

这时候即使脚本按 90% PC 构造了 prompt，服务端日志里的 Prefix cache hit rate 也可能低一些，比如稳定在 85%。

这通常不一定是脚本构造错了，而可能是 MTP 下服务端对 prefix cache query / hit 的统计口径和普通 decode 不完全一致。

所以开启 MTP 时，建议这样判断：

- 先看脚本输出里的 Expected per-request PC hit (%)，确认客户端构造是否接近 90%。
- 再看服务端 metrics 的增量命中率。
- 如果还不确定，可以关掉 MTP 跑一组对照，看命中率是否更接近设定值。

简单说：

脚本里的 prefix_cache_hit = 客户端构造目标

服务端日志里的 Prefix cache hit rate = 服务端统计结果

两者不一定完全相等，尤其是在累计统计和 MTP 场景下。

## Warmup 与 PC 命中率说明（非必看）

脚本有三类 warmup。

### HTTP warmup

参数：
```bash
--http-warmup-requests 4
```

作用：

- 预热 HTTP 连接。
- 预热服务端基础请求路径。
- 不用于 prefix cache。

### Prefix warmup

参数：
```bash
--prefix-warmup-requests 1
--prefix-warmup-output-length 1
```

作用：

- 每个 input length 和 cache hit 组合先发一小轮请求。
- 请求使用同一个共享 prefix。
- 目的是让服务端缓存目标 prefix。

### Case warmup

参数：
```bash
--case-warmup-repeats 1
--case-warmup-request-multiplier 1
```

作用：

- 每个正式 case 前，用同输入、同输出、同并发先跑一小轮。
- 不计入最终 CSV。
- 有助于降低 CUDA graph、JIT、调度路径首次命中的波动。

如果机器时间紧，可以先关掉：
```bash
--case-warmup-repeats 0
```

如果 TTFT 波动明显，建议打开：
```bash
--case-warmup-repeats 1
```

如果需要把 warmup 结果也保存到 JSON：
```bash
--save-warmup-results
```

注意：warmup 结果默认不写入正式 CSV，避免污染正式统计。

### Prefix cache 命中率构造方式

脚本不是让所有 prompt 完全相同，而是：
```plaintext
prompt = shared_prefix + unique_random_suffix
```

同一个 input length 下，不同 BS 默认复用同一个共享前缀：
```plaintext
input=65536, prefix_len=64512
bs1/bs2/bs3/... 使用同一组 shared_prefix
```

每条请求的 suffix 不同，所以不会变成整条 prompt 100% 相同。

这样做的原因：

- 共享前缀用于制造目标 PC 命中率。
- 随机后缀避免整条 prompt 完全相同导致接近 100% 命中。
- 跨 BS 复用同一个共享前缀，可以让不同 BS 对比更公平；如果每个 BS 换 prefix，后一个 BS 会重新承担新 prefix 的冷 miss，TTFT 会被污染。

### 为什么服务端 Prefix cache hit rate 可能不是刚好 90%

对于 64k/90%：
```plaintext
input = 65536
prefix_len = 64512
单请求理论 PC 命中率 = 64512 / 65536 = 90%
```

服务端日志里的：
```plaintext
Prefix cache hit rate
```

通常是累计或近似累计口径，不一定是当前 case 的单请求独立命中率。

例如：
```plaintext
64k input + 1k output + 90% PC hit
concurrency-list 1 2 3
request-multiplier 8
case-warmup-repeats 1
```

长请求数量大致是：
```plaintext
prefix warmup: 1
bs1 case warmup: 1
bs1 正式: 8
bs2 case warmup: 2
bs2 正式: 16
bs3 case warmup: 3
bs3 正式: 24
总长请求: 55
```

第一个 prefix warmup 请求通常是 miss，后续 54 个长请求命中 90% prefix。因此服务端累计命中率理论上接近：
```plaintext
54 * 64512 / (55 * 65536) = 88.36%
```

如果服务端日志看到：
```plaintext
Prefix cache hit rate: 88.3%
```

这和理论累计值匹配，不说明 prompt 构造失败。

你这次后续日志里看到：
```plaintext
Prefix cache hit rate: 89.7%
```

也属于正常范围，说明 PC 命中率基本符合预期。

## 输出指标和 CSV 字段说明（非必看）

脚本运行时，每个 warmup 和正式 bench case 都会打印类似 `vllm bench serve` 的分段结果块。

示例：
```plaintext
============ Serving Benchmark Result ============
Phase:                                    bench
Mode:                                     fixed
SLA pass:                                 PASS
Cache hit target (%):                     90.00
Expected per-request PC hit (%):          90.00
Prefix length:                            64512
Input length:                             65536
Output length:                            1024
Successful requests:                      8
Failed requests:                          0
Maximum request concurrency:              1
Benchmark duration (s):                   122.13
Total input tokens:                       573440
Total generated tokens:                   8192
Request throughput (req/s):               0.07
Output token throughput (tok/s):          67.07
Total token throughput (tok/s):           4762.27
---------------Time to First Token----------------
Mean TTFT (ms):                           2916.17
Median TTFT (ms):                         2915.38
P95 TTFT (ms):                            2936.81
P99 TTFT (ms):                            2939.60
-----Time per Output Token (excl. 1st token)------
Mean TPOT (ms):                           12.07
Median TPOT (ms):                         12.07
P95 TPOT (ms):                            12.15
P99 TPOT (ms):                            12.16
---------------Inter-token Latency----------------
Mean ITL (ms):                            35.75
Median ITL (ms):                          35.75
P95 ITL (ms):                             35.78
P99 ITL (ms):                             35.78
==================================================
```

结果文件：
```plaintext
<timestamp>_prefix_cache_benchmark.csv
<timestamp>_prefix_cache_benchmark.json
```

CSV 重点字段：

<sheet token="IadPsyvzAh8v4MtHz3LcuJesnBb_oPDkWp"/>

###

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/并发|并发]]（metric）
- [[飞书知识图谱/实体/CUDA Graph|CUDA Graph]]（concept）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-4.7-W8A8|GLM-4.7-W8A8]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/Prefix Cache|Prefix Cache]]（concept）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/SLA|SLA]]（metric）
- [[飞书知识图谱/实体/Speculative Decoding|Speculative Decoding]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

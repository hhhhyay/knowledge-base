---
title: 记录
tags:
  - AI
  - LLM
  - Benchmark
model:
  - deepseek-ai
  - DeepSeek-R1-awq
  - DeepSeek-R1-Distill-Qwen-14B
  - DeepSeek-V3
  - Qwen3-8B
  - qwen3-8b-fp16
framework:
  - OpenCompass
  - SGLang
  - vLLM
issue_type: []
source: feishu
feishu_token: PIadwOlFJize85kedzhcTM1hn0e
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/PIadwOlFJize85kedzhcTM1hn0e
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 记录

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/PIadwOlFJize85kedzhcTM1hn0e)

#### git代理加速
```plaintext
https://ghproxy.net/
https://ghp.ci/ 
https://gh-proxy.com/
https://ghproxy.homeboyc.cn/
https://github.ur1.fun/
https://github.moeyy.xyz/
https://gitdl.cn/
```


```plaintext
#豆瓣：
-i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
#清华：
-i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
#网易：
-i https://mirrors.163.com/ --trusted-host mirrors.163.com
#阿里云：
-i https://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
-i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
(实在不行的话，改成http)
#中科大：
-i https://pypi.mirrors.ustc.edu.cn/simple/ --trusted-host pypi.mirrors.ustc.edu.cn

```


#### 下载快传

wget https://ksefile.hpccube.com:65241/efile/share/fasttrans-client.tar.gz

#### 下载大模型

sudo apt-*get* *update*

sudo apt-get install git-lfs

git lfs clone [http://113.200.138.88:18080/aimodels/findsource-dependency/wenet_onnxruntime.git](http%3A%2F%2F113.200.138.88%3A18080%2Faimodels%2Ffindsource-dependency%2Fwenet_onnxruntime.git)

export HF_ENDPOINT=https://hf-mirror.com

huggingface-cli download   --resume-download  deepseek-ai/DeepSeek-V3  --local-dir DeepSeek-V3

使用 `aria2c` 下载单个权重：

apt-get update && apt-get install aria2
```bash {wrap}
    # 将 <你的直链> 替换为你刚刚复制的链接  表示使用 16 个连接来下载，可以显著提高速度
    aria2c -x 16 -s 16 "<你的直链>"   
```

aria2c -x 16 -s 16 https://modelscope.cn/models/cognitivecomputations/DeepSeek-R1-awq/resolve/master/model-00042-of-00074.safetensors
```plaintext {wrap}
wget https://hf-cdn.sufy.com/hfd/hfd.sh
chmod a+x hfd.sh
export HF_ENDPOINT=https://hf-cdn.sufy.com
./hfd.sh deepseek-ai/DeepSeek-R1-Distill-Qwen-14B --tool aria2c -x 4
```

下载脚本：

aria2.sh，
```plaintext {wrap}
model=$1

apt update
apt install aria2 -y
export HF_ENDPOINT=https://hf-cdn.sufy.com
bash ./hfd.sh $1
#bash ./hfd.sh $1 --dataset

```

测试使用【例如下载DeepSeek-R1-Distill-Qwen-14B】
```plaintext {wrap}
bash aria2.sh deepseek-ai/DeepSeek-R1-Distill-Qwen-14B
```

#### 异地拉代码

123.235.52.21:32620

git clone -b v0.4.1 http://10.6.10.68/dcutoolkit/deeplearing/opencompass.git  

git clone -b v0.4.1 http://123.235.52.21:32620/dcutoolkit/deeplearing/opencompass.git  

[[飞书知识图谱/资源/如何在异地集群push代码到青岛(10.6.10.68)--cJDTlncg|如何在异地集群push代码到青岛(10.6.10.68)]]

#### 查看进程完整版

 ps -ax | grep PID

kill -9 

#### Git pull报错

git stash

<image token="Zb5AbnHtUoT6koxKcLyccP5qn4b" width="771" height="271" align="center"/>

#### 过滤报错

<image token="IcEUb2b99o2PKwxPIapcHJyjnNp" width="1093" height="53" align="center"/>

使用grep -a命令即可

#### 批量解压zip文件

<image token="X9Ejb7AIxopXZCxijDNcwoO8nxc" width="1655" height="581" align="center"/>

cd /**path**/**to**/directory && find . -**name** "*.zip" | xargs -n 1 unzip -d .

查看隐藏文件夹：ls -al

#### 查找文件夹最新的文件

find /path/to/directory -type f -printf '%TY-%Tm-%Td %p\n' | sort -r | head -n 1

#### 下载frpc_linux_amd64_v0.2

git clone https://gitcode.com/open-source-toolkit/62605.git

torch.cuda.is_available()

#### 查看谁在用卡

<image token="LFWyb4KERoxGMRxwOkucqkY1nTg" width="1111" height="547" align="center"/>

1、hy-smi --showpids 在容器里

2、cat /proc/35921/cgroup

3、docker ps -a | grep 61336a0

####  大模型gemm分析

1、添加环境变量
```python {wrap}
export ROCBLAS_LAYER=3     # 打印gemmsize
export TENSILE_DB=0x8000   # 打印gemm使用的kernel
```

2、执行命令前添加hipprof
```python
hipprof python test.py
```

运行完成后会生成result_xxx_kernel.csv，根据该csv文件，可以分析出kernel占比，根据高占比kernel找出对应的gemmsize。

3、gemm单测
```bash
cd /opt/dtk/rocblas/lib/rocblas/benchmark_tool
chmod +x rocblas-bench
测试size如:
./rocblas-bench -f gemm_ex --transposeA N --transposeB N -m 12288 -n 1 -k 4096 --alpha 1 --a_type f16_r --lda 12288 --b_type f16_r --ldb 4096 --beta 0 --c_type f16_r --ldc 12288 --d_type f16_r --ldd 12288 --compute_type f32_r --algo 0 --solution_index 0 --flags 0

```

分段分析（hipprof&torchprof）可参考：[[飞书知识图谱/资源/vllm-prof--cDCBFnkg|vllm-prof]]

#### 文件内容分割

log文件第100行之后的内容提取出来放到另一个文件中
```plaintext {wrap}
tail -n +101 input.log > output.log
```

`-n +101` 表示从第101行开始输出（即跳过前100行）
```plaintext {wrap}
awk 'NR>100' input.log > output.log
```

NR表示行号，NR>100会打印100行之后的内容

#### 禅道用户开通

禅道地址：http://hpczentao.sugon.com/my/

用户名是邮箱前缀，密码是用户名（首字母大写）+123

比如：liuyr1@sugon.com

用户名：liuyr1

密码：Liuyr1123

#### grep命令
```json {wrap}
# 当前目录递归搜索，只显示文件名
grep -rlF 'need to load: -' .
grep -rilF 'need to load: -' . #不区分大小写

# 若只想当前层（不递归）
grep -lF 'need to load: -' *
```

```plaintext {wrap}
查找包含环境变量的所有shell
grep -rl 'SGLANG_USE_FUSED_RMS_QUANT_PATH' . --include='*.sh' 

查找包含环境变量的所有shell并进行替换
grep -rl 'SGLANG_USE_FUSED_RMS_QUANT_PATH' . --include='*.sh' |  xargs sed -i 's/SGLANG_USE_FUSED_RMS_QUANT_PATH/SGLANG_USE_RMS_QUANT_PATH/g'
```


```bash {wrap}
# 查找当前目录下所有 .py 文件中包含 "import torch" 的文件
find . -maxdepth 2 -name "*.py" -exec grep -l "import torch" {} +

```

**SLA要求：Mean_TTFT≤3s；MeanTPOT≤50ms；**

客户端测试脚本如下：
```bash {wrap}
model_path=/model/DeepSeek-R1-awq

model=${model_path##*/}
tp=8
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="./client/${model}-${mode}-$time-nopc-nocp"

data_type="awq"
port=8001

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

pairs=("512 512")

echo "data_type,batch,prompt_tokens,completion_tokens,Output_token_throughput(tok/s),Total_Token_throughput(tok/s),Mean_TTFT(ms),Mean_TPOT(ms),Mean_throughput,duration" > $all_log


#for batch in $(seq 60 1 65) ; do
for batch in 1 2 4 8 16 32 64 75 77 79 80 81 83 85 87 89 91 93 95; do
    for pair in "${pairs[@]}"; do
        prompt_tokens=${pair%% *}
        completion_tokens=${pair#* }
        vllm bench serve \
        --model ${model_path} \
        --random-input-len $prompt_tokens \
        --random-output-len $completion_tokens \
        --port $port \
        --dataset-name random \
        --num-prompts $batch --max-concurrency $batch --ignore-eos  2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log

        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total Token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`

        Mean_TTFT=`grep -a "^Mean TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`

        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`

        Mean_throughput=$(echo "scale=2; 1000/$Mean_TPOT" | bc -l)

        echo "$data_type,$batch,$prompt_tokens,$completion_tokens,$Output_token_throughput,$Total_Token_throughput,$Mean_TTFT,$Mean_TPOT,$Mean_throughput,$Benchmark_duration">> $all_log

     done
done


cat $all_log


```


```bash
export HIP_VISIBLE_DEVICES=0
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_ZERO_OVERHEAD=1 # zero



model_path=/model/qwen3/Qwen3-8B
model=${model_path##*/}
tp=1 #K100AI为8
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="./das17-1025/bw21/qwen3-8b-fp16"
data_type="fp16"
port=8021
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

vllm serve $model_path --host 0.0.0.0 --port $port \
        --dtype float16 --tensor-parallel-size $tp \
        --trust-remote-code \
        --max-num-seqs 1024 \
        --max-seq-len-to-capture 40960  \
        --distributed-executor-backend=mp --no-enable-prefix-caching \
        --no-enable-chunked-prefill  2>&1 | tee  ${logpath}-$time.log

```


```bash
model_path=/model/qwen3/Qwen3-8B

model=${model_path##*/}
tp=1
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="./client-512/bw21/${model}-${mode}-$time"
data_type="fp16"
port=8021
csv_name="${model}-tp${tp}-$time"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"


echo "input,output,request_rate,num_prompts,concurrency,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,mean_ttft_ms,p95_ttft_ms,p99_ttft_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log


batch_seq=$({
  seq 1 1 64
  seq 64 2 256
  seq 256 4 512
  seq 512 8 1024
})

pairs=(
"512 4096 256"
)

for pair in "${pairs[@]}"; do
  prompt_tokens=$(echo "$pair" | awk '{print $1}')
  completion_tokens=$(echo "$pair" | awk '{print $2}')
  max_bs=$(echo "$pair" | awk '{print $3}')

       for batch in $batch_seq;do
        [[ $batch -gt $max_bs ]] && break

        vllm bench serve \
        --model ${model_path} \
        --random-input-len $prompt_tokens \
        --random-output-len $completion_tokens \
        --port $port \
        --metric-percentiles 95,99 \
        --dataset-name random \
        --num-prompts $batch --max-concurrency $batch  --ignore-eos  2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log

        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total Token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        request_rate=`grep -a "^Traffic request rate" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`


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

        echo "$prompt_tokens,$completion_tokens,$request_rate,$batch,$batch,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log
     done
done

```

## 关联资源

- [[飞书知识图谱/资源/如何在异地集群push代码到青岛(10.6.10.68)--cJDTlncg|如何在异地集群push代码到青岛(10.6.10.68)]] · 飞书源链接不可用
- [[飞书知识图谱/资源/vllm-prof--cDCBFnkg|vllm-prof]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/deepseek-ai|deepseek-ai]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-awq|DeepSeek-R1-awq]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Qwen-14B|DeepSeek-R1-Distill-Qwen-14B]]（model）
- [[飞书知识图谱/实体/DeepSeek-V3|DeepSeek-V3]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OpenCompass|OpenCompass]]（framework）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/Qwen3-8B|Qwen3-8B]]（model）
- [[飞书知识图谱/实体/qwen3-8b-fp16|qwen3-8b-fp16]]（model）
- [[飞书知识图谱/实体/RPS|RPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/SLA|SLA]]（metric）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

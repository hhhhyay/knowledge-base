---
title: opencompass精度测试
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-awq-mtp
  - DeepSeek-R1-Distill-Llama-70B
  - DeepSeek-R1-Distill-Qwen-32B
  - DeepSeek-R1-Qwen-32B
  - Llama-33-70B-Instruct
  - Qwen3-30B-A3B-w8a8
framework:
  - OpenCompass
  - PyTorch
  - vLLM
issue_type:
  - 超时
source: feishu
feishu_token: MNxwwIk2HizPgfkyxbIcAib8nsm
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/MNxwwIk2HizPgfkyxbIcAib8nsm
last_synced_at: 2026-06-07T17:07:53.393Z
---

# opencompass精度测试

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/MNxwwIk2HizPgfkyxbIcAib8nsm)

## DCU环境

git clone -b 0.4.2 [http://10.16.6.30/dcutoolkit/deeplearing/opencompass.git](http%3A%2F%2F10.16.6.30%2Fdcutoolkit%2Fdeeplearing%2Fopencompass.git)

git clone -b 0.4.2 [http://112.11.119.99:10068](http%3A%2F%2F10.16.6.30%2Fdcutoolkit%2Fdeeplearing%2Fopencompass.git)[/dcutoolkit/deeplearing/opencompass.git](http%3A%2F%2F10.16.6.30%2Fdcutoolkit%2Fdeeplearing%2Fopencompass.git) （外网链接）

按照readme配置环境
```bash {wrap}
pip install -r requirements.txt
pip install -r requirements/api.txt
pip install -r requirements/extra.txt

#编译whl包并安装
python setup.py bdist_wheel 
cd dist
pip install opencompass*
```

### 数据集准备（部分数据集可以从界林文件夹copy下数据集/public/home/jujl1/data）
```plaintext
wget https://github.com/open-compass/opencompass/releases/download/0.2.2.rc1/OpenCompassData-core-20240207.zip
unzip OpenCompassData-core-20240207.zip (解压后为data目录)

#math500需要用下界林的数据
cp /public/home/jujl1/data/math/* data/math/


```

py配置文件参考（examples/vllm/ds32.py）：
```python {wrap}
from mmengine.config import read_base

with read_base():
#    from opencompass.configs.datasets.aime2024.aime2024_gen_6e39a4 import aime2024_datasets
    from opencompass.configs.datasets.gsm8k.gsm8k_gen_17d0dc import gsm8k_datasets
#    from opencompass.configs.datasets.mmlu.mmlu_gen_4d595a import mmlu_datasets
#    from opencompass.configs.datasets.gpqa.gpqa_openai_simple_evals_gen_5aeece import gpqa_datasets
#    from opencompass.configs.datasets.livecodebench.livecodebench_gen_6966bc import LCB_datasets
    from opencompass.configs.datasets.math.math_500_gen import math_datasets
#    from opencompass.configs.datasets.ceval.ceval_zero_shot_gen_bd40ef import ceval_datasets
    from opencompass.configs.datasets.humaneval.humaneval_gen import humaneval_datasets
    from opencompass.configs.summarizers.example import summarizer

datasets = sum([v for k, v in locals().items() if k.endswith("_datasets") or k == 'datasets'], [])
work_dir = '/mnt/gbj/distill/acc-0810/ds32-cuda-32k-bs32' #输出日志路径

from opencompass.models import OpenAISDK

api_meta_template = dict(round=[
    dict(role='HUMAN', api_role='HUMAN'),
    dict(role='BOT', api_role='BOT', generate=True),
], )

models = [
    dict(
        abbr='DeepSeek-R1-Qwen-32B', # 输出log中记录模型名称
        type=OpenAISDK,
        path='/model/DeepSeek-R1-Distill-Qwen-32B', # server中设置的模型名称，未设置同server中的模型路径
        openai_api_base="http://0.0.0.0:8000/v1", # api端口
        tokenizer_path="/model/DeepSeek-R1-Distill-Qwen-32B", # 模型路径，用于输入prompt准备
        key='EMPTY',
        meta_template=api_meta_template, # 对应chat验证，base请注释
        temperature=0,
        query_per_second=64,
        max_out_len=32768,
        max_seq_len=32768,
        pred_postprocessor=dict(
            type='opencompass.utils.text_postprocessors.extract_non_reasoning_content'),
        batch_size=32),
]

```

在线测试server按照正常性能测试起就行

例如：
```bash {wrap}
vllm serve /model/DeepSeek-R1-Distill-Qwen-32B --trust-remote-code  --dtype float16  -tp 4 --max-model-len 32768 --enforce-eager 
```

执行精度测试命令
```bash {wrap}
python run.py  examples/vllm/ds32.py
```

## NV环境
```bash {wrap}
docker run -it --network=host --name=acc  --privileged --device=/dev/kfd --device=/dev/dri --ipc=host  --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined  --ulimit stack=-1:-1 --ulimit memlock=-1:-1 --entrypoint bash --gpus all  -v `pwd`:/mnt  -v /public/DL_DATA/llm-models/:/model/ nvcr.io/nvidia/pytorch:24.10-py3 bash
```

安装依赖
```bash {wrap}
pip install vllm==0.9.2 -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

### Q&A {folded="true"}

### 例如ds70b-tp4使用cudagraph模式测试math500精度数据集时会由于超时连接不到客户端的情况如下：http://hpczentao.sugon.com/bug-view-102749.html

<image token="LlKTb9y3HoFBOfxKuC8cMZHNnwg" width="1780" height="716" align="center"/>

进行排查，2个步骤：

①先加-r在重新跑下看看是否是同一个batch报错，比如上图是在bs14报的错

比如：python run.py examples/vllm/ds70.py -r

看结果还是卡在同位置已经1h了

<image token="LZiIb4scro8OuZxE5ngcc9VQnre" width="1844" height="1031" align="center"/>

②vim opencompass/models/openai_api.py +601 在下面打印下input，添加print(repr(input))

<image token="SGgVbGMGLoHgAsxnLzJcXmhunfb" width="1000" height="337" align="center"/>

输出类似的列表

<image token="RoDHb7aiUooDHPxi87zcgWkSnJe" width="804" height="162" align="center"/>

将输出的batch内容替换下test_api.py（[[飞书知识图谱/资源/test_api.py--cM9Punsc|test_api.py]]）的data列表，修改下模型地址及最大长度即可，最大长度需小于设置的上下文长度

<image token="OOPlbVYQfofBrPxaYolcXVgtn7f" width="1534" height="315" align="center"/>

执行下脚本python test_api.py 2>&1 | tee test.log，生成log如下：最后一组请求超时而失败

<image token="GgYGbq4booPXywxBZR1cgXcinMl" width="1699" height="263" align="center"/>

在test_api.py里添加下timeout=7200，再试下

<image token="PJTBbhVp8o4Mb0xX3ovcUAsMnIg" width="1157" height="553" align="center"/>

由于eager可正常跑通精度测试，但cudagraph会超时，因此，eager和cudagraph都进行了测试验证bs15

发现最后输入的回答会一直重复，直至32k，server端表现不太一样，从下面结果看看出是cudagraph性能低，输出32k超过1h（opencompass机制为1h）从而出现客户端请求超时的问题，研发还在排查

<image token="PVDBb6pmao35UWxURQwc1bYsntc" width="1838" height="566" align="center"/>

<image token="Izngbrrs2oEIcFxHnTgcNjOBnng" width="1846" height="831" align="center"/>


```bash {wrap}
export VLLM_USE_GLOBAL_CACHE13=0
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export HSA_FORCE_FINE_GRAIN_PCIE=1 #K100AI需要
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4

export VLLM_WORKER_MULTIPROC_METHOD=spawn
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1
export HSA_FORCE_FINE_GRAIN_PCIE=1
export W4A16_MOE_CUDA=1
export DPSK_FP16_QUICK=0

tp=8
port=8001
model_path=/module/DeepSeek-R1-awq-mtp
model=${model_path##*/}
time=$(date "+%m%d-%H%M")
logpath="/mnt/gbj/distill/acc-0812/server/${model}-${mode}-$time-mtp-v1-nopc-nocp"
data_type="awq"
port=8001
csv_name="${model}-tp${tp}-$time"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

max_model_len=32768

vllm serve $model_path \
        --trust-remote-code --host 0.0.0.0 \
        --tensor-parallel-size $tp --dtype float16  --max-num-seqs 256 \
        --port $port -q awq_marlin --gpu-memory-utilization 0.9 --block-size 64  --disable-log-requests \
        --max-model-len $max_model_len --max-seq-len-to-capture $max_model_len --max-num-batched-tokens $max_model_len \
        --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 1}' \
        --no-enable-prefix-caching --no-enable-chunked-prefill --compilation-config '{"full_cuda_graph": true}'  $1 2>&1 | tee ${logpath}.log
```

```bash {wrap}
export VLLM_USE_GLOBAL_CACHE13=0
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_HOST_IP=13.13.6.22
export NCCL_SOCKET_IFNAME=ens73f1np1
export GLOO_SOCKET_IFNAME=ens73f1np1
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_IB_HCA=mlx5_0:1,mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_1:1,mlx5_8:1,mlx5_9:1
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export VLLM_RPC_TIMEOUT=1800000
export NCCL_TOPO_FILE="/mnt/newtopo.xml"
export VLLM_PCIE_USE_CUSTOM_ALLREDUCE=1  #k100ai需要
```


```bash {wrap}
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4
#model=${model_path##*/}
time=$(date "+%m%d-%H%M")
logpath="/mnt/gbj/distill/acc-0812/server/v1"
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

HIP_VISIBLE_DEVICES=0,1,2,3 vllm serve /model/DeepSeek-R1-Distill-Llama-70B --trust-remote-code  --dtype float16  -tp 4 --max-model-len 32768 --compilation-config '{"full_cuda_graph": true}' --port 8001 2>&1 | tee $logpath-ds-70.log  &
HIP_VISIBLE_DEVICES=4,5,6,7 vllm serve /model/DeepSeek-R1-Distill-Llama-70B --trust-remote-code  --dtype float16  -tp 4 --max-model-len 32768 --enforce-eager --port 8002 2>&1 | tee $logpath-ds-70-eager.log  &

```

```python {wrap}
from mmengine.config import read_base

with read_base():
    from opencompass.configs.datasets.gsm8k.gsm8k_gen_17d0dc import gsm8k_datasets
    from opencompass.configs.datasets.humaneval.humaneval_gen import humaneval_datasets
    from opencompass.configs.datasets.math.math_500_gen import math_datasets
    from opencompass.configs.summarizers.example import summarizer

datasets = sum([v for k, v in locals().items() if k.endswith("_datasets") or k == 'datasets'], [])
work_dir = '/mnt/gbj/distill/acc-0812/ds70-cuda-32k-bs32' #输出日志路径

from opencompass.models import OpenAISDK

api_meta_template = dict(round=[
    dict(role='HUMAN', api_role='HUMAN'),
    dict(role='BOT', api_role='BOT', generate=True),
], )

models = [
    dict(
        abbr='DeepSeek-R1-Distill-Llama-70B', # 输出log中记录模型名称
        type=OpenAISDK,
        path='/model/DeepSeek-R1-Distill-Llama-70B', # server中设置的模型名称，未设置同server中的模型路径
        openai_api_base="http://0.0.0.0:8001/v1", # api端口
        tokenizer_path="/model/DeepSeek-R1-Distill-Llama-70B", # 模型路径，用于输入prompt准备
        key='EMPTY',
        meta_template=api_meta_template, # 对应chat验证，base请注释
        temperature=0,
        query_per_second=64,
        max_out_len=32768,
        max_seq_len=32768,
        pred_postprocessor=dict(
            type='opencompass.utils.text_postprocessors.extract_non_reasoning_content'),
        batch_size=32),
]

```

```bash {wrap}
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_RPC_TIMEOUT=1800000
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=1
export VLLM_RANK1_NUMA=1
HIP_VISIBLE_DEVICES=2,3 vllm serve /model/Qwen3-30B-A3B-w8a8 --host 0.0.0.0 --port 9092 --dtype bfloat16 --tensor-parallel-size 2 --compilation-config '{"full_cuda_graph": true}' --max-num-batched-tokens 8192 --no-enable-prefix-caching 
```



```bash {wrap}
export HIP_VISIBLE_DEVICES=0,1,2,3
# NUMA绑定，提高RCCL稳定性
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
#export VLLM_RANK4_NUMA=1
#export VLLM_RANK5_NUMA=1
#export VLLM_RANK6_NUMA=1
#export VLLM_RANK7_NUMA=1
# 同流推理，已默认启用
export ALLREDUCE_STREAM_WITH_COMPUTE=1
# 通信调优参数
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
# 零消耗调度
export VLLM_ZERO_OVERHEAD=1
export VLLM_RPC_TIMEOUT=1800000

#V1.9.0c版本驱动需要
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_P2P_NCHANNELS=4
export NCCL_MAX_NCHANNELS=16
export NCCL_MAX_P2P_NCHANNELS=4
export NCCL_NCHANNELS_PER_PEER=4
export NCCL_RINGS="0 1 3 2 4 6 5 7"
export NCCL_P2P_NVL_CHUNKSIZE=131072

model_path=/model/llama3.3/Llama-33-70B-Instruct
model=${model_path##*/}
tp=4 #K100AI为8
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
logpath="./server/${model}-${mode}-$time-v0-nopc-nocp-cuda-zero"
data_type="fp16"
csv_name="${model}-tp${tp}-$time"
port=8089
if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi


VLLM_USE_V1=0 vllm serve $model_path \
        --host 0.0.0.0 --port $port --dtype float16 \
        --tensor-parallel-size $tp  \
        --max-model-len 32768 \
        --max-seq-len-to-capture 32768 2>&1 | tee  $logpath.log

```

```bash {wrap}
./kunlun-benchmark vllm server  --prompt_type normal_distribution  --min_input_len ${min_input_len}  --max_input_len ${max_input_len} --max_output_len 500  --min_output_len 300  --concurrency ${concurrency}  --query_num $query_num --port ${port}   --tp ${tp} --model_path ${model_path}    --result_dir ${logdir}/result-md/  --work_mode manual 
```

## 关联资源

- [[飞书知识图谱/资源/test_api.py--cM9Punsc|test_api.py]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/file/KxNjbCyF6oJNsoxqe52cM9Punsc)

## 关联知识

- [[飞书知识图谱/实体/超时|超时]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-awq-mtp|DeepSeek-R1-awq-mtp]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Llama-70B|DeepSeek-R1-Distill-Llama-70B]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Qwen-32B|DeepSeek-R1-Distill-Qwen-32B]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Qwen-32B|DeepSeek-R1-Qwen-32B]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/Llama-33-70B-Instruct|Llama-33-70B-Instruct]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OpenCompass|OpenCompass]]（framework）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）
- [[飞书知识图谱/实体/Qwen3-30B-A3B-w8a8|Qwen3-30B-A3B-w8a8]]（model）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

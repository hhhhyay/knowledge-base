---
title: opencompass精度测试方法（DCU）
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Distill-Qwen-32B
  - DeepSeek-R1-Qwen-32B
framework:
  - OpenCompass
  - vLLM
issue_type: []
source: feishu
feishu_token: VADRwyctUiLkvykzRN6cfV7rn3c
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/VADRwyctUiLkvykzRN6cfV7rn3c
last_synced_at: 2026-06-07T17:07:53.393Z
---

# opencompass精度测试方法（DCU）

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/VADRwyctUiLkvykzRN6cfV7rn3c)

## DCU环境

<view type="1">

  <file token="FG1IbebTdouPFfxxH2NccPHhnIf" name="opencompass.zip"/>

</view>

按照readme配置环境
```bash {wrap}
unzip opencompass.zip
cd ./opencompass

pip install -r requirements.txt
pip install -r requirements/api.txt
pip install -r requirements/extra.txt

#编译whl包并安装
python setup.py bdist_wheel 
cd dist
pip install opencompass*
```

### 数据集准备
```plaintext
wget https://github.com/open-compass/opencompass/releases/download/0.2.2.rc1/OpenCompassData-core-20240207.zip
unzip OpenCompassData-core-20240207.zip (解压后为data目录)

```

<callout emoji="round_pushpin" background-color="light-orange" border-color="light-orange">

注意，测试gsm8k和math500数据集需要配置相关文件，指定数对应数据集路径:

./opencompass/opencompass/configs/datasets/gsm8k/gsm8k_gen_17d0dc.py

./opencompass/opencompass/configs/datasets/math/math_500_gen.py
</callout>

### py配置文件参考（ds32.py）：

#日常测试gsm8k，math_500_gen，humaneval三个数据集即可
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
        temperature=0, # 温度默认为0
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
python run.py  ds32.py
```

此工具支持断点续测
```shell {wrap}
#./20260317_141138为实际路径
Python run.py -r ./20260317_141138  examples/vllm/ds32.py
```

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-Distill-Qwen-32B|DeepSeek-R1-Distill-Qwen-32B]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-Qwen-32B|DeepSeek-R1-Qwen-32B]]（model）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/OpenCompass|OpenCompass]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

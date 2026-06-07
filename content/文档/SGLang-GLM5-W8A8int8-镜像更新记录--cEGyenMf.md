---
title: SGLang-GLM5-W8A8int8-镜像更新记录
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-V3
  - GLM-5-W8A8
  - GLM5-W8A8int8
  - MiniMax-M2.5-W8A8
framework:
  - Mooncake
  - OpenCompass
  - SGLang
issue_type:
  - 精度问题
source: feishu
feishu_token: GnrpwZEeoiHNTRkpwjycEGyenMf
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/GnrpwZEeoiHNTRkpwjycEGyenMf
last_synced_at: 2026-06-07T17:07:53.393Z
---

# SGLang-GLM5-W8A8int8-镜像更新记录

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/GnrpwZEeoiHNTRkpwjycEGyenMf)

## 代码分支：v0.5.10rc0_dev_maxiao

## 代码仓地址：https://developer.sourcefind.cn/codes/OpenDAS/sglang

## 测试脚本 & 服务端启动命令

### Prefill 性能测试脚本

#### 使用接口  **/v1/completions**

##### 脚本代码 prefill_throughput.py
```python {wrap}
import asyncio
import aiohttp
import json
import time
import random
import argparse
from datetime import datetime
import tqdm
import math
from tqdm.asyncio import tqdm_asyncio

TIMESTAMP_FORMAT = "%Y-%m-%d-%H-%M-%S"

time_stamp = datetime.now().strftime(TIMESTAMP_FORMAT)

tokenizer = None

global_prefix_ids: list[int] | None = None

def create_prompts(input_len: int, num_req: int=1, prefix_len: int=0, prefix_ids: list[int] | None = None) -> list[list[int]]:
    if prefix_len > 0:
        assert input_len >= prefix_len
        if prefix_ids is None:
            prefix_ids = [random.randint(42, 10000) for _ in range(prefix_len)]
        else:
            assert len(prefix_ids) == prefix_len
    else:
        prefix_ids = []

    return [prefix_ids + [random.randint(42, 10000) for _ in range(input_len-prefix_len)] for _ in range(num_req)]

def detokenize_to_str(token_id_list: list[int]) -> str:
    from transformers import AutoTokenizer
    global tokenizer
    if tokenizer == None:
        assert args.tokenizer_path != ''
        tokenizer = AutoTokenizer.from_pretrained(args.tokenizer_path)

    return tokenizer.decode(token_id_list)

def completion_data(model: str, base_url: str, prompt: list[int], output_len: int=10):
    #if args.str_input:
    prompt = detokenize_to_str(prompt)
    data_template = {
        "model": model,
        "stream": True,
        "max_tokens": output_len,
        "temperature": 0.6,
        "prompt": prompt,
        "top_p": 0.95,
        # "ignore_eos": True,
        # "stream_options" : {
        #     "include_usage": True},
    }
    url = f"{base_url}/v1/completions"
    return data_template, url
    # prompt_key = "input_ids"
    # if args.str_input:
    #     prompt = detokenize_to_str(prompt)
    #     prompt_key = "text"   
    # data_template = {
    #     "stream": True,
    #     prompt_key: prompt,
    #     "sampling_params": {
    #         "max_new_tokens": output_len,
    #         "temperature": 0.6,
    #         "top_p": 0.95,
    #         "ignore_eos": True,
    #     },
    # }
    # url = f"{base_url}/generate"
    # return data_template, url    
    
async def single_task(prompt_ids: list[int], output_len: int = 10, if_print: bool = True):

    data_template, url = completion_data(args.model, args.base_url, prompt=prompt_ids, output_len=output_len)

    t_start = time.time()    
    timeout = aiohttp.ClientTimeout(total=6000)
    connector = aiohttp.TCPConnector(limit=0)  # 连接数
    first_token = True
    last_time = t_start
    ttft = 0
    tpot = []
    async with aiohttp.ClientSession(connector=connector, timeout=timeout) as session:
        async with session.post(url, json=data_template) as response:
            chunks = []
            async for chunk, _ in response.content.iter_chunks():
                if first_token:
                    last_time = time.time()
                    ttft = last_time - t_start
                    first_token = False
                else:
                    current_time = time.time()
                    tpot.append(current_time - last_time)
                    last_time = current_time
                chunks.append(chunk)
            output = b"".join(chunks).decode("utf-8")
    
    if if_print:
        print(f"prompt_len: {len(prompt_ids)}, ttft: {ttft*1000:.6f} ms", flush=True)
    
    return ttft, tpot

async def prefill_benchmark():

    # bench
    prompt_len_list = []
    assert args.max_input_length_k != 0
    min_input_len = int(args.min_input_length_k*1024)
    max_input_len = args.max_input_length_k*1024
    max_step_len = args.max_input_length_step_k*1024

    prompt_len = min_input_len
    while prompt_len < max_input_len-100:
        prompt_len_list.append(prompt_len)
        if prompt_len <= max_step_len:
            prompt_len *= 2
        else:
            prompt_len += max_step_len
    prompt_len_list.append(max_input_len-100)

    global global_prefix_ids
    global_prefix_ids = create_prompts(max_input_len)[0]

    if args.prefix_cache:
        prefix_ratio_list = [0, 0.2, 0.4, 0.6, 0.8]
        # warmup prefix
        prompt_len = max_input_len - 100
        print(f"warmup prefix start ...", flush=True)
        tasks = [
            asyncio.create_task(
                single_task(
                    global_prefix_ids[:prompt_len], 
                    1,
                    if_print=False,
                )
            ) for _ in range(args.dp_size*2)
        ]
        results = await asyncio.gather(*tasks)
        print(f"warmup prefix finished", flush=True)
    else:
        prefix_ratio_list = [0]

    chunk_size = args.chunk_size_k * 1024

    save_path = f"{args.save_path}/{time_stamp}_prefill_throughput_results.json"
    result_dict = {
        "single_req": {},
        "concurrency": {},
    }
    

    # test single concurrency (only for dp_size == 1)
    for prefix_ratio in prefix_ratio_list:
        print(f"\nPrefix Ratio: {prefix_ratio:.2f}", flush=True)
        if args.dp_size == 1:
            throughput_list = []
            for prompt_len in prompt_len_list:
                throughput = await eval_normal(prompt_len=prompt_len, prefix_ratio=prefix_ratio)
                throughput_list.append(throughput)

            result_dict["single_req"][prefix_ratio] = {
                "prompt_len_list": prompt_len_list, 
                "throughput_list": throughput_list
            }

            with open(save_path, "w") as f:
                json.dump(result_dict, f, indent=4)
            print(f"save result dict to {save_path}")

            if args.plot:
                plot_results(result_dict)

        if args.concurrency:
            # test concurrency (all dp_size)
            finished_prompt_len_list = []
            if len(prompt_len_list) > 0:
                throughput_list = []
                for prompt_len in prompt_len_list:
                    concurrency = math.ceil(chunk_size / (prompt_len*(1-prefix_ratio)))
                    throughput = await eval_concurrency(
                        prompt_len=prompt_len,
                        concurrency=concurrency,
                        workload=5 if concurrency > 1 else 2,
                        prefix_ratio=prefix_ratio,
                    )
                    throughput_list.append(throughput)
                    finished_prompt_len_list.append(prompt_len)
                
                    result_dict['concurrency'][prefix_ratio] = {
                        "prompt_len_list": finished_prompt_len_list, 
                        "throughput_list": throughput_list
                    }

                    with open(save_path, "w") as f:
                        json.dump(result_dict, f, indent=4)
                    print(f"save result dict to {save_path}")

                    if args.plot:
                        plot_results(result_dict)

async def eval_normal(prompt_len: int, prefix_ratio: float) -> float:
    print(f"prompt_len: {prompt_len} start eval single request throughput")
    prefix_len = int(prompt_len * prefix_ratio)
    num_repeats = args.num_repeats
    tmp_throughput_list = []

    prefix_prompt = global_prefix_ids[:prefix_len]

    prompt_ids_list = create_prompts(
        prompt_len, 
        num_req=num_repeats+1, 
        prefix_len=prefix_len,
        prefix_ids=prefix_prompt,
    )
    #warmup
    ttft, _ = await single_task(prompt_ids_list[-1], if_print=False)
    for prompt_ids in prompt_ids_list[:-1]:
        ttft, _ = await single_task(prompt_ids)
        throughput = (prompt_len-prefix_len)/ttft
        tmp_throughput_list.append(throughput)
    avg_throughput = sum(tmp_throughput_list) / len(tmp_throughput_list)
    return avg_throughput

async def eval_concurrency(
    prompt_len: int, 
    concurrency: int, 
    workload: int=5,
    prefix_ratio: float=0,
) -> float:
    print(f"prompt_len: {prompt_len} start eval concurrency throughput")
    dp_size = args.dp_size
    num_reqs = concurrency * workload * dp_size

    prefix_len = int(prompt_len * prefix_ratio)
    prefix_prompt = global_prefix_ids[:prefix_len]

    prompt_ids_list = create_prompts(
        prompt_len, 
        num_req=num_reqs,
        prefix_len=prefix_len,
        prefix_ids=prefix_prompt,
    )

    t_start = time.time()
    tasks = [
        asyncio.create_task(
            single_task(prompt_ids, output_len=1, if_print=False)
        ) for prompt_ids in prompt_ids_list
    ]
    results = await tqdm_asyncio.gather(*tasks, desc=f"eval concurrency throughput, prompt_len: {prompt_len}")
    t_end = time.time()
    t_duration = t_end - t_start

    throughput = (prompt_len-prefix_len) * num_reqs / t_duration
    
    print(f"prompt_len: {prompt_len}, throughput: {throughput:.2f} tokens / s", flush = True)
    return throughput

def plot_results(result_dict: dict, save_path: str | None = None):
    import matplotlib.pyplot as plt
    import numpy as np

    plt.figure(figsize=(10, 5))
    for prefix_ratio, results in result_dict["single_req"].items():
        prompt_len_list = results["prompt_len_list"]
        throughput_list = results["throughput_list"]
        compute_len_list = np.array(prompt_len_list) * (1-float(prefix_ratio))

        plt.plot(compute_len_list, throughput_list, marker='o', label=f"Single Request, Prefix={prefix_ratio}")

    for prefix_ratio, results in result_dict["concurrency"].items():
        prompt_len_list = results["prompt_len_list"]
        throughput_list = results["throughput_list"]
        compute_len_list = np.array(prompt_len_list) * (1-float(prefix_ratio))
        
        plt.plot(compute_len_list, throughput_list, marker='o', label=f"Concurrency, Prefix={prefix_ratio}")

    plt.ylim(bottom=0)
    if args.x_log_scale:
        plt.xscale('log')
    plt.xlabel('New Compute Length')
    plt.ylabel('Throughput (tokens/s)')
    plt.title('Prefill Throughput Benchmark')
    plt.legend(bbox_to_anchor=(0.5, 1.05), loc='lower center', ncol=4)

    plt.tight_layout(pad=0.1)
    plt.grid(True)
    if save_path == None:
        save_path = f"{args.save_path}/{time_stamp}_prefill_throughput.png"
    print(f"save result figure to {save_path}")
    plt.savefig(save_path)

def replot():
    assert args.result_path != ''
    with open(args.result_path, "r") as f:
        result_dict = json.load(f)

    suffix = '_prefill_throughput_results.json'
    if args.result_path.endswith(suffix):
        prefix = args.result_path.rstrip(suffix)
    else:
        prefix = f"{args.save_path}/{time_stamp}"
    new_save_path = f"{prefix}_prefill_throughput.png"
    plot_results(result_dict=result_dict, save_path=new_save_path)
    

if __name__ == '__main__':
    argparser = argparse.ArgumentParser(description='Prefill Throughput Benchmark')

    argparser.add_argument(
        '--base-url',
        type=str,
        default='http://127.0.0.1:30002',
    )

    argparser.add_argument(
        '--model',
        type=str,
        default='DeepSeek-V3',
    )

    argparser.add_argument(
        '--min-input-length-k',
        type=float,
        default=1,
        help='min input length test, '
        '--min-input-length-k 1 means 1024.'
    )

    argparser.add_argument(
        '--max-input-length-k',
        type=int,
        default=0,
        help='max input length test, '
        '--max-input-length-k 128 means 128k.'
    )

    argparser.add_argument(
        '--max-input-length-step-k',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--chunk-size-k',
        type=int,
        default=8,
    )

    argparser.add_argument(
        '--num-repeats',
        type=int,
        default=3,
    )

    argparser.add_argument(
        '--prefix-cache',
        action=argparse.BooleanOptionalAction,
        default=True,
    )

    argparser.add_argument(
        '--concurrency',
        action=argparse.BooleanOptionalAction,
        default=True,
    )

    argparser.add_argument(
        '--dp-size',
        type=int,
        default=1,
    )

    argparser.add_argument(
        '--str-input',
        action=argparse.BooleanOptionalAction,
        default=False,
        help='Default we use token_ids_list as prompt. '
        'If inference engine does not support token_ids_list as input, '
        'we can set --str-input and we will detokenizer token_ids_list to str.'
    )

    argparser.add_argument(
        '--tokenizer-path',
        type=str,
        default='',
    )

    argparser.add_argument(
        '--plot',
        action=argparse.BooleanOptionalAction,
        default=True,
    )

    argparser.add_argument(
        '--x-log-scale',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--save-path',
        type=str,
        default="./",
    )

    argparser.add_argument(
        '--replot',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--result-path',
        type=str,
        default="",
    )

    args = argparser.parse_args()

    if args.replot:
        replot()
    else:
        asyncio.run(prefill_benchmark())

# python3 prefill_throughput.py --base-url http://127.0.0.1:8080 --model MiniMax-M2.5-W8A8 --max-input-length-k 128 
```

##### 参考启动命令
```python {wrap}
python prefill_throughput.py \
  --base-url http://10.16.1.51:30010 \
  --model /module/GLM-5-W8A8 --max-input-length-k 50  
  --tokenizer-path /module/GLM-5-W8A8 \
  2>&1 | tee prefill_glm5-int8-ep16dp16-mtp3-$(date "+%m%d-%H%M").log
```

#### 使用接口  **/generate （**<text bgcolor="yellow">**未更新**</text>**）**

##### 脚本代码 prefill_throughput_read.py

##### 参考启动命令

### Decode 性能测试脚本

#### 使用接口 **/v1/completions**）

##### 脚本代码 python decode_throughput.py
```python {wrap}
import asyncio
import aiohttp
import json
import time
import random
import argparse
from datetime import datetime
import numpy as np

TIMESTAMP_FORMAT = "%Y-%m-%d-%H-%M-%S"

time_stamp = datetime.now().strftime(TIMESTAMP_FORMAT)

tokenizer = None

global_prefix_ids: list[int] | None = None

def create_prompts(input_len: int, num_req: int=1, prefix_len: int=0, prefix_ids: list[int] | None = None) -> list[list[int]]:
    if prefix_len > 0:
        assert input_len >= prefix_len
        if prefix_ids is None:
            prefix_ids = [random.randint(42, 10000) for _ in range(prefix_len)]
        else:
            assert len(prefix_ids) == prefix_len
    else:
        prefix_ids = []

    return [prefix_ids + [random.randint(42, 10000) for _ in range(input_len-prefix_len)] for _ in range(num_req)]

    
def detokenize_to_str(token_id_list: list[int]) -> str:
    from transformers import AutoTokenizer
    global tokenizer
    if tokenizer == None:
        assert args.tokenizer_path != ''
        tokenizer = AutoTokenizer.from_pretrained(args.tokenizer_path)
    return tokenizer.decode(token_id_list)

def completion_data(model: str, base_url: str, prompt: str | list[int], ouptut_len: int=10):
    
    prompt = detokenize_to_str(prompt)
    data_template = {
        "model": model,
        "stream": True,
        "max_tokens": ouptut_len,
        "temperature": 0.6,
        "prompt": prompt,
        "top_p": 0.95,
        "ignore_eos": True,
        "stream_options" : {
            "include_usage": True},
    }
    url = f"{base_url}/v1/completions"
    return data_template, url
    
    
async def single_task(prompt_ids: list[int], output_len: int=10, sleep_time: float=0, if_print: bool=False):

    if sleep_time > 0:
        await asyncio.sleep(sleep_time)

    data_template, url = completion_data(args.model, args.base_url, prompt=prompt_ids, ouptut_len=output_len)

    t_start = time.time()    
    timeout = aiohttp.ClientTimeout(total=6000)
    connector = aiohttp.TCPConnector(limit=0)  # 连接数
    first_token = True
    first_token_time = t_start
    last_token_time = t_start
    ttft = 0
    response_interval = []
    async with aiohttp.ClientSession(connector=connector, timeout=timeout) as session:
        async with session.post(url, json=data_template) as response:
            async for chunk in response.content:

                chunk = chunk.decode('utf-8')
                if chunk.startswith("data: "):
                    chunk_str = chunk.strip()[6:].strip()
                    if chunk_str != "[DONE]":
                        chunk_dict = json.loads(chunk_str)
                        if first_token:
                            first_token_time = time.time()
                            first_token = False
                            last_token_time = first_token_time
                        else:
                            current_time = time.time()
                            response_interval_time = round((current_time - last_token_time)*1000, 2)
                            response_interval.append(response_interval_time)
                            last_token_time = current_time
                        if 'usage' in chunk_dict and chunk_dict.get("usage") is not None:
                            usage_data = {
                                'prompt_tokens': chunk_dict["usage"]["prompt_tokens"],
                                'completion_tokens': chunk_dict["usage"]["completion_tokens"],
                                'total_tokens': chunk_dict["usage"]["total_tokens"]
                            }
                    else:
                        last_token_time = time.time()

    ttft = first_token_time - t_start
    tpot = np.median(np.array(response_interval)).item()/1000
    
    if if_print:
        print(f"ttft: {ttft*1000:.2f} ms, step time: {tpot * 1000:.2f} ms, "
              f"output_len: {usage_data['completion_tokens']}, "
              f"len_response_interval = {len(response_interval)}, "
              f"response_interval = {response_interval}", 
              flush=True)
    return ttft, tpot
    
async def decode_benchmark():

    bs_list = []
    max_bs_step = args.max_batch_size_step
    cur_bs = args.min_batch_size
    while cur_bs <= args.max_batch_size:
        bs_list.append(cur_bs)
        if cur_bs < max_bs_step:
            cur_bs *= 2
        else:
            cur_bs += max_bs_step
    
    max_kv_cache_len = args.max_total_kvcache_length_k*1024
    min_input_len = int(args.min_input_length_k*1024)
    max_input_len = args.max_input_length_k*1024
    max_step_len = args.max_input_length_step_k*1024

    num_repeats = args.num_repeats
    dp_size = args.dp_size

    assert args.other_request_output_length > args.target_request_output_length
    reserve_output_length = args.other_request_output_length + 20

    global global_prefix_ids
    random.seed(60)
    global_prefix_ids = create_prompts(max_input_len)[0]
    prompt_len = max_input_len - reserve_output_length
    print(f"warmup prefix start ...", flush=True)
    tasks = [
        asyncio.create_task(
            single_task(
                global_prefix_ids[:prompt_len], 
                1,
            )
        ) for _ in range(args.dp_size*2)
    ]
    results = await asyncio.gather(*tasks)
    print(f"warmup prefix finished", flush=True)

    save_path = f"{args.save_path}/{time_stamp}_decode_tpot_results.json"

    # bench
    tpot_var_bs_list = []
    for i, bs in enumerate(bs_list):
        print(f"\n--- Local BS={bs} ---")
        max_local_input_len = min(
            max_kv_cache_len // bs, 
            max_input_len
        ) - reserve_output_length
        assert max_local_input_len > 0
        prompt_len_list = []
        prompt_len = min_input_len
        while prompt_len < max_local_input_len:
            prompt_len_list.append(prompt_len)
            if prompt_len <= max_step_len:
                prompt_len *= 2
            else:
                prompt_len += max_step_len
        prompt_len_list.append(max_local_input_len)

        tpot_var_len_list = []
        for prompt_len in prompt_len_list:
            tmpt_tpot_list = []
            print(f"\nPrompt length: {prompt_len}")
            for _ in range(num_repeats):
                prompt_ids_list = create_prompts(
                    prompt_len, 
                    num_req=bs*dp_size,
                    prefix_len=prompt_len,
                    prefix_ids=global_prefix_ids[:prompt_len]
                )

                tasks = []
                for i, prompt_ids in enumerate(prompt_ids_list):
                    if i == bs*dp_size-1:
                        tasks.append(
                            asyncio.create_task(
                                single_task(
                                    prompt_ids, 
                                    args.target_request_output_length, 
                                    args.target_request_wait_time, 
                                    if_print=True
                                )
                            )
                        )
                    else:
                        tasks.append(
                            asyncio.create_task(
                                single_task(
                                    prompt_ids, 
                                    args.other_request_output_length,
                                )
                            )
                        )
                results = await asyncio.gather(*tasks)
                tpot = results[-1][1]
                tmpt_tpot_list.append(tpot)
            avg_tpot = sum(tmpt_tpot_list) / len(tmpt_tpot_list)
            tpot_var_len_list.append(avg_tpot)
        
        tpot_var_bs_list.append(
            {
                "bs": bs,
                "prompt_len_list": prompt_len_list,
                "tpot_var_len_list": tpot_var_len_list
            }
        )

        result_dict = {
            "bs_list": bs_list[:len(tpot_var_bs_list)],
            "tpot_var_bs_list": tpot_var_bs_list
        }

        with open(save_path, "w") as f:
            json.dump(result_dict, f, indent=4)
        print(f"save result dict to {save_path}")

        if args.plot:
            plot_results(result_dict)

def plot_results(result_dict: dict, save_path: str | None = None):
    import matplotlib.pyplot as plt
    import numpy as np

    bs_list = result_dict['bs_list']
    tpot_var_bs_list = result_dict['tpot_var_bs_list']

    plt.figure(figsize=(10, 5))
    for i, bs in enumerate(bs_list):
        prompt_len_list = tpot_var_bs_list[i]["prompt_len_list"]
        tpot_var_len_list = np.array(tpot_var_bs_list[i]["tpot_var_len_list"]) * 1000
        plt.plot(prompt_len_list, tpot_var_len_list, marker='o', label=f'bs={bs}')
    

    if args.x_log_scale:
        plt.xscale('log')
    plt.ylim(bottom=0)
    plt.xlabel('Context Length')
    plt.ylabel('Step Time (ms)')
    plt.title('Decode TPOT Benchmark')
    plt.legend(bbox_to_anchor=(0.5, 1.05), loc='lower center', ncol=7)

    plt.tight_layout(pad=0.1)
    plt.grid(True)
    if save_path == None:
        save_path = f"{args.save_path}/{time_stamp}_decode_tpot.png"
    print(f"save result figure to {save_path}")
    plt.savefig(save_path)

def replot():
    assert args.result_path != ''
    with open(args.result_path, "r") as f:
        result_dict = json.load(f)

    suffix = '_decode_tpot_results.json'
    if args.result_path.endswith(suffix):
        prefix = args.result_path.rstrip(suffix)
    else:
        prefix = f"{args.save_path}/{time_stamp}"
    new_save_path = f"{prefix}_decode_tpot.png"
    plot_results(result_dict=result_dict, save_path=new_save_path)

if __name__ == '__main__':
    argparser = argparse.ArgumentParser(description='Decode Step Time Benchmark')

    argparser.add_argument(
        '--base-url',
        type=str,
        default='http://127.0.0.1:30002',
    )

    argparser.add_argument(
        '--model',
        type=str,
        default='DeepSeek-V3',
    )

    argparser.add_argument(
        '--min-input-length-k',
        type=float,
        default=1,
        help='min input length test, '
        '--min-input-length-k 1 means 1024.'
    )

    argparser.add_argument(
        '--max-input-length-k',
        type=int,
        default=0,
        help='max input length test, '
        '--max-input-length-k 128 means 128k.'
    )

    argparser.add_argument(
        '--max-input-length-step-k',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--max-total-kvcache-length-k',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--min-batch-size',
        type=int,
        default=1,
    )

    argparser.add_argument(
        '--max-batch-size',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--max-batch-size-step',
        type=int,
        default=16,
    )

    argparser.add_argument(
        '--other-request-output-length',
        type=int,
        default=128,
    )

    argparser.add_argument(
        '--target-request-output-length',
        type=int,
        default=32,
    )

    argparser.add_argument(
        '--target-request-wait-time',
        type=float,
        default=0.1,
    ) 

    argparser.add_argument(
        '--num-repeats',
        type=int,
        default=3,
    )

    argparser.add_argument(
        '--dp-size',
        type=int,
        default=1,
    )

    argparser.add_argument(
        '--str-input',
        action=argparse.BooleanOptionalAction,
        default=False,
        help='Default we use token_ids_list as prompt. '
        'If inference engine does not support token_ids_list as input, '
        'we can set --str-input and we will detokenizer token_ids_list to str.'
    )

    argparser.add_argument(
        '--tokenizer-path',
        type=str,
        default='',
    )

    argparser.add_argument(
        '--plot',
        action=argparse.BooleanOptionalAction,
        default=True,
    )

    argparser.add_argument(
        '--x-log-scale',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--save_path',
        type=str,
        default="./",
    )

    argparser.add_argument(
        '--replot',
        action=argparse.BooleanOptionalAction,
        default=False,
    )

    argparser.add_argument(
        '--result-path',
        type=str,
        default="",
    )

    args = argparser.parse_args()
    # import pdb
    # pdb.set_trace()
    if args.replot:
        replot()
    else:
        asyncio.run(decode_benchmark())

# python3 decode_throughput.py --base-url http://127.0.0.1:8080 --model DeepSeek-V3 --max-input-length-k 128 --max-total-kvcache-length-k 300 --max-batch-size 32 
```

##### 参考启动命令
```python {wrap}
python decode_throughput.py \
  --base-url http://10.16.1.36:30005 \
  --model /module/GLM-5-W8A8 \
  --min-input-length-k 1 \
  --max-input-length-k 50 \
  --max-input-length-step-k 1 \
  --max-total-kvcache-length-k 980 \
  --min-batch-size 1 \
  --max-batch-size 32 \
  --max-batch-size-step 1 \
  --dp-size 16 \
  --num-repeats 3 \
  --tokenizer-path /module/GLM-5-W8A8 \
  2>&1 | tee decode_glm5-int8-ep16dp16-mtp3-$(date "+%m%d-%H%M").log
```

#### 使用接口 **/generate （**<text bgcolor="yellow">**未更新**</text>**）**

### 启动服务命令（PD分离场景）

**mc_topo.config （NMZ-400G网卡，mooncake）**
```yaml {wrap}
0000:9f:00.0 mlx5_2 hip:0
0000:57:00.0 mlx5_3 hip:1
0000:5e:00.0 mlx5_4 hip:2
0000:05:00.0 mlx5_5 hip:3
0000:e5:00.0 mlx5_6 hip:4
0000:c1:00.0 mlx5_7 hip:5
0000:cc:00.0 mlx5_8 hip:6
0000:b1:00.0 mlx5_9 hip:7
```

**topo.config （NMZ-400G网卡，deepep）**
```yaml {wrap}
0000:9f:00.0 mlx5_2 2
0000:57:00.0 mlx5_3 3
0000:5e:00.0 mlx5_4 4
0000:05:00.0 mlx5_5 5
0000:e5:00.0 mlx5_6 6
0000:c1:00.0 mlx5_7 7
0000:cc:00.0 mlx5_8 8
0000:b1:00.0 mlx5_9 9

```

#### prefill节点

##### pp8
```python {wrap}
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
export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
# nccl通信
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
# mooncake
export MC_TOPO_FILE_FORCE=/home/mc_topo.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ALLREDUCE_STREAM_WITH_COMPUTE=1
# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
export TVM_HOME=/home/tilelang/3rdparty/tvm
export TVM_LIBRARY_PATH=$TVM_HOME/build
export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH
export ROCBLAS_TENSILE_LIBPATH=/home/library_gpu6_glm5_int8
model_path=/module/GLM-5-W8A8
model=${model_path##*/}
tp=1 
pp=8
dp=1
ep=1
attn_cp_size=1
nodes=1
rank=0
host_ip=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${host_ip}" ]; then
    host_ip=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
master_ip=$host_ip
# max_model_len=65536
gpu_mem=0.8
port=30000
dist_port=5000
time=$(date "+%m%d-%H%M")
logpath="dserver/${model}-tp${tp}-dp${dp}-ep${ep}-pp${pp}-cp${attn_cp_size}-$(hostname)"
logfile="${logpath}/glm5_int8_${time}.log"

mkdir -p "${logpath}"

# --context-length "${max_model_len}" \
option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1 "
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
option+=" --quantization slimquant_marlin "
option+=" --attn-cp-size ${attn_cp_size} "
option+=" --disable-cuda-graph "
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
option+=" --disaggregation-mode prefill "
option+=" --pp-max-micro-batch-size 1"
# option+=" --enable-nsa-prefill-context-parallel"
# option+=" --nsa-prefill-cp-mode round-robin-split"
option+=" --moe-dense-tp-size 1"

python3 -m sglang.launch_server --model-path "${model_path}" ${option} \
                                --trust-remote-code \
                                --kv-cache-dtype fp8_e4m3 --dtype bfloat16 --mem-fraction-static "${gpu_mem}" \
                                --host "${host_ip}" --port "${port}" --dist-init-addr "${master_ip}:${dist_port}" \
                                --nnodes "${nodes}" --node-rank "${rank}" \
                                --tp-size "${tp}" --pp-size "${pp}" --dp-size "${dp}" --ep-size "${ep}" \
                                2>&1 | tee "${logfile}"

```

##### cp8
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
# deep_ep
export ROCSHMEM_DISABLE_HDP_FLUSH=1
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
export ROCSHMEM_HEAP_SIZE=3173741824
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=128
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ROCSHMEM_TOPO_FILE_FORCE=/home/topo.config
# mooncake
export MC_TOPO_FILE_FORCE=/home/mc_topo_400g.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export ALLREDUCE_STREAM_WITH_COMPUTE=1
# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
export SGLANG_ROCM_USE_AITER_MOE=0
#export TVM_HOME=/workspace/tilelang/3rdparty/tvm
#export TVM_LIBRARY_PATH=$TVM_HOME/build
#export PYTHONPATH=$TVM_HOME/python:$PYTHONPATH

# export ROCBLAS_TENSILE_LIBPATH=/home/rocblas/library_gpu6_glm5_int8
# export HIPBLASLT_TUNING_OVERRIDE_FILE=/home/hipblaslt/hipblaslt.config
# export W8A8_SUPPORT_METHODS=3

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
option+=" --disable-cuda-graph"
# option+=" --enforce-piecewise-cuda-graph --disable-cuda-graph"
#option+=" --disable-cuda-graph "
option+=" --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"
option+=" --disaggregation-mode prefill "
#option+=" --pp-max-micro-batch-size 1"

python3 -m sglang.launch_server --model-path "${model_path}" ${option} \
                                --trust-remote-code \
                                --kv-cache-dtype fp8_e4m3 --dtype bfloat16 --mem-fraction-static "${gpu_mem}" \
                                --host "${host_ip}" --port "${port}" --dist-init-addr "${master_ip}:${dist_port}" \
                                --nnodes "${nodes}" --node-rank "${rank}" \
                                --tp-size "${tp}" --pp-size "${pp}" --dp-size "${dp}" --ep-size "${ep}" \
                                2>&1 | tee "${logfile}"

```

#### decode节点

##### ep16dp16tp1, mtp3

##### node0
```python {wrap}
#!/usr/bin/env bash
set -euo pipefail

# =========================
# 环境变量
# =========================
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_ENABLE_SPEC_V2=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/profiling
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
####
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FUSED_RMS_QUANT=1                                       
export SGLANG_USE_RMS_QUANT_PATH=1     
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1
####
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
export ROCSHMEM_TOPO_FILE_FORCE=/home/topo.config
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export MC_TOPO_FILE_FORCE=/home/mc_topo.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1
export ROCBLAS_TENSILE_LIBPATH=/home/bw1000/rocblas/library_gpu6_glm5_int8
export HIPBLASLT_TUNING_OVERRIDE_FILE=/home/bw1000/hipblaslt/hipblaslt.config

# 可选调试项，默认关闭
# 抓取gemm size
# export ROCBLAS_LAYER=3
# export HIPBLASLT_LOG_LEVEL=3
# 锁定mtp接收率
# export SGLANG_SIMULATE_ACC_LEN=3.2
# export SGLANG_SIMULATE_ACC_METHOD=match-expected
# 抓取hiplog
# export HIP_LOG_LEVEL=4
# export HIP_MODULE_MASK=0x7fffffff
# export HIP_ENABLE_LIST=hip
# export HIP_LOG_OUTPUT_PATH=/workspace/hip_log

# =========================
# 可改配置
# =========================
MODEL_PATH="/module/GLM-5-W8A8/"

# 自动获取本机 IP
HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi

PORT="30023"

NNODES="2"
NODE_RANK="0"

MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

TP_SIZE="16"
PP_SIZE="1"
DP_SIZE="16"
EP_SIZE="16"

CUDA_GRAPH_MAX_BS="32"
MAX_RUNNING_REQUESTS="512"

SPEC_ALGO="EAGLE"
SPEC_NUM_STEPS="3"
SPEC_TOPK="1"
SPEC_NUM_DRAFT_TOKENS="4"

MEM_FRACTION_STATIC="0.8"
CONTEXT_LENGTH="131072"
PAGE_SIZE="64"
KV_CACHE_DTYPE="fp8_e4m3"

DISAGG_MODE="decode"
IB_DEVICES="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

# =========================
# 日志配置
# =========================
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_${SPEC_ALGO,,}_s${SPEC_NUM_STEPS}_k${SPEC_TOPK}_d${SPEC_NUM_DRAFT_TOKENS}_tp${TP_SIZE}_pp${PP_SIZE}_dp${DP_SIZE}_ep${EP_SIZE}_node${NODE_RANK}_${HOST_NAME}_${TIME_STR}.log"

echo "========================================"
echo "log file: $LOG_FILE"
echo "host    : $HOST"
echo "port    : $PORT"
echo "master  : $MASTER_IP"
echo "dist    : $DIST_INIT_ADDR"
echo "node    : $NODE_RANK/$NNODES"
echo "tp/pp/dp/ep: ${TP_SIZE}/${PP_SIZE}/${DP_SIZE}/${EP_SIZE}"
echo "spec    : ${SPEC_ALGO} steps=${SPEC_NUM_STEPS} topk=${SPEC_TOPK} draft=${SPEC_NUM_DRAFT_TOKENS}"
echo "========================================"

python3 -m sglang.launch_server \
  --pp-size "${PP_SIZE}" \
  --tp-size "${TP_SIZE}" \
  --nsa-prefill-backend flashmla_auto \
  --nsa-decode-backend flashmla_kv \
  --context-length "${CONTEXT_LENGTH}" \
  --trust-remote-code \
  --dtype bfloat16 \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --page-size "${PAGE_SIZE}" \
  --kv-cache-dtype "${KV_CACHE_DTYPE}" \
  --model-path "${MODEL_PATH}" \
  --mem-fraction-static "${MEM_FRACTION_STATIC}" \
  --disable-radix-cache \
  --chunked-prefill-size -1 \
  --quantization slimquant_marlin \
  --cuda-graph-max-bs "${CUDA_GRAPH_MAX_BS}" \
  --moe-dense-tp-size 1 \
  --dp-size "${DP_SIZE}" \
  --ep-size "${EP_SIZE}" \
  --enable-dp-attention \
  --moe-a2a-backend deepep \
  --enable-dp-lm-head \
  --port "${PORT}" \
  --host "${HOST}" \
  --disaggregation-ib-device "${IB_DEVICES}" \
  --disaggregation-mode "${DISAGG_MODE}" \
  --max-running-requests "${MAX_RUNNING_REQUESTS}" \
  --speculative-algorithm "${SPEC_ALGO}" \
  --speculative-num-steps "${SPEC_NUM_STEPS}" \
  --speculative-eagle-topk "${SPEC_TOPK}" \
  --speculative-num-draft-tokens "${SPEC_NUM_DRAFT_TOKENS}" \
  --nnodes "${NNODES}" \
  --node-rank "${NODE_RANK}" \
  --dist-init-addr "${DIST_INIT_ADDR}" \
  2>&1 | tee "$LOG_FILE"
```

##### node1
```python {wrap}
#!/usr/bin/env bash
set -euo pipefail

# =========================
# 环境变量
# =========================
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_ENABLE_SPEC_V2=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/profiling
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
####
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FUSED_RMS_QUANT=1                                       
export SGLANG_USE_RMS_QUANT_PATH=1     
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1
####
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
export ROCSHMEM_TOPO_FILE_FORCE=/home/topo.config
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export MC_TOPO_FILE_FORCE=/home/mc_topo.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1
export ROCBLAS_TENSILE_LIBPATH=/home/bw1000/rocblas/library_gpu6_glm5_int8
export HIPBLASLT_TUNING_OVERRIDE_FILE=/home/bw1000/hipblaslt/hipblaslt.config

# 可选调试项，默认关闭
# 抓取gemm size
# export ROCBLAS_LAYER=3
# export HIPBLASLT_LOG_LEVEL=3
# 锁定mtp接收率
# export SGLANG_SIMULATE_ACC_LEN=3.2
# export SGLANG_SIMULATE_ACC_METHOD=match-expected
# 抓取hiplog
# export HIP_LOG_LEVEL=4
# export HIP_MODULE_MASK=0x7fffffff
# export HIP_ENABLE_LIST=hip
# export HIP_LOG_OUTPUT_PATH=/workspace/hip_log

# =========================
# 可改配置
# =========================
MODEL_PATH="/module/GLM-5-W8A8/"

# 自动获取本机 IP
HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi

PORT="30023"

NNODES="2"
NODE_RANK="1"

MASTER_IP="10.16.1.42"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

TP_SIZE="16"
PP_SIZE="1"
DP_SIZE="16"
EP_SIZE="16"

CUDA_GRAPH_MAX_BS="32"
MAX_RUNNING_REQUESTS="512"

SPEC_ALGO="EAGLE"
SPEC_NUM_STEPS="3"
SPEC_TOPK="1"
SPEC_NUM_DRAFT_TOKENS="4"

MEM_FRACTION_STATIC="0.8"
CONTEXT_LENGTH="131072"
PAGE_SIZE="64"
KV_CACHE_DTYPE="fp8_e4m3"

DISAGG_MODE="decode"
IB_DEVICES="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

# =========================
# 日志配置
# =========================
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_${SPEC_ALGO,,}_s${SPEC_NUM_STEPS}_k${SPEC_TOPK}_d${SPEC_NUM_DRAFT_TOKENS}_tp${TP_SIZE}_pp${PP_SIZE}_dp${DP_SIZE}_ep${EP_SIZE}_node${NODE_RANK}_${HOST_NAME}_${TIME_STR}.log"

echo "========================================"
echo "log file: $LOG_FILE"
echo "host    : $HOST"
echo "port    : $PORT"
echo "master  : $MASTER_IP"
echo "dist    : $DIST_INIT_ADDR"
echo "node    : $NODE_RANK/$NNODES"
echo "tp/pp/dp/ep: ${TP_SIZE}/${PP_SIZE}/${DP_SIZE}/${EP_SIZE}"
echo "spec    : ${SPEC_ALGO} steps=${SPEC_NUM_STEPS} topk=${SPEC_TOPK} draft=${SPEC_NUM_DRAFT_TOKENS}"
echo "========================================"

python3 -m sglang.launch_server \
  --pp-size "${PP_SIZE}" \
  --tp-size "${TP_SIZE}" \
  --nsa-prefill-backend flashmla_auto \
  --nsa-decode-backend flashmla_kv \
  --context-length "${CONTEXT_LENGTH}" \
  --trust-remote-code \
  --dtype bfloat16 \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --page-size "${PAGE_SIZE}" \
  --kv-cache-dtype "${KV_CACHE_DTYPE}" \
  --model-path "${MODEL_PATH}" \
  --mem-fraction-static "${MEM_FRACTION_STATIC}" \
  --disable-radix-cache \
  --chunked-prefill-size -1 \
  --quantization slimquant_marlin \
  --cuda-graph-max-bs "${CUDA_GRAPH_MAX_BS}" \
  --moe-dense-tp-size 1 \
  --dp-size "${DP_SIZE}" \
  --ep-size "${EP_SIZE}" \
  --enable-dp-attention \
  --moe-a2a-backend deepep \
  --enable-dp-lm-head \
  --port "${PORT}" \
  --host "${HOST}" \
  --disaggregation-ib-device "${IB_DEVICES}" \
  --disaggregation-mode "${DISAGG_MODE}" \
  --max-running-requests "${MAX_RUNNING_REQUESTS}" \
  --speculative-algorithm "${SPEC_ALGO}" \
  --speculative-num-steps "${SPEC_NUM_STEPS}" \
  --speculative-eagle-topk "${SPEC_TOPK}" \
  --speculative-num-draft-tokens "${SPEC_NUM_DRAFT_TOKENS}" \
  --nnodes "${NNODES}" \
  --node-rank "${NODE_RANK}" \
  --dist-init-addr "${DIST_INIT_ADDR}" \
  2>&1 | tee "$LOG_FILE"
```

##### ep8dp8tp1, mtp3
```bash {wrap}
#!/usr/bin/env bash
set -euo pipefail

# =========================
# 环境变量
# =========================
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export SGLANG_ENABLE_SPEC_V2=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/profiling
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
####
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_FUSED_RMS_QUANT=1                                       
export SGLANG_USE_RMS_QUANT_PATH=1     
export SGLANG_USE_FUSED_SILU_MUL_QUANT=1
####                         
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
export ROCSHMEM_TOPO_FILE_FORCE=/home/topo.config
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export MC_TOPO_FILE_FORCE=/home/mc_topo.config
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
export NCCL_SOCKET_IFNAME=ens61f1np1
export GLOO_SOCKET_IFNAME=ens61f1np1
export ROCBLAS_TENSILE_LIBPATH=/home/bw1000/rocblas/library_gpu6_glm5_int8
export HIPBLASLT_TUNING_OVERRIDE_FILE=/home/bw1000/hipblaslt/hipblaslt.config



# =========================
# 可改配置
# =========================
MODEL_PATH="/module/GLM-5-W8A8/"

# 自动获取本机 IP
HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi

PORT="30023"

NNODES="1"
NODE_RANK="0"

MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

TP_SIZE="8"
PP_SIZE="1"
DP_SIZE="8"
EP_SIZE="8"

CUDA_GRAPH_MAX_BS="16" 
MAX_RUNNING_REQUESTS="128" 

SPEC_ALGO="EAGLE"
SPEC_NUM_STEPS="3"
SPEC_TOPK="1"
SPEC_NUM_DRAFT_TOKENS="4"

MEM_FRACTION_STATIC="0.8"
CONTEXT_LENGTH="131072"
PAGE_SIZE="64"
KV_CACHE_DTYPE="fp8_e4m3"

DISAGG_MODE="decode"
IB_DEVICES="mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9"

# =========================
# 日志配置
# =========================
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_${SPEC_ALGO,,}_s${SPEC_NUM_STEPS}_k${SPEC_TOPK}_d${SPEC_NUM_DRAFT_TOKENS}_tp${TP_SIZE}_pp${PP_SIZE}_dp${DP_SIZE}_ep${EP_SIZE}_node${NODE_RANK}_${HOST_NAME}_${TIME_STR}.log"

echo "========================================"
echo "log file: $LOG_FILE"
echo "host    : $HOST"
echo "port    : $PORT"
echo "master  : $MASTER_IP"
echo "dist    : $DIST_INIT_ADDR"
echo "node    : $NODE_RANK/$NNODES"
echo "tp/pp/dp/ep: ${TP_SIZE}/${PP_SIZE}/${DP_SIZE}/${EP_SIZE}"
echo "spec    : ${SPEC_ALGO} steps=${SPEC_NUM_STEPS} topk=${SPEC_TOPK} draft=${SPEC_NUM_DRAFT_TOKENS}"
echo "========================================"

python3 -m sglang.launch_server \
  --pp-size "${PP_SIZE}" \
  --tp-size "${TP_SIZE}" \
  --nsa-prefill-backend flashmla_auto \
  --nsa-decode-backend flashmla_kv \
  --context-length "${CONTEXT_LENGTH}" \
  --trust-remote-code \
  --dtype bfloat16 \
  --dist-timeout 10000 \
  --watchdog-timeout 3600 \
  --page-size "${PAGE_SIZE}" \
  --kv-cache-dtype "${KV_CACHE_DTYPE}" \
  --model-path "${MODEL_PATH}" \
  --mem-fraction-static "${MEM_FRACTION_STATIC}" \
  --disable-radix-cache \
  --chunked-prefill-size -1 \
  --quantization slimquant_marlin \
  --cuda-graph-max-bs "${CUDA_GRAPH_MAX_BS}" \
  --moe-dense-tp-size 1 \
  --dp-size "${DP_SIZE}" \
  --ep-size "${EP_SIZE}" \
  --enable-dp-attention \
  --moe-a2a-backend deepep \
  --enable-dp-lm-head \
  --port "${PORT}" \
  --host "${HOST}" \
  --deepep-mode low_latency \
  --max-running-requests "${MAX_RUNNING_REQUESTS}" \
  --speculative-algorithm "${SPEC_ALGO}" \
  --speculative-num-steps "${SPEC_NUM_STEPS}" \
  --speculative-eagle-topk "${SPEC_TOPK}" \
  --speculative-num-draft-tokens "${SPEC_NUM_DRAFT_TOKENS}" \
  --nnodes "${NNODES}" \
  --node-rank "${NODE_RANK}" \
  --dist-init-addr "${DIST_INIT_ADDR}" \
  2>&1 | tee "$LOG_FILE"

```

#### SGLang-Router
```python {wrap}
python3 -m sglang_router.launch_router --pd-disaggregation --prefill http://10.16.1.36:30000 --decode http://10.16.1.46:30023 --policy cache_aware --port 30005
```

## 镜像更新记录

### 2026.04.13

1. 更新镜像，千卡路径：/public/home/sglang/sglang_glm5_int8_0413_latest.tar
	1. 更新lightop包：支持融合算子
  <view type="1">

    <file token="R8pHbKirFo2zMwxVvagcy5ainSh" name="lightop-0.6.0+das1.1git9d86de7.abi1.dtk2604.torch2.9-cp310-cp310-linux_x86_64.whl"/>

  </view>

  1. 更新deepgemm包：解决大bs下精度问题（mtp精度问题）
  <view type="1">

    <file token="OyKrbvLAcoRq65xL2upcJoJCnib" name="deepgemm-2.1.0+das.dtk2604.01290c7-cp310-cp310-linux_x86_64.whl"/>

  </view>

  1. Tuning lm head gemm，文件位置：千卡集群：/public/home/sglang/maxiao/library_gpu6_glm5_int8
  添加环境变量：<text bgcolor="yellow">export ROCBLAS_TENSILE_LIBPATH=/home/library_gpu6_glm5_int8</text>
1. pd分离场景下，decode节点开启mtp3，并行策略设置ep16dp16tp1，opencompass测试<text bgcolor="yellow">humaneval精度正常</text>
  <image token="R7hOb2IJUou40ox3o4AcNCs5nng" width="2560" height="858" align="center"/>

### 2026.04.14

1. 更新镜像，千卡路径：/public/home/sglang/sglang_glm5_int8_0414_v2.tar
  快传路径：rayfile-c -a [wuzh01.hpccube.com](http%3A%2F%2Fwuzh01.hpccube.com) -P 65012 -u guobj -w 5306a83256Z3VvYmo=_ed1c657ea795a76a -no-meta -symbolic-links follow -retry 10 -retrytimeout 30 -o download -s '/images/sglang_glm5_int8_0414_v2.tar' -d <请输入下载目标全路径并替换尖括号及本内容>
  1. 更换flash_mla包，优化<text bgcolor="yellow">tp1</text>性能
  <view type="1">

    <file token="VvoqbMipPo472sx3Rpxc1qTsnXc" name="flash_mla-1.2.0+das.opt1.dtk2604.torch290.2604141429.ga45f64-cp310-cp310-manylinux_2_28_x86_64.whl"/>

  </view>

  1. 去掉decode节点启动脚本环境变量：<text bgcolor="yellow">GPU_FLUSH_ON_EXECUTION=true</text>。该环境变量为debug阶段使用，会把runtime的lauchkernel API变成同步，影响性能，之前没有检查到。
	1. 框架侧act_quant接入lightop版本，不再使用tilelang版本。
1. pd分离场景，prefill节点使用tp8cp8，decode节点使用ep16dp16tp1、开启mtp3，opencompass测试<text bgcolor="yellow">humaneval精度正常</text>
  <image token="Oxu7be5wnozvnsxm8Mvc2OICnIb" width="1982" height="558" align="center"/>

### 2026.04.23 <text bgcolor="light-red">（因无节点、暂未出包）</text>

<text bgcolor="light-red">**新镜像因无节点、暂未出。需在0414_v2镜像里自行更新依赖包。另：后续有机器会在发版镜像验证glm5，如果可以，之后根据发版镜像走。**</text>

<view type="1">

  <file token="AgCdb8YS7odBhpxDIdCcASoAnje" name="aiter-0.1.1.dev631+ga5416af99-cp310-cp310-linux_x86_64.whl"/>

</view>

<view type="1">

  <file token="GCBZb2FWtoCfDyx3rCQcpULpnnh" name="lightop-0.6.0+das1.1gita43dcca.abi1.dtk2604.torch2.9-cp310-cp310-linux_x86_64.whl"/>

</view>

<view type="1">

  <file token="Nldhb48ito8I3dxLhTLcwO6in0c" name="sglang-0.0.0.dev0-py3-none-any.whl"/>

</view>

1. ~~更新镜像，乌镇研发集群路径：~~
	1. ~~更新lightop包，融合set_kv_buffer和quant算子；paged mqa logits算子融合；融合get_logits_head_gate算子~~
    <view type="1">

      <file token="DMNibyiBAoU8zWxmA7Hcqjd5nPc" name="lightop-0.6.0+das1.1gita43dcca.abi1.dtk2604.torch2.9-cp310-cp310-linux_x86_64.whl"/>

    </view>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/精度问题|精度问题]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-V3|DeepSeek-V3]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/GLM5-W8A8int8|GLM5-W8A8int8]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/MiniMax-M2.5-W8A8|MiniMax-M2.5-W8A8]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OpenCompass|OpenCompass]]（framework）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）

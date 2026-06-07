---
title: 专项优化模型测试流程
tags:
  - AI
  - LLM
  - Benchmark
model:
  - deepseek-r1
  - DeepSeek-R1-bf16
  - DeepSeek-R1-INT8
  - Mixtral-8x22B
  - Qwen2-72B-Instruct
  - Qwen2-72B-Instruct-quantized.w8a8
framework:
  - PyTorch
  - vLLM
issue_type: []
source: feishu
feishu_token: Aq5uwv2UZib2apkTqLEcnIEZn0g
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/Aq5uwv2UZib2apkTqLEcnIEZn0g
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 专项优化模型测试流程

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/Aq5uwv2UZib2apkTqLEcnIEZn0g)

使用vllm serve模式测试

测试结果：[[飞书知识图谱/资源/专项优化模型性能验收--caqJ1nif|专项优化模型性能验收]]

性能仪表盘：[[飞书知识图谱/资源/专项优化模型性能跟踪--c9SB1n5f|专项优化模型性能跟踪]]

测试过程数据：[[飞书知识图谱/资源/专项优化模型——最大并发数--c7ndmnOc|专项优化模型——最大并发数]]

提频测试数据：[[飞书知识图谱/资源/hycpu升频验证（千卡集群）--cnJDqngL|hycpu升频验证（千卡集群）]]

## 调度优化（250412）

<callout emoji="bullettrain_front" background-color="light-orange" border-color="light-orange">

目前零消耗调度优化和step8调度优化是排他优化，不能放在一起使用。是cpu端的逻辑优化 和卡没关系，bw、k100ai节点的intelcpu or hycpu都可测
</callout>

### 零消耗调度优化（20240414更新）暂时

http://10.6.10.68/dcutoolkit/deeplearing/vllm/-/tree/v0.7.2-zero，需要用该分支出一个vllm包，然后替换镜像中的vllm即可；

出包：
```plaintext {wrap}
git clone -b v0.7.2-zero http://123.235.52.21:32620/dcutoolkit/deeplearing/vllm.git
ln -s /usr/lib/x86_64-linux-gnu/librt.so.1  /usr/lib/x86_64-linux-gnu/librt.so
pip install cmake==3.29
ADD_GIT_VERSION=1 VLLM_INSTALL_PUNICA_KERNELS=1 python setup.py bdist_wheel
```

<quote-container>

zero消耗调度优化包：/public/home/shantf/bw1000/ali_vllm/whl/vllm-0.7.2+das.opt1.beta.dbbad40.dtk2504-cp310-cp310-linux_x86_64.whl
</quote-container>

测试方法：

在服务端加上环境变量export HIP_ALLOC_INITIALIZE=0、export VLLM_ZERO_OVERHEAD=1

### step8调度优化

**环境安装：**

**vllm0.7.2**

需要将镜像中的vllm替换成/public/home/zhuww/whl/dtk-25.04-rc7/0403/vllm-0.7.2+das.opt1.beta.d00b46f.dtk2504-cp310-cp310-linux_x86_64.whl

**vllm0.6.2**

/public/home/zhuww/whl/dtk-25.04-rc7/0402/vllm-0.6.2+das.opt3.4dee2b4.dtk2504-cp310-cp310-linux_x86_64.whl

**测试方法：**

在之前测试在线性能的基础上，服务端添加--num-scheduler-steps 8 即可测试step8调度优化的性能

## 671b全试

### bw环境

**起容器**
```bash {wrap}
docker run -it  --shm-size 500g --network=host  --name=671-gbj-fixpy --privileged --device=/dev/kfd --device=/dev/dri --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined  -v `pwd`:/mnt -v /public/opendas/DL_DATA/llm-models:/model  -v /opt/hyhal:/opt/hyhal:ro image.sourcefind.cn:5000/dcu/admin/base/custom:vllm0.7.2-ubuntu22.04-dtk25.04-rc7-das1.5-py3.10-20250327-fixpy bash
```

**环境变量（****hy****），容器默认是RING，需要unset NCCL_ALGO**
```plaintext
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_HOST_IP=13.13.6.20
export NCCL_SOCKET_IFNAME=ens47f1np1
export GLOO_SOCKET_IFNAME=ens47f1np1
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export NCCL_IB_HCA=mlx5_0:1,mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1

export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
#export NCCL_NET_GDR_LEVEL=7
#export NCCL_NET_GDR_READ=1
#export NCCL_SDMA_COPY_ENABLE=0
unset NCCL_ALGO
export VLLM_RPC_TIMEOUT=1800000
export VLLM_MLA_DISABLE=1
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
```

**起server服务：**[[飞书知识图谱/资源/Deepseek671b VLLM使用部署方法说明--cNf5rnfd|Deepseek671b VLLM使用部署方法说明]]

使用的最优方案mtp并行解码
```bash {wrap}
VLLM_RPC_TIMEOUT=1800000 vllm serve /model/deepseek-r1/DeepSeek-R1-bf16/ --disable-log-requests --trust-remote-code  --distributed-executor-backend ray --dtype bfloat16 --max-model-len 34240 --max-seq-len-to-capture 34240 -tp 32 --gpu-memory-utilization 0.92 --max-num-seqs 128 --num-speculative-tokens 1 --speculative-disable-by-batch-size 33  2>&1 | tee log/34240.log
```

**客户端：**
```bash {wrap}
python3 benchmark_serving_07.py  --model /model/deepseek-r1/DeepSeek-R1-bf16/ --random-input-len 512 --random-output-len 1024   --csv-path ./csv --dataset-name random --dtype bfloat16 --save-result --csv-name 671-bf16  --ignore-eos --max-concurrency 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 --port 8000 --ratio 1  2>&1 | tee log/671-bf16.log
```

## 671b-INT8测试

server
```bash {wrap}
VLLM_RPC_TIMEOUT=1800000 vllm serve  /model/deepseek-r1/DeepSeek-R1-INT8/  --trust-remote-code  --distributed-executor-backend ray --dtype bfloat16 --max-model-len 16384 -tp 16 --gpu-memory-utilization 0.95 --max-num-seqs 128 --num-speculative-tokens 1 --speculative-disable-by-batch-size 33 
```

Client
```bash {wrap}
python3 benchmark_serving_07.py  --model /model/deepseek-r1/DeepSeek-R1-INT8/ --random-input-len 512 --random-output-len 1024   --csv-path ./csv --dataset-name random --dtype bfloat16 --save-result --csv-name 671-int8  --ignore-eos --max-concurrency 1 2 3 4 5 6 7 8 9 10  --port 8000 --ratio 1  2>&1 | tee log/671-int8.log
```

## K100AI&BW测试环境

<callout emoji="trophy" background-color="light-orange" border-color="light-orange">

若Mixtral-8x22B和QwQ-32B等模型，不能控制输入输出token长度，则在服务端加--max-model-len=32000，或者16324；

若测试过程中，设置的最大并发大于256，需要在服务端加--max-num-seqs 1024，这里的值需要大于所设置的最大并发；
</callout>

拉取测试工具vllm_toolkit 
```bash {wrap}
git clone http://123.235.52.21:32620/dcutoolkit/deeplearing/vllm_toolkit.git
```

起容器：
```plaintext {wrap}
docker run -it  --shm-size 500g --network=host  --name=wen-docker --privileged --device=/dev/kfd --device=/dev/dri --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined  -v `pwd`:/mnt -v /public/opendas/DL_DATA/llm-models:/mnt/model  -v /opt/hyhal:/opt/hyhal:ro image.sourcefind.cn:5000/dcu/admin/base/pytorch:2.4.1-ubuntu22.04-dtk25.04-py3.10-fixpy bash
```

起server服务：
```bash {wrap}
#环境变量：
export HSA_FORCE_FINE_GRAIN_PCIE=1
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_P2P_LEVEL=SYS
export NCCL_LAUNCH_MODE=GROUP
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1
#fp16
#八卡--max-model-len 16384
#四卡--max-model-len 32768
python -m vllm.entrypoints.openai.api_server --model /mnt/model/Qwen2-72B-Instruct --trust-remote-code --host 0.0.0.0 --port 8088  --tensor-parallel-size 4 --max-model-len 32768  --dtype float16　--num-scheduler-steps 8 

#int8
#二卡--max-model-len 16384
#四卡--max-model-len 32768
HIP_VISIBLE_DEVICES=0,1,2,3 python -m vllm.entrypoints.openai.api_server --model  /mnt/model/Qwen2-72B-Instruct-quantized.w8a8 --trust-remote-code --host 0.0.0.0 --port 8088  --tensor-parallel-size 4 --max-model-len 32768 --dtype float16 --enforce-eager 


#bs大于256时添加下面参数
#--max-num-seqs 1024  
```

执行测试：
```plaintext {wrap}
cd /mnt/toolkit_use/serve

HIP_VISIBLE_DEVICES=0,1,2,3 python3 benchmark_serving_07.py  --model /mnt/model/Qwen2-72B-Instruct-quantized.w8a8 --random-input-len 512 --random-output-len 1024   --csv-path ./result --dataset-name random --dtype float16 --save-result   --ignore-eos --port 8088 --max-concurrency 16 --ratio 1
```

## 关联资源

- [[飞书知识图谱/资源/专项优化模型性能验收--caqJ1nif|专项优化模型性能验收]] · 飞书源链接不可用
- [[飞书知识图谱/资源/专项优化模型性能跟踪--c9SB1n5f|专项优化模型性能跟踪]] · 飞书源链接不可用
- [[飞书知识图谱/资源/专项优化模型——最大并发数--c7ndmnOc|专项优化模型——最大并发数]] · 飞书源链接不可用
- [[飞书知识图谱/资源/hycpu升频验证（千卡集群）--cnJDqngL|hycpu升频验证（千卡集群）]] · 飞书源链接不可用
- [[飞书知识图谱/资源/Deepseek671b VLLM使用部署方法说明--cNf5rnfd|Deepseek671b VLLM使用部署方法说明]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/并发|并发]]（metric）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/deepseek-r1|deepseek-r1]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-bf16|DeepSeek-R1-bf16]]（model）
- [[飞书知识图谱/实体/DeepSeek-R1-INT8|DeepSeek-R1-INT8]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/Mixtral-8x22B|Mixtral-8x22B]]（model）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）
- [[飞书知识图谱/实体/Qwen2-72B-Instruct|Qwen2-72B-Instruct]]（model）
- [[飞书知识图谱/实体/Qwen2-72B-Instruct-quantized.w8a8|Qwen2-72B-Instruct-quantized.w8a8]]（model）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

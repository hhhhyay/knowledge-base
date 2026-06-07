---
title: sglang_nmz测试文档
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Channel-INT8
  - GLM-5-W8A8
framework:
  - Mooncake
  - PyTorch
  - SGLang
  - vLLM
issue_type:
  - 精度问题
source: feishu
feishu_token: Jr7AwfL7HigAVBkfsF9chHRznjb
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/Jr7AwfL7HigAVBkfsF9chHRznjb
last_synced_at: 2026-06-07T17:07:53.393Z
---

# sglang_nmz测试文档

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/Jr7AwfL7HigAVBkfsF9chHRznjb)

节点登录参考昊宇文档[[飞书知识图谱/资源/NMZ机器测试流程--cvLWMnhf|NMZ机器测试流程]]

[[飞书知识图谱/资源/flash_attn发包测试记录--ciMXYnPt|flash_attn发包测试记录]]

[[飞书知识图谱/资源/flashmla发包更新记录--czCYUnof|flashmla发包更新记录]]

[[飞书知识图谱/资源/Lightop发包更新记录--cjYoOnVg|Lightop发包更新记录]]

[[飞书知识图谱/资源/deepgemm发包记录--cJ4uunQf|deepgemm发包记录]]

~~最新dtk：~~~~http://112.11.77.146:65182/jenkins/rocm/26.04/intel/centos8/DTK-26.04-NMZ-0119-centos8-x86_64.tar.gz~~~~ 【64cu解决lmhead】【dpsk-int8】~~

创建容器：

<callout emoji="tada" background-color="light-orange" border-color="light-orange">

docker run -it --network=host --name=gbj_sgl_1022d --privileged --device=/dev/kfd --device=/dev/dri --ipc=host --shm-size=1024G --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -u root --ulimit stack=-1:-1 --ulimit memlock=-1:-1 -v /opt/hyhal:/opt/hyhal:ro -v `pwd`:/mnt -v /public/opendas/DL_DATA/llm-models:/model:ro sgl_nmz:v4 bash
</callout>

# 最新镜像

千卡集群：/public/home/sglang/sgl_das0416.tar.gz

[[飞书知识图谱/资源/大文件上传下载--cHqT4nFg|大文件上传下载]] 下载参考文档可以使用快传进行下载

rayfile-c -a wuzh01.hpccube.com -P 65012 -u guobj -w 5306a83256Z3VvYmo=_566ecfdbb066bc1c -no-meta -symbolic-links follow -retry 10 -retrytimeout 30 -o download -s '/images/sgl_das0416.tar.gz' -d <请输入下载目标全路径并替换尖括号及本内容> 

# 镜像版本

## 【质量组云浩帮忙ci出的镜像，镜像里环境均为最近比较新的包】

**10.16.6.35:5000/jenkins/model_test_env/sglang:daily-20260416-1454**

乌镇本地：直接docker pull

<callout emoji="wrench" background-color="light-orange" border-color="light-orange">

docker pull **10.16.6.35:5000/jenkins/model_test_env/sglang:daily-20260416-1454**
</callout>

千卡集群：/public/home/sglang/sgl_das0416.tar.gz

## sgl0510rc0_torch290:v3【~~glm5~~、qwen3.5】

存放地址：/public/home/sglang/sgl0510rc0_torch290_v4.tar.gz【千卡集群】

基础镜像：sgl0510rc0_torch290:v3

**镜像环境和v3的区别在：**

**tilelang：**0.1.6.post2+das.dtk2604

**apache-tvm-ffi：**0.1.0

## sgl0510rc0_torch290:v3【qwen3.5】

存放地址：/public/das/sglang/Images/sgl0510rc0_torch290_v3.tar.gz【千卡集群】

基础镜像：

**镜像环境：**

**sglang：**0.0.0.dev10990+g4418f4abb【实际为0.5.10版本】

**sglang-kernel**：0.4.0

**sglang-router**：0.3.2

**deepgemm**：2.1.0+das.dtk2604.def51e3

**deep_ep**：1.0.0+das.opt1.dtk2604.torch290.202604011036.ga42ecd

**flash_attn**：2.6.1+das.opt1.dtk2604.torch291.20260210.g7808665e

**lightop**：0.6.0+das.dtk2604

**triton**：3.3.0+das.opt2.dtk2604.torch291.20260210.g1329924c

## sgl058:v4【Ling-1T-fp8】

存放地址：/public/das/sglang/Images/sgl058_v4.tar.gz 【千卡集群】

基础镜像：sgl058:v3

**镜像环境：**

**sgl-kernel**：v0.5.8

~~**sglang：**~~~~v0.5.8~~

**flash_attn：**2.6.1+das.opt1.dtk2604.torch251.20260207.g0c2610

**lightop**：~~0.6.0+das.dtk2604.8bfcb6b~~

**lightop**：0.6.0+das.dtk2604.3df4fdf

**triton：**3.1.0+das.opt1.811713b.dtk2604

**deepgemm：**2.1.0+das.dtk2604.c32ea3e

**deepep：**1.0.0+das.opt1.dtk2604  【需要添加环境变量<text bgcolor="yellow">**DEEPEP_ENABLE_LL_DISPATCH_OPT=1**</text>】

/mnt/lijian/Code/DeepEP_quant/DeepEP/dist/deep_ep-1.0.0+das.opt1.dtk2604-py3-none-any.whl

**更换了新的galaxy包**

## sgl058:v3

存放地址：/public/home/sglang/sgl058_v3.tar.gz【千卡集群】

基础镜像：sgl058:v2

**镜像环境：**

**sgl-kernel**：v0.5.8

~~**sglang：**~~~~v0.5.8~~

**flash_attn：**2.6.1+das.opt1.dtk2604.torch251.20260207.g0c2610

**lightop**：0.6.0+das.dtk2604.8bfcb6b

**triton：**3.1.0+das.opt1.811713b.dtk2604

**deepgemm：**2.1.0+das.dtk2604.c32ea3e

**deepep：**1.0.0+das.opt1.dtk2604  【需要添加环境变量<text bgcolor="yellow">**DEEPEP_ENABLE_LL_DISPATCH_OPT=1**</text>】

/mnt/lijian/Code/DeepEP_quant/DeepEP/dist/deep_ep-1.0.0+das.opt1.dtk2604-py3-none-any.whl

<text bgcolor="yellow">**更换了glaxy包**</text>

<image token="CN4ibPdtWow33Jx3V5GcFDTInDg" width="979" height="336" align="center"/>

## sgl058:v2

存放地址：/public/home/sglang/sgl058_v2.tar.gz【千卡集群】

基础镜像：sgl:v0.5.8

**镜像环境：**

**sgl-kernel**：v0.5.8

**sglang：**v0.5.8

**flash_attn：**2.6.1+das.opt1.dtk2604.torch251.20260207.g0c2610

**lightop**：0.6.0+das.dtk2604.ead02cb

**triton：**3.1.0+das.opt1.811713b.dtk2604

**deepgemm：**2.1.0+das.dtk2604.c32ea3e

**deepep：**1.0.0+das.opt1.dtk2604  【需要添加环境变量<text bgcolor="yellow">**DEEPEP_ENABLE_LL_DISPATCH_OPT=1**</text>】

/mnt/lijian/Code/DeepEP_quant/DeepEP/dist/deep_ep-1.0.0+das.opt1.dtk2604-py3-none-any.whl

## sgl:v0.5.8

存放地址：/public/das/sglang/Images/sgl_058.tar【千卡集群】

基础镜像：harbor.sourcefind.cn:5443/dcu/admin/base/sglang：0.5.6-ubuntu22.04-dtk26.04-0127-py3.10-20260129

**镜像环境：**

**sgl-kernel**：v0.5.7

**sglang：**v0.5.8

**flash_attn：**2.6.1+das.opt1.dtk2604.20260131.g4edd8bf9【解决pa精度问题】

**lightop**：lightop-0.6.0+das.dtk2604.3535602-cp310-cp310-linux_x86_64.whl【支持tp2、tp1】

**triton：**3.1.0+das.opt1.811713b.dtk2604

**deepgemm：**2.1.0+das.dtk2604.c20e310

<text bgcolor="light-yellow">**使用最新058代码需要替换下deepgemm、deepep包：**</text>

**deepgemm：**/public/home/sglang/yiqa/deepgemm-2.1.0+das.dtk2604.c32ea3e-cp310-cp310-linux_x86_64.whl

**deepep：**/public/home/sglang/yiqa/deep_ep-1.0.0+das.opt1.dtk2604-py3-none-any.whl

**bailing-tp8测试脚本：**
```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=3
# sysctl -w kernel.numa_balancing=0

# #mtp overlap
# export SGLANG_ENABLE_SPEC_V2=1
# # triton改写算子
# export SGLANG_KVALLOC_KERNEL=1
# export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
# export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
# export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
# export SGLANG_GET_LAST_LOC=1
# export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
# export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
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


export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
# export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # 大ep下无效
# export SGLANG_USE_OPT_CAT=0  # 性能优化版q cat
# export SGLANG_USE_FUSED_MLA_CAT=0  # q cat融入mla，不再需要上一条
# export SGLANG_USE_FUSED_RMSNORM_ROPE=0  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
# export SGLANG_USE_FUSED_RMS_QUANT=0                                                                                                                    
# export USE_FUSED_RMS_QUANT=0  # lmslim and vllm                                                                                                        
# export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # w8a8下未适配
# export SGLANG_USE_RMS_QUANT_PATH=0


# export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
# export HIP_GRAPH_USE_CMD_CACHE=0
# export SGLANG_USE_FUSED_BAILING_MOE_SUM_ADD=0
export SGLANG_USE_FP8_W8A8_MOE=1
export SGLANG_USE_FUSED_BAILING_RMS_QUANT=1
export SGLANG_USE_FUSED_BAILING_SILU_MUL_FP8_QUANT=1
export SGLANG_USE_FUSED_BAILING_RMS_ROTARY=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1

# custom_allreduce no memory copy
export HIP_KERNEL_EVENT_SYSTENFENCE=1
# export SGLANG_KV_LAYOUT_DCU_FA=1

model_path=/module/Ling-1T-FP8
model=${model_path##*/}
tp=8 #K100AI为8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=6000
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="dserver/$model-tp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 3 1 1 0 7 5 5 4 "
# option+=" --disable-radix-cache "
# option+=" --chunked-prefill-size -1"
# option+=" --max-running-requests 1024 "
# # option+=" --page-size 64"
# # option+=" --ep-size $ep "
# # option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head" 
# # option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
# # option+=" --enable-torch-compile --torch-compile-max-bs 64"
# #option+=" --pp-max-micro-batch-size 2"
# # option+=" --cuda-graph-max-bs 128"
# # option+=" --page-size 64"
# # option+=" --context-length 6000"
# option+=" --disable-cuda-graph "
# #option+=" --disaggregation-prefill-pp 8"
# # option+=" --enable-single-batch-overlap"
# # option+=" --enable-dp-attention --moe-a2a-backend deepep  --deepep-mode low_latency"
# #SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
# # option+=" --disaggregation-ib-device mlx5_9"

python3 -m sglang.launch_server \
  --trust-remote-code \
  --host 0.0.0.0 --port 30000 \
  --model-path /module/Ling-1T-FP8 \
  --chunked-prefill-size -1 \
  --disable-radix-cache \
  --cuda-graph-max-bs 128 \
  --attention-backend fa3 --page-size 64 \
  --tp-size 8 --kv-cache-dtype fp8_e4m3 \
  --mem-fraction-static 0.9 \
  2>&1 | tee bailing.log
```

**bailing-ep8dp2tp4测试脚本：**
```plaintext {wrap}

```


基础镜像：sgl_nmz:v6  

需替换以下包【千卡集群】 

<text bgcolor="light-yellow">~~公网IP：~~</text>[<text bgcolor="light-yellow">~~112.11.119.99:18000~~</text>](http%3A%2F%2F112.11.119.99%3A18000%2F)

~~**Vllm:**~~~~http://10.16.4.1:8000/customized/vllm/dtk-26.04-NMZ-0119-centos8/0.9.2%2Bdas.opt5.dtk2604-56d6c689/vllm-0.9.2%2Bdas.opt5.dtk2604.20260129.g56d6c689-cp310-cp310-manylinux_2_28_x86_64.whl~~

~~**lmslim：**~~~~http://10.16.4.1:8000/customized/lmslim/dtk26.04-nmz-0115/0.3.1%2Bdas.opt3.dtk2604-3529a6c9/lmslim-0.3.1%2Bdas.opt3.dtk2604.20260121.g3529a6c9-cp310-cp310-manylinux_2_28_x86_64.whl~~

~~**flash_attn：**~~~~http://10.16.4.1:8000/customized/flash-attn/dtk-26.04-NMZ-0119-centos8/master-dev-034ec12d/flash_attn-2.6.1%2Bdas.opt1.dtk2604.20260128.g034ec12d-cp310-cp310-manylinux_2_28_x86_64.whl~~~~  ~~

~~/public/home/sglang/liucong/NMZ/flash_attn-2.6.1+das.opt1.dtk2604.20260122.g7dbcb126-cp310-cp310-manylinux_2_28_x86_64.whl~~

~~**lightop**~~~~：lightop-0.6.0+das.dtk2604.3535602-cp310-cp310-linux_x86_64.whl【支持tp2、tp1】~~

~~0.6.0+das.dtk2604.b29d4b2【支持tp4】~~

~~/public/home/sglang/liucong/NMZ/lightop-0.6.0+das.dtk2604.e283df5-cp310-cp310-linux_x86_64.whl~~

~~/public/home/sglang/liucong/NMZ/lightop-0.6.0+das.dtk2604.c6e3a7c-cp310-cp310-linux_x86_64.whl~~

~~**triton**~~~~：/public/home/sglang/liucong/NMZ/triton-3.1.0+das.opt1.811713b.dtk2604-cp310-cp310-linux_x86_64.whl~~

~~**deepgemm：**~~~~/public/home/sglang/liucong/NMZ/deepgemm-2.1.0+das.dtk2604.c20e310-cp310-cp310-linux_x86_64.whl~~

# 镜像版本【dpsk-int8】[sglangv0.5.6]

最新性能测试环境【乌镇本地】：

在sgl_nmz:v6 基础上，替换以下包

dtk：http://112.11.77.146:65182/jenkins/rocm/26.04/intel/centos8/DTK-26.04-NMZ-0119-centos8-x86_64.tar.gz 【64cu解决lmhead】

Deepgemm：/public/home/guobj/nmz/sgl/whl/deepgemm-2.1.0+das.dtk2604.unknown-cp310-cp310-linux_x86_64.whl  

## sgl_nmz:v6 

存放地址：/public/home/guobj/nmz/images/sgl_nmz_v6.tar.gz （乌镇节点）

  /public/das/sglang/Images/sgl_nmz_v6.tar.gz （千卡集群）

基础镜像：sglang_nmz:v5【21节点gbj_sgl_0116d】

镜像环境：

**dtk：**dtk-26.04-NMZ-0115-centos8   

**deep_ep**                           1.0.0+c11ee7aa.20260114

**deepgemm**                           2.1.0+das.dtk2604.dc33693  

**lightop**                           0.6.0+das.dtk2604.5c7ef07

**flash_attn**                    2.6.1+das.opt1.dtk2604.20251231.ge0bf7182

**flash_mla**                          1.0.0+das.opt1.dtk2604

**Mooncake-transfer-engine    0.3.7.post2**

【测试pd分离如果有出现握手失败，需要重新安装下mooncake包】

/mnt/lijian/mooncake/mooncake-wheel/dist/mooncake_transfer_engine-0.3.7.post2-cp310-cp310-manylinux_2_17_x86_64.manylinux_2_28_x86_64.manylinux_2_35_x86_64.whl

/mnt/lijian/temp/DeepEP/dist/deep_ep-1.0.0+c11ee7aa.20260114-py3-none-any.whl

## sglang_nmz:v5【72cu环境】

存放地址：/public/home/guobj/nmz/images/sgl_nmz_v5.tar.gz （乌镇节点）

基础镜像：sglang_nmz:v3【21节点gbj_sgl_1022d】

替换包：

lightop：

flash_attn：1231

flash_mla：0109

deep_ep：0107

mooncake：/mnt/lijian/mooncake/mooncake-wheel/dist/mooncake_transfer_engine-0.3.7.post2-cp310-cp310-manylinux_2_17_x86_64.manylinux_2_28_x86_64.manylinux_2_35_x86_64.whl

## sglang_nmz:v4

存放地址：/public/home/guobj/nmz/images/sgl_nmz_v4.tar.gz （乌镇节点）

基础镜像：sglang_nmz:v3【20节点gbj_sgl_1022d】

**替换包：**

sglang：sglang1225_

flash-mla：flash-mla-1.0.0+das.opt1.dtk2604.20251220.g655cf8f0（tp4-fp8）

lightop：0.6.0+das.dtk2604.4ced019  【rms_norm_dynamic_per_token_quant支持非连续输入】

~~flash_attn：2.6.1+das.opt1.dtk2604.20251219.g524df18c~~

flash_attn：

/mnt/lijian/mooncake/mooncake-wheel/dist/mooncake_transfer_engine-0.3.7.post2-cp310-cp310-manylinux_2_17_x86_64.manylinux_2_28_x86_64.manylinux_2_35_x86_64.whl

mooncake-transfer-engine-0.3.7.post2：/mnt/lijian/mooncake/mooncake-wheel/dist/mooncake_transfer_engine-0.3.7.post2-cp310-cp310-manylinux_2_17_x86_64.manylinux_2_28_x86_64.manylinux_2_35_x86_64.whl

sglang-router-0.2.3：/public/home/guobj/nmz/sgl/whl/sglang_router-0.2.3-cp38-abi3-manylinux_2_28_x86_64.whl 【v0.5.6环境重新编译】

~~lightop：0.6.0+das.dtk2604.5e230c6【松林tunning】~~

~~lightop：lightop-0.6.0+das.dtk2604.685efb0【mtp在大bs下精度问题】~~

~~sglang-router-0.2.1：/mnt/nmz/sgl/sglang1220_885cff59/sgl-router/dist/sglang_router-0.2.1-cp310-cp310-linux_x86_64.whl~~

<text bgcolor="light-gray">~~sglang：sglang1220_885cff596（fp8_e4m3+mtp）~~</text>

<text bgcolor="light-gray">~~lightop：lightop-0.6.0+das.dtk2604.0e78ee6  (fp8_e4m3算子融合)【不支持pp模式】~~</text>

**环境变量：**

<callout emoji="desert_island" background-color="light-orange" border-color="light-orange">

export SGLANG_USE_FUSED_RMSNORM_ROPE=1 【配合fp8_e4m3、lightop算子融合】

#hipGraphLaunch

export HSA_KERNARG_POOL_SIZE=8388608

export ROC_AQL_QUEUE_SIZE=131072

</callout>

## sglang_nmz:v3

存放地址：/public/home/guobj/nmz/images/sglang_nmz_v3.tar.gz （乌镇节点）

/root/gbj/images/sglang_nmz_v3.tar.gz  （142节点）

基础镜像：sglang_nmz:v2

**替换包：**

sglang：sglang1217_0b2c2e3f（ep高吞吐模式）

flash_mla：（tp1-fp8）

dtk：dtk-26.04-NMZ-1215-galaxy-centos8（解决D2H问题）

**环境变量：**

<callout emoji="musical_score" background-color="light-orange" border-color="light-orange">

export USE_DCU_CUSTOM_ALLREDUCE=1

export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0

export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_KVALLOC_KERNEL=1

export ROCSHMEM_HEAP_SIZE=3000000000

export NVSHMEM_SYMMETRIC_SIZE=2450125184

#export ROCSHMEM_HEAP_SIZE=2350125184

export ROCSHMEM_MAX_NUM_CONTEXTS=32

export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384

export ROCSHMEM_TOPO_FILE="/mnt/gbj/built-in-BW-topo-input.xml"

export SGLANG_USE_LIGHTOP=1

export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # disable in w8a8

export SGLANG_USE_OPT_CAT=1

export SGLANG_USE_FUSED_RMS_QUANT=0  # disable in w8a8

export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # disable in w8a8

export SGLANG_TORCH_PROFILER_DIR=/workspace/prof

export SGLANG_SET_CPU_AFFINITY=1

export HIP_KERNEL_BATCH_CEILING=100

export GPU_MAX_HW_QUEUES=3

sysctl -w kernel.numa_balancing=0

export LD_LIBRARY_PATH=/mnt/gbj/rocblas-install/lib/:$LD_LIBRARY_PATH

export LD_LIBRARY_PATH=/root/deepep_nvshmem/dushmem_install/lib/:$LD_LIBRARY_PATH  #deep_nvshmem

#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要

#D2H

<text bgcolor="light-yellow">export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer</text>

<text bgcolor="light-yellow">export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer</text>

export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy

export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）

export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy

export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
</callout>

## sglang_nmz:v2

存放地址：/root/gbj/images/sglang_nmz_v2.tar.gz  （142节点）

基础镜像：sglang_nmz:v1

**新增包：**

deep_ep：1.0.0.20251211   （nvshmem）

存放地址：sglang_nmz:v2  镜像的/root/deepep_nvshmem

**替换包：**

lightop：0.6.0+das.dtk2604.8a1bd94  

[[飞书知识图谱/资源/lightop-0.6.0+das.dtk2604.8a1bd94-cp310-cp310-linux_x86_64.whl--ceEfBnpd|lightop-0.6.0+das.dtk2604.8a1bd94-cp310-cp310-linux_x86_64.whl]]

flashmla：1.0.0+das.opt1.dtk2604.20251205.g3013806d (tp8-fp8)

[[飞书知识图谱/资源/flash_mla-1.0.0+das.opt1.dtk2604.20251205.g3013806d-cp310-cp310-manylinux_2_28_x86_64.whl--c5roTnrg|flash_mla-1.0.0+das.opt1.dtk2604.20251205.g3013806d-cp310-cp310-manylinux_2_28_x86_64.whl]]

**环境变量：**

<callout emoji="bread" background-color="light-orange" border-color="light-orange">

export USE_DCU_CUSTOM_ALLREDUCE=1

export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0

export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_KVALLOC_KERNEL=1

#export ROCSHMEM_HEAP_SIZE=1900000000

#export ROCSHMEM_MAX_NUM_CONTEXTS=32

#export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384

#export ROCSHMEM_TOPO_FILE="/mnt/gbj/built-in-BW-topo-input.xml"

export SGLANG_USE_LIGHTOP=1

export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # disable in w8a8

export SGLANG_USE_OPT_CAT=1

export SGLANG_USE_FUSED_RMS_QUANT=0  # disable in w8a8

export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # disable in w8a8

export SGLANG_TORCH_PROFILER_DIR=/workspace/prof

export SGLANG_SET_CPU_AFFINITY=1

export HIP_KERNEL_BATCH_CEILING=100

export GPU_MAX_HW_QUEUES=3

sysctl -w kernel.numa_balancing=0

export LD_LIBRARY_PATH=/mnt/gbj/rocblas-install/lib/:$LD_LIBRARY_PATH

export LD_LIBRARY_PATH=/root/deepep_nvshmem/dushmem_install/lib/:$LD_LIBRARY_PATH  #deep_nvshmem
</callout>

## sglang_nmz:v1

存放地址：/root/liucong/sglang_nmz_v1.tar

基础镜像：image.sourcefind.cn:5000/dcu/admin/base/custom:vllm0.9.2-rockylinux8.6-dtk26.04-py3.10-20251204-nmz   

替换测试包部分使用[[飞书知识图谱/资源/NMZ测试记录--cfhJBnCf|NMZ测试记录]]的推荐部分

安装sglang的相关依赖：

/root/gbj/sglang/*whl

/root/gbj/xgrammar-0.1.25-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl

/root/gbj/torchvision-0.20.1+das.opt1.dtk2604-cp310-cp310-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl

更换了galaxy:/root/lizhigong/libgalaxyhip.so.5.2.25484.1857-0f32e4f0

**环境变量**

<callout emoji="bread" background-color="light-orange" border-color="light-orange">

export USE_DCU_CUSTOM_ALLREDUCE=1

export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0

export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000

export SGLANG_KVALLOC_KERNEL=1

export SGLANG_USE_LIGHTOP=1

export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # disable in w8a8

export SGLANG_USE_OPT_CAT=1

export SGLANG_USE_FUSED_RMS_QUANT=0  # disable in w8a8

export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # disable in w8a8

export SGLANG_TORCH_PROFILER_DIR=/workspace/prof

export SGLANG_SET_CPU_AFFINITY=1

export HIP_KERNEL_BATCH_CEILING=100

export GPU_MAX_HW_QUEUES=3

sysctl -w kernel.numa_balancing=0

export LD_LIBRARY_PATH=/root/rocblas-install/lib/:$LD_LIBRARY_PATH
</callout>

# prof分析

## hipprof

参考腾飞文档[[飞书知识图谱/资源/tx元宝DeepSeek测试--cAWcpnxs|tx元宝DeepSeek测试]]

sglang的D2H的hipprof单侧

hipprof python test.py

<view type="2">

  <file token="HppCbBVJNoMVHMxFXeZcZbkinAh" name="test.py"/>

</view>

## torchprof

export SGLANG_TORCH_PROFILER_DIR=/workspace/prof

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 # <text bgcolor="light-yellow">dtk26.04上需要设置，防止出现kernel丢失</text>

客户端：需要加上--profile

[[飞书知识图谱/资源/sglang性能分析方法--cMF01nmf|sglang性能分析方法]]

**PD分离prof参考聪哥文档【**[[飞书知识图谱/资源/SGLang源码安装--cybx1nwh|SGLang源码安装]]**】**

PD节点正常起服务，在请求时添加标黄部分
```plaintext {wrap}
python -m sglang.bench_serving --backend sglang --model /model/vllm-w8a8-models/DeepSeek-R1-Channel-INT8 --num-prompts 16  --max-concurrency 16 --dataset-name random-ids --random-input-len 4000 --random-output-len 1 --random-range-ratio 1 --port 30002 --profile --pd-separated --profile-prefill-url http://10.16.6.12:30003


python -m sglang.bench_serving --backend sglang --model /data2/DeepSeek-R1-Channel-INT8 --num-prompts 16  --m--dataset-name random-ids --random-input-len 1 --random-output-len 128 --random-range-ratio 1 --port 30003 --profile --pd-separated --profile-decode-url http://10.16.6.21:30003
```

## curl方式【推荐】
```cpp {wrap}
#替换下ip，decode，prefill抓prof时可以把"num_steps": 20,改为5，不然会很大

curl -X POST http://10.16.6.21:30003/start_profile \
  -H "Content-Type: application/json" \
  -d '{
    "output_dir": "/tmp/profiles",
    "start_step": 5,
    "num_steps": 20,
    "activities": ["CPU", "GPU"]
  }'
```

# nmz节点登录

## 乌镇

<callout emoji="🎁" background-color="light-orange" border-color="light-orange">

节点：均与其他同学共用

10.16.6.12【主要测试节点】、10.16.6.20、10.16.6.21
</callout>

```plaintext {wrap}
ssh guobj@112.11.119.99 -p 65024
动态验证码：
密码：Wyfc666.
ssh 10.16.6.12
```

## 

<view type="1">

  <file token="MLR5bbosuoJJAjxSeMDc3eUSn9e" name="NXSetupU-x64-10.3.2.zip"/>

</view>

<image token="WofWbGGYdo2ws0xPBCJcy8U8n3g" width="861" height="978" align="left"/>

```plaintext {wrap}
sg_zhangqha 密码Sugon@123

226节点：10.60.150.226  root/hygon123;
```


```plaintext {wrap}
sg_wangqw2 密码wangqw2@123

128节点：10.60.156.128 root/hygon123;
```


```plaintext {wrap}
# 需要从226 128节点转
142节点：10.60.56.142 root/hygon123;
```



```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
#export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
#export ROCSHMEM_HEAP_SIZE=1900000000
#export ROCSHMEM_MAX_NUM_CONTEXTS=32
#export UCX_ROCM_IPC_SIGPOOL_MAX_ELEMS=16384
#export ROCSHMEM_TOPO_FILE="/workspace/built-in-BW-topo-input.xml"
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # disable in w8a8
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FUSED_RMS_QUANT=0  # disable in w8a8
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # disable in w8a8
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=8
sysctl -w kernel.numa_balancing=0
export LD_LIBRARY_PATH=/mnt/nmz/rocblas/rocblas-install/lib/:$LD_LIBRARY_PATH

export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
export SGLANG_ENABLE_SPEC_V2=1
# export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
export SGLANG_USE_FA_FP8=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
#hipGraphLaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=1

#rm -rf /tmp/torchinductor_root
#rm -rf ~/.triton/cache

model_path=/model/vllm-w8a8-models/DeepSeek-R1-Channel-INT8
model=${model_path##*/}
tp=8 
pp=1
dp=1
ep=1
nodes=1
rank=0
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=32768
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tp$tp-dp$dp-ep$ep"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1 "
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size 16384"
option+=" --max-running-requests 256 "

option+=" --speculative-algorithm EAGLE --speculative-num-steps 1  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1  --cuda-graph-max-bs 256 "
option+=" --enable-torch-compile --torch-compile-max-bs 64"



#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization slimquant_marlin --kv-cache-dtype fp8_e4m3 \
                                --host $host_ip  --port 30003 --trust-remote-code  \
                                --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla 2>&1 | tee  ${logpath}-$time.log

```

# Q&A

## mooncake握手失败，需要重新安装下mooncake包

卸载重装mooncake包即可

【测试pd分离如果有出现握手失败，需要重新安装下mooncake包】

/mnt/lijian/mooncake/mooncake-wheel/dist/mooncake_transfer_engine-0.3.7.post2-cp310-cp310-manylinux_2_17_x86_64.manylinux_2_28_x86_64.manylinux_2_35_x86_64.whl

/mnt/lijian/temp/DeepEP/dist/deep_ep-1.0.0+c11ee7aa.20260114-py3-none-any.whl

## ep16dp16报ss1.ss_family == ss2.ss_family. 2 vs 10

torchrun nccl Multi machine and multi card training error: ss1.ss_family == ss2.ss_family. 2 vs 10 

和其他64cu节点跑ep16dp16会报错【[torchrun nccl Multi machine and multi card training error: ss1.ss_family == ss2.ss_family. 2 vs 10 ·](https%3A%2F%2Fgithub.com%2Fpytorch%2Fpytorch%2Fissues%2F135355)】

<image token="BXk7b3UjyonyJaxMr4Zcx0wXnge" width="1315" height="468" align="center"/>

注释掉断言后出的torch包可以正常起来服务

http://10.16.4.1:8000/customized/pytorch/dtk26.04-nmz-1202/2.5.1-dev-6d3a335f-noassert/torch-2.5.1%2Bdas.opt1.dtk2604.20260117.g6d3a335f.noassert-cp310-cp310-manylinux_2_28_x86_64.whl

## 报错gloo通信error，需要指定下网卡名

[[Bug]: Gloo 库无法在两台计算机之间进行通信 · Issue #6329 · vllm-project/vllm](https%3A%2F%2Fgithub.com%2Fvllm-project%2Fvllm%2Fissues%2F6329)

export GLOO_SOCKET_IFNAME=ens19f0

export NCCL_SOCKET_IFNAME=ens19f0

## bmz节点跑int8报flashmla支持e4m3

 sglang_v057/python/sglang/srt/layers/attention/dcu_mla_backend.py  +654

<image token="GO44bgC3OoUS4YxgFZMcv943nqe" width="636" height="462" align="center"/>

改为

<image token="IA1Ub2grqoC5FZxB60tcf2qGnSh" width="763" height="357" align="center"/>


```bash {wrap}
export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200

# export LD_LIBRARY_PATH=/workspace/nvshmem_install/lib:$LD_LIBRARY_PATH
# export DEEP_EP_DEVICE_TO_HCA_MAPPING=0:mlx5_2:1,1:mlx5_3:1,2:mlx5_4:1,3:mlx5_5:1,4:mlx5_6:1,5:mlx5_7:1,6:mlx5_8:1,7:mlx5_9:1
# export NVSHMEM_SYMMETRIC_SIZE=1900000000

# export ROCSHMEM_MAX_NUM_CONTEXTS=48
# export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288
# export ROCSHMEM_HEAP_SIZE=1900000000
# export ROCSHMEM_TOPO_FILE_FORCE="/home/pd_config/mc_topo.config"


export SGLANG_TORCH_PROFILER_DIR=/home/profile
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


export ALLREDUCE_STREAM_WITH_COMPUTE=1

# 融合算子
export SGLANG_USE_LIGHTOP=1  # 使用lightop的rope和topk算子，可用
export SGLANG_USE_LIGHTOP_MOE_SUM_MUL_ADD=0  # 大ep下无效
export SGLANG_USE_OPT_CAT=1  # 性能优化版q cat
export SGLANG_USE_FUSED_MLA_CAT=1  # q cat融入mla，不再需要上一条
export SGLANG_USE_FUSED_RMSNORM_ROPE=0  # rms+rope+set_kv_buffer融合，融合后可去掉k cat，与上一条可搭配
export SGLANG_USE_FUSED_RMS_QUANT=0   #glm 有精度问题
export USE_FUSED_RMS_QUANT=1 # lmslim and vllm
export SGLANG_USE_FUSED_SILU_MUL_QUANT=0  # w8a8下未适配
export SGLANG_USE_RMS_QUANT_PATH=0

export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_GRAPH_USE_CMD_CACHE=0
export SGLANG_KV_LAYOUT_DCU_FA=0

export SGLANG_USE_FP8_W8A8_MOE=1
export NCCL_SOCKET_IFNAME=ens61f0np0
export GLOO_SOCKET_IFNAME=ens61f0np0

#model_path=/module2/GLM-5-W8A8
model_path=/model/vllm-fp8-models/GLM-5-W8A8
model=${model_path##*/}
tp=16 #K100AI为8
pp=1
dp=1
ep=1
nodes=2
rank=1
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
master_ip=10.16.6.83
max_model_len=6000
gpu_mem=0.8
time=$(date "+%m%d-%H%M")
mode="cudagraph"
logpath="server/$model-tpp$tp-dp$dp-ep$ep-$(hostname)"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 1 1 2 2 3 3"
#option+=" --disable-radix-cache "
#option+=" --chunked-prefill-size 16384"
# export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
# export CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
#option+=" --max-running-requests 1024 "
option+=" --page-size 64 "
option+=" --nsa-prefill-backend flashmla_auto --nsa-decode-backend flashmla_kv "
#option+=" --enable-nsa-prefill-context-parallel --nsa-prefill-cp-mode round-robin-split "
# option+=" --ep-size $ep "
# option+=" --dp-size $dp --enable-dp-attention --moe-dense-tp-size 1 --enable-dp-lm-head"
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 3  --speculative-eagle-topk 1  --speculative-num-draft-tokens 4 "
# option+=" --enable-torch-compile --torch-compile-max-bs 64"
#option+=" --pp-max-micro-batch-size 2"
#option+=" --cuda-graph-max-bs 512 "
#option+=" --page-size 64" --nsa-prefill-backend flashmla_sparse
#option+=" --context-length 6000"
#option+=" --disable-cuda-graph  " #--skip-server-warmup
option+=" --quantization slimquant_marlin "
# option+=" --quantization w8a8_fp8  "
#option+=" --disaggregation-prefill-pp 8"
# option+=" --enable-single-batch-overlap"
# option+=" --enable-dp-attention --moe-a2a-backend deepep  --deepep-mode low_latency"
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
# option+=" --disaggregation-ib-device mlx5_9 --quantization fp8 --kv-cache-dtype bf16 " --quantization w8a8_int8  --json-model-override-args '{"num_hidden_layers": 6}'
#--quantization slimquant_w4a8_marlin --attention-backend dcu_mla --kv-cache-dtype fp8_e4m3 --quantization slimquant_marlin
python3 -m sglang.launch_server --model-path $model_path  ${option} \
                                --trust-remote-code  \
                             --kv-cache-dtype fp8_e5m2  --dtype bfloat16 --mem-fraction-static $gpu_mem \
                                --host $host_ip --port 30000 --dist-init-addr $master_ip:5000 --nnodes $nodes --node-rank $rank \
                                --tp-size $tp --dp-size $dp 2>&1 | tee  ${logpath}-test--tensor$hostname-$time.log
```

## 关联资源

- [[飞书知识图谱/资源/NMZ机器测试流程--cvLWMnhf|NMZ机器测试流程]] · 飞书源链接不可用
- [[飞书知识图谱/资源/flash_attn发包测试记录--ciMXYnPt|flash_attn发包测试记录]] · 飞书源链接不可用
- [[飞书知识图谱/资源/flashmla发包更新记录--czCYUnof|flashmla发包更新记录]] · 飞书源链接不可用
- [[飞书知识图谱/资源/Lightop发包更新记录--cjYoOnVg|Lightop发包更新记录]] · 飞书源链接不可用
- [[飞书知识图谱/资源/deepgemm发包记录--cJ4uunQf|deepgemm发包记录]] · 飞书源链接不可用
- [[飞书知识图谱/资源/大文件上传下载--cHqT4nFg|大文件上传下载]] · 飞书源链接不可用
- [[飞书知识图谱/资源/lightop-0.6.0+das.dtk2604.8a1bd94-cp310-cp310-linux_x86_64.whl--ceEfBnpd|lightop-0.6.0+das.dtk2604.8a1bd94-cp310-cp310-linux_x86_64.whl]] · 飞书源链接不可用
- [[飞书知识图谱/资源/flash_mla-1.0.0+das.opt1.dtk2604.20251205.g3013806d-cp310-cp310-manylinux_2_28_x86_64.whl--c5roTnrg|flash_mla-1.0.0+das.opt1.dtk2604.20251205.g3013806d-cp310-cp310-manylinux_2_28_x86_64.whl]] · 飞书源链接不可用
- [[飞书知识图谱/资源/NMZ测试记录--cfhJBnCf|NMZ测试记录]] · 飞书源链接不可用
- [[飞书知识图谱/资源/tx元宝DeepSeek测试--cAWcpnxs|tx元宝DeepSeek测试]] · 飞书源链接不可用
- [[飞书知识图谱/资源/sglang性能分析方法--cMF01nmf|sglang性能分析方法]] · 飞书源链接不可用
- [[飞书知识图谱/资源/SGLang源码安装--cybx1nwh|SGLang源码安装]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/精度问题|精度问题]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-INT8|DeepSeek-R1-Channel-INT8]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/NVSHMEM|NVSHMEM]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/SLA|SLA]]（metric）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

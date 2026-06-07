---
title: 海光vLLM环境搭建及DeepSeek模型测试说明（pd分离）
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Channel-INT8
framework:
  - vLLM
issue_type:
  - 崩溃
source: feishu
feishu_token: JgoxdoLSaoEnYExEFEtch3rXn0f
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/JgoxdoLSaoEnYExEFEtch3rXn0f
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 海光vLLM环境搭建及DeepSeek模型测试说明（pd分离）

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/JgoxdoLSaoEnYExEFEtch3rXn0f)

# 镜像环境:

## 元宝最优性能环境（截止到0322）

<callout emoji="beach_with_umbrella" background-color="light-orange" border-color="light-orange">

1. 元宝模型：DeepSeek-R1-Channel-INT8 服务器：nmz 72cu
1. 下面环境是元宝DeepSeek-R1-Channel-INT8 模型使用的最优性能方案（截止到0322），其他模型仅供参考。
1. 对应的测试环境变量和参数可以参考下面“**pd非对称切分部署方式”**
1. 元宝更多环境可参考[[飞书知识图谱/资源/tx元宝DeepSeek测试--cAWcpnxs|tx元宝DeepSeek测试]]
</callout>

**P节点环境推荐使用：**

镜像：harbor.sourcefind.cn:5443/dcu/admin/base/vllm:0.9.2-ubuntu22.04-dtk26.04-0130-py3.10-20260204

替换：

<view type="1">

  <file token="VnfQbo8nLof9dfxF4xLcCaJznvb" name="vllm-0.9.2+das.opt6.dtk2604-cp310-cp310-linux_x86_64.whl"/>

</view>

<view type="1">

  <file token="H8aOboClFoBbtaxTaPrcCrATn6c" name="lightop-0.6.0+das.dtk2604.5781e3c-cp310-cp310-linux_x86_64.whl"/>

</view>

<view type="1">

  <file token="LRIbbUh1BoYOIqx77JvcNZVinMb" name="lmslim-0.3.1+das.opt3.fb67d47.dtk2604-cp310-cp310-linux_x86_64.whl"/>

</view>

<view type="1">

  <file token="BOVabBfeEo1CqHxfVCKceJDDnVg" name="flash_mla-1.0.0+das.opt1.dtk2604.20260210.g4742da75-cp310-cp310-manylinux_2_28_x86_64.whl"/>

</view>

**d节点推荐使用**

镜像：harbor.sourcefind.cn:5443/dcu/admin/base/vllm:0.9.2-ubuntu22.04-dtk26.04-0130-py3.10-20260204

加载方式：进入到乌镇研发节点/public/opendas/ArchivedFile/docker-images/das18-20260204/

<block-ref id="OH3bd1kuDoTUJkx9bBbc3r0mnUh"/>

<file token="VLMObKaPRobJUWxlU0Vcwaq1nNf" name="vllm-0.9.2+das.opt3.d2feb10.dtk2604-cp310-cp310-linux_x86_64.whl"/>

<block-ref id="EfcFdlarVoVhpmxIddXcI0FHnef"/>

<file token="FJ5NbMtuAoABvcxA5O2c2zOvnPe" name="lmslim-0.3.1+das.opt3.fb67d47.dtk2604-cp310-cp310-linux_x86_64.whl"/>

<block-ref id="T9PTd49yYoh98lxdSmMcAzBqnmb"/>

<file token="G741brF13ocekRxjY7McPz5QnBh" name="lightop-0.6.0+das.dtk2604.2c5fa2c-cp310-cp310-linux_x86_64.whl"/>

<block-ref id="J6umdWg6qob972xPuXCcigIlnSh"/>

<file token="B7UPbp346o9vsGxOYXEciCypnre" name="flash_mla-1.0.0+das.opt1.dtk2604.20260210.g4742da75-cp310-cp310-manylinux_2_28_x86_64.whl"/>

需要替换下脚本文件：

<block-ref id="QyGRdMjSQoFNGrx2TyqcipUAnId"/>

<file token="F28sbipgwo2n6Sx27djcRMRTnTh" name="flashmla_py.tar"/>

替换位置1：/usr/local/lib/python3.10/dist-packages/vllm/v1/attention/backends/mla/flashmla.py

替换位置2：/usr/local/lib/python3.10/dist-packages/vllm/attention/ops/flashmla.py

替换说明：
```plaintext
vllm-attention-ops下面的文件替换/usr/local/lib/python3.10/dist-packages/vllm/attention/ops/flashmla.py
vllm-v1-backends-mla下面的文件替换/usr/local/lib/python3.10/dist-packages/vllm/v1/attention/backends/mla/flashmla.py
```


## 元宝20251201日-bw1000节点（阶段性测试环境）

docker pull image.sourcefind.cn:5000/dcu/admin/base/vllm:0.9.2-ubuntu22.04-dtk25.04.2-py3.10

镜像id：c0fa6efcf973
```bash
docker run -it --name xxx \
--privileged \
--shm-size=1024G \
--device=/dev/kfd --device=/dev/dri/ \
--cap-add=SYS_PTRACE \
--security-opt seccomp=unconfined \
--ulimit memlock=-1:-1 \
--ipc=host \
--network host \
--group-add video \
-v /opt/hyhal:/opt/hyhal:ro \
-v xxx:xxxx \
c0fa6efcf973 \
/bin/bash
```

测试依赖：（先卸载在安装）

<view type="1">

  <file token="Mw9wbzAaWoXWtLxyFyzc41bHnxd" name="vllm-0.9.2+das.opt1.511eecc.dtk25042-cp310-cp310-linux_x86_64.whl"/>

</view>

<view type="1">

  <file token="ZAysbPqmooLlesxcMyMc0bUynCf" name="lightop-0.6.0+das.dtk25042.03c577f-cp310-cp310-linux_x86_64.whl"/>

</view>

<view type="1">

  <file token="ArNobFX9GoJMstxmJ6UcCOkcnrc" name="flash_mla-1.0.0+das.opt1.dtk2504-cp310-cp310-manylinux_2_28_x86_64.whl"/>

</view>

<view type="1">

  <file token="HAWybyZSzootAhxY9iacuF19nNg" name="lmslim-0.3.1+das.opt1.dtk25042-cp310-cp310-linux_x86_64.whl"/>

</view>

# 启动脚本以及环境变量（pd分离部署）

<callout emoji="musical_keyboard" background-color="light-orange" border-color="light-orange">

下面是一部分部署方式，更多可参考[[飞书知识图谱/资源/PD分离的使用--P2PNCCL方案--cOLeYnFe|PD分离的使用--P2PNCCL方案]]
</callout>

## pd非对称切分部署方式

<callout emoji="💡" background-color="light-orange" border-color="light-orange">

（下面是“元宝最优性能部署方式”举例，示例中的节点是乌镇千卡节点 node036 node042....
</callout>

### p：tp2pp4；d：tp8（单实例单机的任意切分方式（满足D的tp>=P的tp)使用。新版本支持。）

下面

#### 代理：
```plaintext
python3 disagg_proxy_p2p_nccl_xpyd.py
特别注意，这里如果服务重启，代理也需要重启
```

<block-ref id="CVH2dpiXJoRb55x1NM2ccLmTnKd"/>

<file token="ZQc6bO9zVoJthoxAjM7cMVsunPd" name="disagg_proxy_p2p_nccl_xpyd.py"/>

#### **P的运行指令和环境变量**：

<text bgcolor="light-red">与以往不同点：enable_asymmetric_p2p：是否是非对称切分；remote_tp_size：D的tpsize；remote_pp_size：D的ppsize </text>
```json {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
vllm serve /module/DeepSeek-R1-Channel-INT8 \
 --trust-remote-code \
 -q slimquant_marlin \
 -tp 2 -pp 4 \
 --port 20012 \
 --dtype bfloat16 \
 --max-model-len 40960 \
 --max-num-seqs 256 \
 --max-num-batched-tokens 6144 \
 --max-seq-len-to-capture 6144 \
 --disable-log-requests \
 --no-enable-prefix-caching \
 --enable-chunked-prefill \
 --block-size 64 \
 --gpu-memory-utilization 0.90 \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 2}' \
 --kv-cache-dtype fp8_e4m3 \
 --enforce-eager \
 --kv-transfer-config '{"kv_connector":"P2pNcclConnector","kv_role":"kv_producer","kv_buffer_size":"1e4","kv_port":"21001","kv_connector_extra_config":{"enable_asymmetric_p2p":true,"remote_tp_size":8,"remote_pp_size":1,"proxy_ip":"10.16.1.36","proxy_port":"30006","http_port":"20012","send_type":"PUT_ASYNC"}}'
```

环境变量：
```bash
rm -rf ~/.cache
rm -rf ~/.triton
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=3
export VLLM_RANK1_NUMA=1
export VLLM_RANK2_NUMA=1
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=7
export VLLM_RANK5_NUMA=5
export VLLM_RANK6_NUMA=5
export VLLM_RANK7_NUMA=4

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=3

export VLLM_REJECT_SAMPLE_OPT=1
export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1

# 可选项
export VLLM_ZERO_OVERHEAD=0
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=1
export VLLM_SPEC_DECODE_EAGER=1
export VLLM_USE_GLOBAL_CACHE13=1
export VLLM_FUSED_MOE_CHUNK_SIZE=16384

export VLLM_USE_FLASH_MLA_FP8=1（不支持）
export USE_FUSED_RMS_QUANT=1

export VLLM_CUSTOM_CACHE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1

export VLLM_USE_FUSED_FILL_RMS_CAT=1
export VLLM_USE_FLASH_ATTN_FP8=1

export VLLM_ZERO_OVERHEAD_ENHANCE=0
export VLLM_USE_FUSED_QA_KVA_GEMM=1

export VLLM_ENABLE_SHARED_EXPERTS_FUSION=1
export VLLM_USE_FUSED_DTBMM=1
export VLLM_USE_CAT_MLA=1

export VLLM_FUSE_CAT_AND_CAST_FP8=1
export VLLM_PP_LAYER_PARTITION=16,15,15,15

export VLLM_FUSED_GATHER_CACHE_CONVERT_FP8=1
export VLLM_FUSED_RN_ROPE_INT8_QUANT=1

export VLLM_HOST_IP=10.16.1.36
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1
export NCCL_IB_HCA=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1
```

#### D的运行指令和环境变量：
```json {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
vllm serve /module/DeepSeek-R1-Channel-INT8 \
 --trust-remote-code \
 -q slimquant_marlin \
 -tp 8 \
 --port 20020 \
 --dtype bfloat16 \
 --max-model-len 40960 \
 --max-num-seqs 256 \
 --max-num-batched-tokens 6144 \
 --max-seq-len-to-capture 6144 \
 --disable-log-requests \
 --no-enable-prefix-caching \
 --enable-chunked-prefill \
 --block-size 64 \
 --gpu-memory-utilization 0.90 \
 --compilation-config '{"full_cuda_graph": true}' \
 --speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 2}' \
 --kv-cache-dtype fp8_e4m3 \
 --kv-transfer-config '{"kv_connector":"P2pNcclConnector","kv_role":"kv_consumer","kv_buffer_size":"1e9","kv_port":"22000","kv_connector_extra_config":{"proxy_ip":"10.16.1.36","proxy_port":"30006","http_port":"20020","send_type":"PUT_ASYNC"}}'
```

<callout emoji="skull" background-color="light-orange" border-color="light-orange">

注意点：如果D的切分方式设置了VLLM_PP_LAYER_PARTITION这个参数：

需要在P节点设置“VLLM_PP_LAYER_PARTITION_D”和D的“VLLM_PP_LAYER_PARTITION”保持一致
</callout>

环境变量
```bash
rm -rf ~/.cache
rm -rf ~/.triton
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_NUMA_BIND=1
export VLLM_RANK0_NUMA=0
export VLLM_RANK1_NUMA=0
export VLLM_RANK2_NUMA=0
export VLLM_RANK3_NUMA=0
export VLLM_RANK4_NUMA=1
export VLLM_RANK5_NUMA=1
export VLLM_RANK6_NUMA=1
export VLLM_RANK7_NUMA=1

export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export W8A8_SUPPORT_METHODS=3

export VLLM_REJECT_SAMPLE_OPT=1
export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1

# 可选项
export VLLM_ZERO_OVERHEAD=1
export VLLM_USE_LIGHTOP_RMS_ROPE_CONCAT=1
export VLLM_SPEC_DECODE_EAGER=1
export VLLM_USE_GLOBAL_CACHE13=1
export VLLM_FUSED_MOE_CHUNK_SIZE=16384

export VLLM_USE_FLASH_MLA_FP8=1
export USE_FUSED_RMS_QUANT=1

export VLLM_CUSTOM_CACHE=1
export HIP_KERNEL_EVENT_SYSTENFENCE=1

export VLLM_USE_FUSED_FILL_RMS_CAT=1
export VLLM_USE_FLASH_ATTN_FP8=1

export VLLM_ZERO_OVERHEAD_ENHANCE=1
export VLLM_USE_FUSED_QA_KVA_GEMM=1

export VLLM_ENABLE_SHARED_EXPERTS_FUSION=1
export VLLM_USE_FUSED_DTBMM=1 
export VLLM_USE_CAT_MLA=1

export VLLM_HOST_IP=$(hostname -I | awk '{print $1}')
export NCCL_SOCKET_IFNAME=enp33s0f3u1
export GLOO_SOCKET_IFNAME=enp33s0f3u1
export NCCL_IB_HCA=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1
```


## pd对称切分部署方式 （单实例多机）

<callout emoji="tada" background-color="light-orange" border-color="light-orange">

单实例多机相较于单实例单机，不同在于一个p或者d需要至少两台节点部署，一般使用ray部署

执行步骤是：

先设置好环境变量（建议设置在容器的~/.bashrc下面），然后重启容器（激活环境变量）；

启动ray命令；<text underline="true">这里注意如果先起ray，在设置环境变量，会造成环境变量设置失效</text>。

先执行代理，在执行p、d节点起在线服务命令（可以同时执行p d，不需要先等p执行完，在起d。）
</callout>

### 代理：

<block-ref id="HXQsdfq9RokSjpx7YZLcnHp5nqe"/>

<file token="HSW8bx6VFoHhO8xnsGLc4Uhynxh" name="disagg_proxy_p2p_nccl_xpyd_mult_mac.py"/>

<block-ref id="BIMad8fUmo2xeUxmkodcyIaTnhf"/>

<file token="GonObEM5koEHVuxmk3QcmXFJnep" name="disagg_proxy_p2p_nccl_xpyd.py"/>

```plaintext
在P的节点执行代理如下
cd vllm/examples/online_serving/disaggregated_serving_p2p_nccl_xpyd
python3 disagg_proxy_p2p_nccl_xpyd_mult_mac.py  # 最新版本执行，老版本没有这个文件，就执行disagg_proxy_p2p_nccl_xpyd.py
特别注意，这里如果服务重启，代理也需要重启
```

代理端口设置说明：

<image token="XEvybSel7oyimwxWndWc25OSnje" width="1189" height="897" align="left"/>

### p节点设置

<callout emoji="skull" background-color="light-orange" border-color="light-orange">

这里是单实例多机的部署方式，需要设置ray。单实例单机部署不需要设置ray以及对应的环境变量

举例：p节点有两个13.13.2.11、13.13.2.23，其中13.13.2.11作为p节点中的ray头节点

注意：这里的“13.13.2.11”是头节点的ip，需要和头节点环境变量中的“VLLM_HOST_IP”一致
</callout>

```plaintext
head节点执行：
ray start --head --node-ip-address=13.13.2.11  --port=6801 --num-gpus=8 --num-cpus=32

worker节点执行：
ray start --address='13.13.2.11:6801' --num-gpus=8 --num-cpus=32
```

<text bgcolor="green">enable_multiple_machines:true：是否是跨机的这里P和D的服务都要设置，只要有一个跨机，就要设置true</text>；<text bgcolor="green">enable_asymmetric_p2p：是否是非对称切分，若p和d的切分方式不同，则设置成true</text>；<text bgcolor="green">remote_tp_size：D的tpsize</text>；<text bgcolor="green">remote_pp_size：D的ppsize</text>
```sql {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
vllm serve /models/vllm-w8a8-models/DeepSeek-R1-Channel-INT8 \
--trust-remote-code \
--dtype bfloat16 \
--port 20005 \
--max-model-len 51200 \
-tp 8 -pp 2 \
--max-seq-len-to-capture 32768 \
--distributed-executor-backend ray \
--gpu-memory-utilization 0.8 \
--max-num-seqs 256 \
--disable-log-requests \
--block-size 64 \
--enable-chunked-prefill \
--enable-prefix-caching \
--speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 2}' \
-q slimquant_marlin \
--enforce-eager \
--kv-transfer-config '{"kv_connector":"P2pNcclConnector","kv_role":"kv_producer","kv_buffer_size":"1e4","kv_port":"21001","kv_connector_extra_config":{"enable_multiple_machines":true, "enable_asymmetric_p2p":true,"remote_tp_size":16,"remote_pp_size":1,"proxy_ip":"13.13.2.11","proxy_port":"30002","http_port":"20005","send_type":"PUT_ASYNC"}}'
```

命令参数说明：

"proxy_ip"需要设置成p节点中头节点的ip，这里举例为13.13.2.11；

"proxy_port"：要和代理的服务端口一致；比如30002；

"http_port"：要和vllm起服务的“--port”一致；比如20005；

### d节点设置

ray设置，举例：d节点有两个13.13.2.21 13.13.2.22，其中13.13.2.21作为d节点中的ray头节点
```plaintext
head节点执行：
ray start --head --node-ip-address=13.13.2.21  --port=6802 --num-gpus=8 --num-cpus=32

worker节点执行：
ray start --address='13.13.2.21:6802' --num-gpus=8 --num-cpus=32
```

注意：这里的“13.13.2.21”是头节点的ip，需要和头节点环境变量中的“VLLM_HOST_IP”一致

启动脚本：（需要换成实际模型的路径）

*1p8d测试，只需要按照现有的d节点脚本进行增加，每个d节点脚本中的“--port”要和“http_port”一样，且不同d节点脚本间的“kv_port”、“--port”和“http_port”不能一样，并且注意修改环境变量“IP_CONFIG_FILE”的ip_config.txt内容：需要向下换行补充额外添加的d节点ip：如下所示*

<image token="XCD2bl98eosswIxbm3ac9SmenSh" width="963" height="474" align="left"/>

# d1节点
```sql {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
vllm serve /models/vllm-w8a8-models/DeepSeek-R1-Channel-INT8 \
--trust-remote-code \
--dtype bfloat16 \
--port 20006 \
--max-model-len 51200 \
-tp 8 -pp 2 \
--max-seq-len-to-capture 32768 \
--distributed-executor-backend ray \
--gpu-memory-utilization 0.9 \
--max-num-seqs 256 \
--disable-log-requests \
--block-size 64 \
--enable-chunked-prefill \
--enable-prefix-caching \
--compilation-config '{"full_cuda_graph": true}' \
--speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 2}' \
-q slimquant_marlin \
--kv-transfer-config '{"kv_connector":"P2pNcclConnector","kv_role":"kv_consumer","kv_buffer_size":"1e9","kv_port":"22001","kv_connector_extra_config":{"proxy_ip":"13.13.2.11","proxy_port":"30002","http_port":"20006","send_type":"PUT_ASYNC"}}'

```

# d2节点
```json {wrap}
rm -rf ~/.cache
rm -rf ~/.triton
vllm serve /models/vllm-w8a8-models/DeepSeek-R1-Channel-INT8 \
--trust-remote-code \
--dtype bfloat16 \
--port 20007 \
--max-model-len 51200 \
-tp 8 -pp 2 \
--max-seq-len-to-capture 32768 \
--distributed-executor-backend ray \
--gpu-memory-utilization 0.9 \
--max-num-seqs 256 \
--disable-log-requests \
--block-size 64 \
--enable-chunked-prefill \
--enable-prefix-caching \
--compilation-config '{"full_cuda_graph": true}' \
--speculative_config '{"method": "deepseek_mtp", "num_speculative_tokens": 2}' \
-q slimquant_marlin \
--kv-transfer-config '{"kv_connector":"P2pNcclConnector","kv_role":"kv_consumer","kv_buffer_size":"1e9","kv_port":"22002","kv_connector_extra_config":{"proxy_ip":"13.13.2.11","proxy_port":"30002","http_port":"20007","send_type":"PUT_ASYNC"}}'
```

# d3

# d4

# d5

......

命令参数说明：

"proxy_ip"需要设置成p节点中头节点的ip，这里举例为13.13.2.11；

"proxy_port"：要和代理的服务端口一致；比如30002；

"http_port"：要和vllm起服务的“--port”一致；比如20006；

### 环境变量：（建议放在各个容器的~/.bashrc里，重启容器激活，重新进入容器）
```plaintext
export IP_CONFIG_FILE=/mnt/tx_test/yuanbao/ip_config.txt
export NCCL_NCHANNELS_PER_PEER=2

export ALLREDUCE_STREAM_WITH_COMPUTE=1
export VLLM_HOST_IP=$(hostname -I | awk '{print $3}')
export NCCL_SOCKET_IFNAME=ens61f1np1
export GLOO_SOCKET_IFNAME=ens61f1np1
export HIP_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
unset NCCL_ALGO
export NCCL_MIN_NCHANNELS=16
export NCCL_MAX_NCHANNELS=16
export NCCL_NET_GDR_READ=1
export NCCL_IB_HCA=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1
export VLLM_USE_LIGHTOP_MOE_SUM_MUL_ADD=1 # moe sum + mul + add
export USE_FUSED_RMS_QUANT=1 # rmsnorm + quant
export VLLM_SPEC_DECODE_EAGER=1

export VLLM_USE_GLOBAL_CACHE13=1
export VLLM_FUSED_MOE_CHUNK_SIZE=16384
export VLLM_USE_OPT_OP=1 # 使用优化后的pa、rmsnorm、silu_and_mul kernel
export VLLM_ZERO_OVERHEAD=0
export VLLM_ENABLE_TBO=0
export VLLM_PP_LAYER_PARTITION=35,26

export Allgather_Base_STREAM_WITH_COMPUTE=1
export SENDRECV_STREAM_WITH_COMPUTE=1
export VLLM_USE_CUDA_GRAPH_SIZES=1 #(默认) 更新为1-24的capture cudagraph
export VLLM_USE_LIGHTOP_FILL_MOE_ALIGN=1 # fill + moe align 
export VLLM_USE_OPT_ZEROS=1 # 消除由于zero初始化引入的triton_
export VLLM_USE_PP_SYNC=1 # 解决pp压测出现的崩溃问题

export USE_FUSED_SILU_MUL_QUANT=1 # mlp siul_and_mul + quant
export VLLM_REJECT_SAMPLE_OPT=1 
export VLLM_USE_CAT_MLA=1 # cat + mla
export VLLM_USE_LIGHTOP=1 # rmsnorm + contiguous、deepseek qk rope
```

环境变量配置说明：

ip_config.txt文件要在p节点设置，例如：

<image token="QHnDbXW0AoYRlExZQ3ycNqhMnkf" width="874" height="163" align="left"/>

”VLLM_HOST_IP“要和”NCCL_SOCKET_IFNAME“ “GLOO_SOCKET_IFNAME”对应；

<image token="IDP6bfX4boJjyTxPI66csmHqnkb" width="985" height="1559" align="left"/>

”NCCL_IB_HCA“设置说明：执行命令：ibdev2netdev，设置状态为“Up”的ib，比如可以设置成“NCCL_IB_HCA=mlx5_2:1,mlx5_3:1,mlx5_4:1,mlx5_5:1,mlx5_6:1,mlx5_7:1,mlx5_8:1,mlx5_9:1”

<image token="Tz4dbGBJioXmqBxAJlpcTEwInWf" width="480" height="285" align="left"/>

### 客户端：
```sql
python path_to_file/benchmark_serving.py  \
    --backend vllm --metric-percentiles 99,95 \
    --model path_to_model  \
    --dataset-name custom \
    --dataset-path path_to_dataset \
    --custom-output-len 2048 \
    --endpoint /v1/completions \
    --custom-skip-chat-template \
    --num-prompts $num_prompts  \
    --max-concurrency $workers \
    --host 0.0.0.0 \
    --port 8000 \
    --random-range-ratio 0 \
    --result-dir output_ds
```

说明：

- path_to_model ：模型文件路径，要和服务端模型路径一致；
- path_to_dataset：实际数据集路径；
- num_prompts：数据集对话的条数；
- --max-concurrency：最大并发；比如可以设置“--max-concurrency 分别设置成4 6 7 8”
- --custom-output-len：设置最大的输出token数，根据数据集实际情况设置；比如可以设置成 “--custom-output-len 2048”

### 预热：（客户端执行的时候，有卡顿需要先执行小批量数据预热）

在pd服务起完之后，需要先运行一个小批量数据，预热下，建立pd之间的通信，防止测试卡顿；
```sql {wrap}
python benchmark_serving.py  \
--model /models/vllm-w8a8-models/DeepSeek-R1-Channel-INT8 \
--backend openai  \
--dataset-name random \
--random-input-len 128 \
--random-output-len 20 \
--num-prompts 8 \
--max-concurrency 8 \
--metric-percentiles 99,95 \
--host 0.0.0.0 \
--ignore-eos \
--port 10002
```

## 关联资源

- [[飞书知识图谱/资源/tx元宝DeepSeek测试--cAWcpnxs|tx元宝DeepSeek测试]] · 飞书源链接不可用
- [[飞书知识图谱/资源/PD分离的使用--P2PNCCL方案--cOLeYnFe|PD分离的使用--P2PNCCL方案]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/崩溃|崩溃]]（issue）
- [[飞书知识图谱/实体/并发|并发]]（metric）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-INT8|DeepSeek-R1-Channel-INT8]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/TBO|TBO]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

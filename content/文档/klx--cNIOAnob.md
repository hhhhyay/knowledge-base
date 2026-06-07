---
title: klx
tags:
  - AI
  - LLM
  - Benchmark
model:
  - kimi-2.5
  - kimi-2.5-w4a16
  - kimi-2.5-w4a8
  - kimi-2.5-w8a8
  - Kimi-K2.5
  - Kimi-K2.5_w4a8
  - Kimi-K2.5_w4a8_gsp_
  - Kimi-K2.5_w4a8.tar.gz
  - Kimi-K2.5_w8a8
  - kimi-k2.5-w8a8.tar.gz
  - Kimi-K25-eagle3
framework:
  - Mooncake
  - PyTorch
  - SGLang
issue_type:
  - 精度问题
source: feishu
feishu_token: YBV3dLBycoUlGwxnyV3cNIOAnob
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/YBV3dLBycoUlGwxnyV3cNIOAnob
last_synced_at: 2026-06-07T17:07:53.393Z
---

# klx

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/YBV3dLBycoUlGwxnyV3cNIOAnob)

 
#  kimi-2.5 推理测试

## 权重下载

### 10.1.1  kimi-2.5-w4a8 权重下载

```plaintext
wget -c https://klx-public.bj.bcebos.com/v1/luzhenghua/model/Kimi-K2.5_w4a8.tar.gz?authorization=bce-auth-v1%2FALTAKSzVyYuuyNAqY5Ybr4J8qB%2F2026-04-01T08%3A12%3A11Z%2F-1%2Fhost%2F411c6a68a6d08dec8e3fe1a7efea512520e0713364fbe05eedc86d17caf1aad7
```


 
### 10.1.2  kimi-2.5-w8a8 权重下载

```plaintext
wget -c https://klx-public.bj.bcebos.com/v1/luzhenghua/model/kimi-k2.5-w8a8.tar.gz?authorization=bce-auth-v1%2FALTAKSzVyYuuyNAqY5Ybr4J8qB%2F2026-04-01T08%3A11%3A33Z%2F-1%2Fhost%2Ffe993b33457e061f5a7732e8a36b6551bf2a2fa4d8b6ac9478c7c2f241f5c54c 
```


 
### 10.1.3 kimi-2.5-w4a16权重下载

```plaintext
modelscope download --model moonshotai/Kimi-K2.5
```


 
## docker容器镜像

### 10.2.1 kimi-2.5-w4a8 容器镜像

```plaintext
#腾讯云镜像
docker pull ccr.ccs.tencentyun.com/taco/infer-poc:sglang-p800-pd-disagg-056_20260410_3566
```


 
```plaintext
#容器启动脚本,映射文件需要自行修改
 #!/bin/bash
 
 readonly CONTAINER_NAME="sglang_0410_3566_kimi2.5"
 
 readonly DOCKER_IMAGE="ccr.ccs.tencentyun.com/taco/infer-poc:sglang-p800-pd-disagg-056_20260410_3566"
 readonly Workspace=${PWD}
 XPU_NUM=8
 DOCKER_DEVICE_CONFIG=" "
 if [ $XPU_NUM -gt 0 ]; then
         for ((idx=0; idx<=$XPU_NUM-1; idx++)); do
                 DOCKER_DEVICE_CONFIG+=" --device=/dev/xpu${idx}:/dev/xpu${idx} "
         done
 fi
 DOCKER_DEVICE_CONFIG+=" --device=/dev/xpuctrl:/dev/xpuctrl "
 docker run -itd ${DOCKER_DEVICE_CONFIG} \
         --privileged \
         --net=host \
         --cap-add=SYS_PTRACE --security-opt seccomp=unconfined \
         --tmpfs /dev/shm:rw,nosuid,nodev,exec,size=32g \
         --cap-add=SYS_PTRACE \
         -v ${Workspace}:/work \
         -v /data2:/data2 \
         --name ${CONTAINER_NAME} \
         -w /work \
         ${DOCKER_IMAGE} /bin/bash                            
```


 
### 10.2.2  kimi-2.5-w8a8 、 kimi-2.5-w4a16、PD分离容器镜像

```plaintext
wget -c https://klx-public.bj.bcebos.com/v1/kcv/D/dockers/docker_sglang-p800-pd-disagg-056%3A20260329_319.tar
```


 
```plaintext
#容器启动脚本
readonly CONTAINER_NAME="sglang_013_0323"
readonly DOCKER_IMAGE="iregistry.baidu-int.com/xpu/sglang-p800-pd-disagg-rc-056:20260323_43"
readonly Workspace=${PWD}
XPU_NUM=8
DOCKER_DEVICE_CONFIG=" "
if [ $XPU_NUM -gt 0 ]; then
for ((idx=0; idx<=$XPU_NUM-1; idx++)); do
DOCKER_DEVICE_CONFIG+=" --device=/dev/xpu${idx}:/dev/xpu${idx} "
done
fi
DOCKER_DEVICE_CONFIG+=" --device=/dev/xpuctrl:/dev/xpuctrl "
docker run -itd ${DOCKER_DEVICE_CONFIG} \
--privileged \
--net=host \
--cap-add=SYS_PTRACE --security-opt seccomp=unconfined \
--tmpfs /dev/shm:rw,nosuid,nodev,exec,size=32g \
--cap-add=SYS_PTRACE \
-v ${Workspace}:/home \
-v /data1:/data1 \
--name ${CONTAINER_NAME} \
-w /home \
${DOCKER_IMAGE} /bin/bash
 
```


 
## 启动服务

### 10.3.1 kimi-2.5-w4a8  单机8卡模型启动服务

```plaintext
#在容器内部
cd /workspace/kimi_k25/w4a8 
bash run_server.sh  #需要修改模型路径
```


 
run_server.sh 脚本内容如下，需要自行修改MASTER_IP 和 MODEL_DIR
```plaintext
#!/bin/bash
 
source "/workspace"/common/common.sh
# Kimi K25 公共环境变量
# LD_LIBRARY_PATH
export LD_LIBRARY_PATH=${CONDA_PREFIX}/lib/python3.10/site-packages/xtorch_ops:${CONDA_PREFIX}/lib/python3.10/site-packages/nvidia/cuda_nvrtc/lib:${CONDA_PREFIX}/lib/python3.10/site-packages/torch_xmlir/:/usr/local/cuda-11.7/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
 
# 设备可见性
export XPU_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
export CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
 
# 稳定性相关
export SGLANG_CLEAN_REQUEST_WHEN_RETRACT=1
export SGLANG_SET_CPU_AFFINITY=1
 
# BKCL 基础配置
export BKCL_ENABLE_XDR=1
export BKCL_TREE_THRESHOLD=1048576
export BKCL_RDMA_VERBS=1
export CUDA_DEVICE_ORDER="OAM_ID"
# Deepep 相关
export XSHMEM_SYMMETRIC_SIZE=4294967296
export XSHMEM_MODE=1
export XSHMEM_QP_NUM_PER_RANK=24
export BKCL_RDMA_NICS=bond0,bond1,bond2,bond3,bond4,bond5,bond6,bond7
#export BKCL_RDMA_NICS=eth1,eth1,eth2,eth2,eth3,eth3,eth4,eth4
 
# Mooncake 相关
export NCCL_IB_GID_INDEX=3
export CUDA_ENABLE_P2P_NO_UVA=1
# Mooncake 拓扑配置
export MC_CUSTOM_TOPO_JSON=/workspace/nic_priority_matrix_test.json
 
# xSGL 框架相关
export XSGL_TRANSPOSE_MATMUL_WEIGHT=1
export XSGL_INTERTYPE_BFP16=1
export ENABLE_FAST_BFP16_ATTN=1
export ENABLE_FAST_BFP16=1
export XSGL_NORMAL_TBO=1
export XSGL_XDNN_QUANT=1
export XSGL_FAST_SWIGLU=1
export USE_FAST_BFP16_FC=1
export XSGL_USE_DEEP_GEMM_BMM=1
export XSGL_INT8_LM_HEAD=1
export XINFER_QUANT_SDNN=1
export XINFER_SIGMOID_GROUP_TOPK_UNSTABLE=1
export XSGL_FUSE_RMS_NORM_QUANT=1
export XPU_FLASH_ATTENTION_DECODER_USE_BALANCE=1
 
# w4a8 量化参数，匹配到的模块使用w8a8，否则使用w4a8
export W4A8_W8_PATTERN=".*self_attn\.q_a_proj*,.*self_attn\.q_b_proj.*,.*self_attn\.attn.*,.*self_attn\.kv_b_proj.*,.*self_attn\.fused_qkv_a_proj_with_mqa.*,.*self_attn\.o_proj.*,.*mlp\.gate_up_proj.*,.*mlp\.down_proj.*,.*mlp\.shared_experts\.down_proj.*,.*mlp\.shared_experts\.gate_up_proj.*"
export PADDING_EXTEND_SEQ_LENS_FOR_CHUNNED_KV=1
export SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
 
# xPytorch 相关
export XMLIR_FORCE_USE_XPU_GRAPH=1
export XMLIR_ENABLE_FAST_FC=true
export XPUAPI_SDNN_BF16_ROUND_MODE=3
#需要自行修改MASTER_IP 和 MODEL_DIR
MASTER_IP=${master_ip:-10.0.0.37}
MODEL_DIR=${model_dir:-/data2/models/Kimi-K2.5_w4a8/}
PORT=${port:-30000}
 
export XBLAS_AUTOTUNE_FILE="/workspace/kimi_k25/fc_fusion_autotune_20260409_200501"
export XPU_FLASH_ATTENTION_DECODER_DYNAMIC_PARTITION=1
 
dp_size=1
cuda_graph_max_bs=32
cuda_graph_bs=$(seq -s ' ' 1 $cuda_graph_max_bs)
max_running_requests=$((dp_size * cuda_graph_max_bs))
 
nohup python3 -m sglang.launch_server \
                --host 0.0.0.0 \
                --port $PORT \
                --model-path "$MODEL_DIR" \
                --trust-remote-code \
                --attention-backend kunlun \
                --mm-attention-backend fa3 \
                --tp-size 8 \
                --ep-size 8 \
                --dp-size 1 \
                --quantization w4a8_int4 \
                --page-size 128 \
                --chunked-prefill-size 16384 \
                --max-running-requests $max_running_requests \
                --mem-fraction-static 0.82 \
                --cuda-graph-max-bs $cuda_graph_max_bs \
                --cuda-graph-bs $cuda_graph_bs \
                --disable-shared-experts-fusion \
                --tool-call-parser kimi_k2 \
                --reasoning-parser kimi_k2 \
                --kv-cache-dtype int8 \
                --quantization-param-path /data2/models/kimi_k25_int8_kv_scale/ \
                --watchdog-timeout 3000000 \
                >log_tp8_ep8_dp1_chunk16k_trace_$(date +"%Y%m%d").log 2>&1 &
```


/workspace/nic_priority_matrix_test.json 配置文件需要修改
```plaintext
{
  "cuda:0": [
    [   
      "mlx5_bond_0"
    ],  
    []  
  ],  
  "cuda:1": [
    [   
      "mlx5_bond_1"
    ],  
    []  
  ],  
  "cuda:2": [
    [   
      "mlx5_bond_2"
    ],  
    []  
  ],  
  "cuda:3": [
    [   
      "mlx5_bond_3"
    ],  
    []  
  ],  
  "cuda:4": [
    [   
      "mlx5_bond_4"
    ],  
    []  
  ],  
  "cuda:5": [
    [   
      "mlx5_bond_5"
    ],  
    []  
  ],  
  "cuda:6": [
    [   
      "mlx5_bond_6"
    ],  
    []  
  ],  
  "cuda:7": [
    [   
      "mlx5_bond_7"
    ],  
    []  
  ]
}
```


 
### 10.3.2 kimi-2.5-w4a8 模型 1P1D 分离启动服务

P节点启动服务命令
```plaintext
# 在容器内部
 cd /workspace/kimi_k25/1p1d
 bash  run_p_server.sh 
```


 
run_p_server.sh 脚本内容
```plaintext
#需要修改模型路径，以及nic_priority_matrix.json内容，参考deepseek内容
#!/bin/bash
MODEL_DIR=${model_dir:-/data1/models/Kimi-K2.5_w4a8/}
source "/workspace"/common/common.sh
NNODES=1
RANK=${rank:-0}
DIST_PORT=6000
PORT=30000
 
export MC_CUSTOM_TOPO_JSON=/workspace/nic_priority_matrix.json
export LD_LIBRARY_PATH=${CONDA_PREFIX}/lib/python3.10/site-packages/xtorch_ops:${CONDA_PREFIX}/lib/python3.10/site-packages/nvidia/cuda_nvrtc/lib:${CONDA_PREFIX}/lib/python3.10/site-packages/torch_xmlir/:/usr/local/cuda-11.7/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
# export LD_LIBRARY_PATH=/XMLIR/normal/xccl_0706/xccl_Linux_x86_64/so:$LD_LIBRARY_PATH
# export PYTHONPATH=/workspace/xSGL/python
 
# BKCL相关
export BKCL_ENABLE_XDR=1
export BKCL_RDMA_NICS=mlx5_bond_1,mlx5_bond_2,mlx5_bond_3,mlx5_bond_4,mlx5_bond_5,mlx5_bond_6,mlx5_bond_7,mlx5_bond_8
#export BKCL_RDMA_NICS=eth1,eth2,eth3,eth4,eth5,eth6,eth7,eth8
export BKCL_TREE_THRESHOLD=1048576
export BKCL_RDMA_VERBS=1
export CUDA_DEVICE_ORDER="OAM_ID"
 
# Mooncake相关
export NCCL_IB_GID_INDEX=3
export CUDA_ENABLE_P2P_NO_UVA=1
 
# envs from xSGL
export XSGL_TRANSPOSE_MATMUL_WEIGHT=1
export XSGL_INTERTYPE_BFP16=1
export ENABLE_FAST_BFP16_ATTN=1
export ENABLE_FAST_BFP16=1
export XSGL_NORMAL_TBO=1
export XSGL_XDNN_QUANT=1
export XSGL_FAST_SWIGLU=1
export USE_FAST_BFP16_FC=1
export XSGL_USE_DEEP_GEMM_BMM=1
export XSGL_INT8_LM_HEAD=1
export XINFER_QUANT_SDNN=1
export XINFER_SIGMOID_GROUP_TOPK_UNSTABLE=1
# flash_attention kernel balance
export XPU_FLASH_ATTENTION_DECODER_USE_BALANCE=1
# xSGL w4a8 量化参数，匹配到的模块使用w8a8，否则使用w4a8
export W4A8_W8_PATTERN=".*self_attn\.q_a_proj*,.*self_attn\.q_b_proj.*,.*self_attn\.attn.*,.*self_attn\.kv_b_proj.*,.*self_attn\.fused_qkv_a_proj_with_mqa.*,.*self_attn\.o_proj.*,.*mlp\.gate_up_proj.*,.*mlp\.down_proj.*,.*mlp\.shared_experts\.down_proj.*,.*mlp\.shared_experts\.gate_up_proj.*"
export PADDING_EXTEND_SEQ_LENS_FOR_CHUNNED_KV=1
export SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1
export XSGL_EAGLE3_MODEL_NO_QUANT=1
export SGLANG_ENABLE_SPEC_V2=True
export SGLANG_ENABLE_OVERLAP_PLAN_STREAM=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
 
# envs from xPytorch
export XMLIR_FORCE_USE_XPU_GRAPH=1
export XMLIR_ENABLE_FAST_FC=true
export XPUAPI_SDNN_BF16_ROUND_MODE=3
export DIST_MULTI_STREAM=true
export XMLIR_DIST_SINGLETON_STREAM=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
 
nohup python -m sglang.launch_server \
  --host 0.0.0.0 \
  --port $PORT \
  --model-path "$MODEL_DIR" \
  --attention-backend kunlun \
  --trust-remote-code \
  --disable-custom-all-reduce \
  --chunked-prefill-size 65536 \
  --page-size 128 \
  --max-running-requests 2 \
  --mem-fraction-static 0.8 \
  --max-prefill-tokens 65536 \
  --tp-size 8 \
  --ep-size 8 \
  --dtype float16 \
  --quantization w4a8_int4 \
  --tool-call-parser kimi_k2 \
  --reasoning-parser kimi_k2 \
  --moe-dense-tp-size 1 \
  --disaggregation-mode prefill \
  --disable-shared-experts-fusion \
  --watchdog-timeout 3000000 >log_p_$(date +"%Y%m%d").log 2>&1 &
```


 
D节点 服务启动命令
```plaintext
# 在容器内部
 cd /workspace/kimi_k25/1p1d
 bash  run_d_server.sh 
```


 run_d_server.sh 脚本内容
```plaintext
#需要修改模型路径，以及nic_priority_matrix.json内容，参考deepseek内容
#!/bin/bash
MODEL_DIR=${model_dir:-/data1/models/Kimi-K2.5_w4a8/}
source "/workspace"/common/common.sh
NNODES=1
RANK=${rank:-0}
DIST_PORT=6000
PORT=30000
 
export MC_CUSTOM_TOPO_JSON=/workspace/nic_priority_matrix.json
export LD_LIBRARY_PATH=${CONDA_PREFIX}/lib/python3.10/site-packages/xtorch_ops:${CONDA_PREFIX}/lib/python3.10/site-packages/nvidia/cuda_nvrtc/lib:${CONDA_PREFIX}/lib/python3.10/site-packages/torch_xmlir/:/usr/local/cuda-11.7/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
# export LD_LIBRARY_PATH=/XMLIR/normal/xccl_0706/xccl_Linux_x86_64/so:$LD_LIBRARY_PATH
# export PYTHONPATH=/workspace/icode/xSGL/python
 
# BKCL相关
export BKCL_ENABLE_XDR=1
export BKCL_RDMA_NICS=mlx5_bond_1,mlx5_bond_2,mlx5_bond_3,mlx5_bond_4,mlx5_bond_5,mlx5_bond_6,mlx5_bond_7,mlx5_bond_0
#export BKCL_RDMA_NICS=eth1,eth2,eth3,eth4,eth5,eth6,eth7,eth8
export BKCL_TREE_THRESHOLD=1048576
export BKCL_RDMA_VERBS=1
export CUDA_DEVICE_ORDER="OAM_ID"
 
# Mooncake相关
export NCCL_IB_GID_INDEX=3
export CUDA_ENABLE_P2P_NO_UVA=1
 
# envs from xSGL
export XSGL_TRANSPOSE_MATMUL_WEIGHT=1
export XSGL_INTERTYPE_BFP16=1
export ENABLE_FAST_BFP16_ATTN=1
export ENABLE_FAST_BFP16=1
export XSGL_NORMAL_TBO=1
export XSGL_XDNN_QUANT=1
export XSGL_FAST_SWIGLU=1
export USE_FAST_BFP16_FC=1
export XSGL_USE_DEEP_GEMM_BMM=1
export XSGL_INT8_LM_HEAD=1
export XINFER_QUANT_SDNN=1
export XINFER_SIGMOID_GROUP_TOPK_UNSTABLE=1
# flash_attention kernel balance
export XPU_FLASH_ATTENTION_DECODER_USE_BALANCE=1
# xSGL w4a8 量化参数，匹配到的模块使用w8a8，否则使用w4a8
export W4A8_W8_PATTERN=".*self_attn\.q_a_proj*,.*self_attn\.q_b_proj.*,.*self_attn\.attn.*,.*self_attn\.kv_b_proj.*,.*self_attn\.fused_qkv_a_proj_with_mqa.*,.*self_attn\.o_proj.*,.*mlp\.gate_up_proj.*,.*mlp\.down_proj.*,.*mlp\.shared_experts\.down_proj.*,.*mlp\.shared_experts\.gate_up_proj.*"
export PADDING_EXTEND_SEQ_LENS_FOR_CHUNNED_KV=1
export SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1
export XSGL_EAGLE3_MODEL_NO_QUANT=1
export SGLANG_ENABLE_SPEC_V2=True
export SGLANG_ENABLE_OVERLAP_PLAN_STREAM=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
 
# envs from xPytorch
export XMLIR_FORCE_USE_XPU_GRAPH=1
export XMLIR_ENABLE_FAST_FC=true
export XPUAPI_SDNN_BF16_ROUND_MODE=3
export DIST_MULTI_STREAM=true
export XMLIR_DIST_SINGLETON_STREAM=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
export XPU_FLASH_ATTENTION_DECODER_USE_BALANCE=1
nohup python -m sglang.launch_server \
  --host 0.0.0.0 \
  --port $PORT \
  --model-path "$MODEL_DIR" \
  --attention-backend kunlun \
  --trust-remote-code \
  --disable-custom-all-reduce \
  --chunked-prefill-size 8192 \
  --page-size 128 \
  --max-running-requests 256 \
  --mem-fraction-static 0.85 \
  --max-prefill-tokens 16384 \
  --tp-size 8 \
  --ep-size 8 \
  --dp-size 8 \
  --enable-dp-attention \
  --dtype float16 \
  --quantization w4a8_int4 \
  --tool-call-parser kimi_k2 \
  --reasoning-parser kimi_k2 \
  --moe-dense-tp-size 1 \
  --disaggregation-mode decode \
  --disable-shared-experts-fusion \
  --cuda-graph-max-bs 32 \
  --load-balance-method round_robin \
  --prefill-round-robin-balance \
  --watchdog-timeout 3000000 >log_d_$(date +"%Y%m%d").log 2>&1 &
```


 
连接PD节点脚本
```plaintext
# 在P节点 相同容器内部
 cd /workspace/kimi_k25/1p1d
 bash  start_minilb.sh 
```


 start_minilb.sh 脚本内容
```plaintext
#!/bin/bash
 
MASTER_P_IP=${master_p_ip:-10.0.0.37} #需要修改为自己P节点IP
MASTER_D_IP=${master_d_ip:-10.0.0.27} #需要修改为自己D节点IP
 
nohup python3 -m sglang_router.launch_router --pd-disaggregation \
  --prefill http://"${MASTER_P_IP}":30000 \
  --decode http://"${MASTER_D_IP}":30000 \
  --host 0.0.0.0 \
  --request-timeout-secs 100000 \
  --port 8025 > test_mini_lb.log 2>&1 &
```


 
### 10.3.3 kimi-2.5-w4a16  单机8卡模型启动服务

```plaintext
#在容器内部
cd /workspace/kimi_k25/w4a16 
bash run_server.sh  #需要修改模型路径
```


 
run_server.sh 脚本内容如下
```plaintext
#!/bin/bash
ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && cd ../../ && pwd)"
source "${ROOT_DIR}"/common/common.sh
 
MASTER_IP=${master_ip:-10.213.204.79}
# bos:/klx-public/shijinxiang/models/Kimi-K2.5_w4a8
MODEL_DIR=${model_dir:-/ssd2/models/Kimi-K2.5_w4a8/}
DRAFT_MODEL_DIR=${draft_model_dir:-/ssd3/models/Kimi-K25-eagle3/}
NNODES=1
RANK=${rank:-0}
DIST_PORT=6000
PORT=80
 
export MC_CUSTOM_TOPO_JSON=/workspace/nic_priority_matrix_test.json
export LD_LIBRARY_PATH=${CONDA_PREFIX}/lib/python3.10/site-packages/xtorch_ops:${CONDA_PREFIX}/lib/python3.10/site-packages/nvidia/cuda_nvrtc/lib:${CONDA_PREFIX}/lib/python3.10/site-packages/torch_xmlir/:/usr/local/cuda-11.7/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
 
# envs from xccl
export BKCL_ENABLE_XDR=1
#export BKCL_RDMA_NICS=bond2,bond2,bond3,bond3,bond4,bond4,bond5,bond5
export BKCL_RDMA_NICS=eth1,eth1,eth2,eth2,eth3,eth3,eth4,eth4
 
# export BKCL_RDMA_NICS=eth1,eth2,eth3,eth4,eth5,eth6,eth7,eth8
export BKCL_TREE_THRESHOLD=1048576
export BKCL_RDMA_VERBS=1
export CUDA_DEVICE_ORDER="OAM_ID"
# envs from xDeepEP
# export XSHMEM_MODE=1
# export XSHMEM_QP_NUM_PER_RANK=24
 
# envs from Mooncake
export NCCL_IB_GID_INDEX=3
export CUDA_ENABLE_P2P_NO_UVA=1
 
# envs from xSGL
export XSGL_TRANSPOSE_MATMUL_WEIGHT=1
export XSGL_INTERTYPE_BFP16=1
export ENABLE_FAST_BFP16_ATTN=1
export ENABLE_FAST_BFP16=1
export XSGL_NORMAL_TBO=1
export XSGL_XDNN_QUANT=1
export XSGL_FAST_SWIGLU=1
export USE_FAST_BFP16_FC=1
export XSGL_USE_DEEP_GEMM_BMM=1
export XSGL_INT8_LM_HEAD=1
export XINFER_QUANT_SDNN=1
export XINFER_SIGMOID_GROUP_TOPK_UNSTABLE=1
# flash_attention kernel balance
export XPU_FLASH_ATTENTION_DECODER_USE_BALANCE=1
# xSGL w4a8 量化参数，匹配到的模块使用w8a8，否则使用w4a8
export W4A8_W8_PATTERN=".*self_attn\.q_a_proj*,.*self_attn\.q_b_proj.*,.*self_attn\.attn.*,.*self_attn\.kv_b_proj.*,.*self_attn\.fused_qkv_a_proj_with_mqa.*,.*self_attn\.o_proj.*,.*mlp\.gate_up_proj.*,.*mlp\.down_proj.*,.*mlp\.shared_experts\.down_proj.*,.*mlp\.shared_experts\.gate_up_proj.*"
 
export XSGL_MOE_USE_INT4_W4A16=true
 
export PADDING_EXTEND_SEQ_LENS_FOR_CHUNNED_KV=1
export SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1
export XSGL_EAGLE3_MODEL_NO_QUANT=1
export SGLANG_ENABLE_SPEC_V2=True
export SGLANG_ENABLE_OVERLAP_PLAN_STREAM=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
 
# envs from xPytorch
export XMLIR_FORCE_USE_XPU_GRAPH=1
export XMLIR_ENABLE_FAST_FC=true
export XPUAPI_SDNN_BF16_ROUND_MODE=3
export DIST_MULTI_STREAM=true
export XMLIR_DIST_SINGLETON_STREAM=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
 
# # 启用xpu runtime profiler
# export XPU_ENABLE_PROFILER_TRACING=1
# export SGLANG_TORCH_PROFILER_DIR=/workspace/profile_log
# export DEEPEP_FORCE_SYNC=1
# export BKCL_FORCE_SYNC=1
# export XSGL_REDIRCT_LOG=1
# export XBLAS_MOE_FC_PRINT_TOKENS_PER_EXPERT=1
 
nohup \
python3 -m sglang.launch_server \
  --host 0.0.0.0 \
  --port $PORT \
  --model-path "$MODEL_DIR" \
  --attention-backend kunlun \
  --mm-attention-backend fa3 \
  --trust-remote-code \
  --tp-size 8 \
  --ep-size 8 \
  --page-size 64 \
  --dtype bfloat16 \
  --chunked-prefill-size 8192 \
  --quantization w4a16_int4 \
  --max-prefill-tokens 8192 \
  --max-running-requests 128 \
  --mem-fraction-static 0.85 \
  --disable-shared-experts-fusion \
  --cuda-graph-max-bs 128 \
  --tool-call-parser kimi_k2 \
  --reasoning-parser kimi_k2 \
  --watchdog-timeout 3000000  > log_$(date +"%Y%m%d").log 2>&1 &
  # --speculative-algorithm EAGLE3 \
  # --speculative-draft-model-path ${DRAFT_MODEL_DIR} \
  # --speculative-num-steps 3 \
  # --speculative-eagle-topk 1 \
  # --speculative-num-draft-tokens 4 \
```


 
### 10.3.3 kimi-2.5-w8a8  双机16卡模型启动服务

```plaintext
#在容器内部
cd /workspace/kimi_k25/w4a16 
bash run_2nodes.sh  #需要修改模型路径
 
脚本额外修改内容：
1）export BKCL_RDMA_NICS=bond2,bond2,bond3,bond3,bond4,bond4,bond5,bond5
修改为：export BKCL_RDMA_NICS=eth1,eth2,eth3,eth4,eth5,eth6,eth7,eth8
 
2）增加：export GLOO_SOCKET_IFNAME=${GLOO_SOCKET_IFNAME:-eth0}
 
3）MC_CUSTOM_TOPO_JSON=/workspace/nic_priority_matrix.json  对应的这个json文件也需要改一下
{
  "cuda:0": [
    [
      "mlx5_bond_0"
    ],
    []
  ],
  "cuda:1": [
    [
      "mlx5_bond_1"
    ],
    []
  ],
  "cuda:2": [
    [
      "mlx5_bond_2"
    ],
    []
  ],
  "cuda:3": [
    [
      "mlx5_bond_3"
    ],
    []
  ],
  "cuda:4": [
    [
      "mlx5_bond_4"
    ],
    []
  ],
  "cuda:5": [
    [
      "mlx5_bond_5"
    ],
    []
  ],
  "cuda:6": [
    [
      "mlx5_bond_6"
    ],
    []
  ],
  "cuda:7": [
    [
      "mlx5_bond_7"
    ],
    []
  ]
}
```


 
run_server.sh 脚本内容如下
```plaintext
#!/bin/bash
ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && cd ../../ && pwd)"
source "${ROOT_DIR}"/common/common.sh
 
# 客户会自行注入模型路径、主节点ip、指定node
DIST_PORT=6000
MASTER_IP=${master_ip:-10.213.207.141}
# bos:/klx-public/shijinxiang/models/Kimi-K2.5_w8a8/
MODEL_DIR=${model_dir:-/ssd3/models/Kimi-K2.5_w8a8/}
NNODES=2
RANK=${rank:-0}
PORT=${port:-30000}
 
export MC_CUSTOM_TOPO_JSON=/workspace/nic_priority_matrix_test.json
export LD_LIBRARY_PATH=${CONDA_PREFIX}/lib/python3.10/site-packages/xtorch_ops:${CONDA_PREFIX}/lib/python3.10/site-packages/nvidia/cuda_nvrtc/lib:${CONDA_PREFIX}/lib/python3.10/site-packages/torch_xmlir/:/usr/local/cuda-11.7/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
export GLOO_SOCKET_IFNAME=${GLOO_SOCKET_IFNAME:-eth0}
 
# envs from xccl
export BKCL_ENABLE_XDR=1
export BKCL_RDMA_NICS=bond2,bond2,bond3,bond3,bond4,bond4,bond5,bond5
#export BKCL_RDMA_NICS=eth1,eth2,eth3,eth4,eth5,eth6,eth7,eth8
export BKCL_TREE_THRESHOLD=1048576
export BKCL_RDMA_VERBS=1
export CUDA_DEVICE_ORDER="OAM_ID"
# envs from xDeepEP, do not set if not using DeepEP
export XSHMEM_MODE=1
export XSHMEM_QP_NUM_PER_RANK=24
# envs from xSGL for DeepEP
export SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK=32
 
# # envs from Mooncake
# export NCCL_IB_GID_INDEX=3
# export CUDA_ENABLE_P2P_NO_UVA=1
 
# envs from xSGL
export XSGL_TRANSPOSE_MATMUL_WEIGHT=1
export XSGL_INTERTYPE_BFP16=1
export ENABLE_FAST_BFP16_ATTN=1
export ENABLE_FAST_BFP16=1
export XSGL_NORMAL_TBO=1
export XSGL_XDNN_QUANT=1
export XSGL_FAST_SWIGLU=1
export USE_FAST_BFP16_FC=1
# w_kc, w_vc使用w8a8计算
export XSGL_USE_DEEP_GEMM_BMM=1
# lm_head也使用w8a8计算
export XSGL_INT8_LM_HEAD=1
# xtorch_ops里所有quant类算子使用sdnn部件
export XINFER_QUANT_SDNN=1
# 使用 topk_norm，性能更好
export XSGL_USE_MOE_SIGMOID_GROUP_TOPK_NORM=1
# 避免开chunk时padding导致精度问题
export PADDING_EXTEND_SEQ_LENS_FOR_CHUNNED_KV=1
# flash_attention kernel balance
export XPU_FLASH_ATTENTION_DECODER_USE_BALANCE=1
 
# envs from xPytorch
export XMLIR_FORCE_USE_XPU_GRAPH=1
export XMLIR_ENABLE_FAST_FC=true
export XPUAPI_SDNN_BF16_ROUND_MODE=3
export DIST_MULTI_STREAM=true
export XMLIR_DIST_SINGLETON_STREAM=1
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
 
# # 启用xpu runtime profiler
# export XPU_ENABLE_PROFILER_TRACING=1
# export SGLANG_TORCH_PROFILER_DIR=/workspace/profile_log
# export DEEPEP_FORCE_SYNC=1
# export BKCL_FORCE_SYNC=1
 
max_bs=${MAX_BS:-48}
cuda_graph_bs=$(seq -s ' ' 1 "$max_bs")
 
# --enable-single-batch-overlap \ TODO 支持SBO
# --init-expert-location round_robin \ 会有精度问题
# per_dp_chunk_size = --chunked-prefill-size // --dp-size
 
nohup python -m sglang.launch_server \
  --host 0.0.0.0 \
  --port "${PORT}" \
  --model-path "$MODEL_DIR" \
  --page-size 64 \
  --trust-remote-code \
  --attention-backend kunlun \
  --mm-attention-backend fa3 \
  --chunked-prefill-size 16384 \
  --nnodes "$NNODES" \
  --node-rank "$RANK" \
  --dist-init-addr "$MASTER_IP":$DIST_PORT \
  --max-running-requests 128 \
  --mem-fraction-static 0.92 \
  --max-prefill-tokens 65536 \
  --tp-size 16 \
  --ep-size 16 \
  --dp-size 4 \
  --enable-dp-attention \
  --moe-a2a-backend deepep \
  --deepep-mode auto \
  --moe-dense-tp-size 1 \
  --enable-dp-lm-head \
  --quantization w8a8_int8 \
  --dtype bfloat16 \
  --disable-shared-experts-fusion \
  --load-balance-method round_robin \
  --prefill-round-robin-balance \
  --cuda-graph-max-bs $max_bs \
  --cuda-graph-bs $cuda_graph_bs \
  --tool-call-parser kimi_k2 \
  --reasoning-parser kimi_k2 \
  --watchdog-timeout 3000000 > log_$(date +"%Y%m%d").log 2>&1 &
```


 
##### 启动方式：

```plaintext
# Node0
cd /workspace/kimi_k25/w4a16 
master_ip=<master ip> \
rank=0 \
model_dir=<模型实际路径> \
bash run_2nodes.sh
 
# Node1
cd /workspace/kimi_k25/w4a16 
master_ip=<master ip> \
rank=1 \
model_dir=<模型实际路径> \
bash run_2nodes.sh
 
# 注意两个节点master_ip相应，都是node0的IP
```


 
 
##  测试脚本

### 10.4.1 单机8卡、1P1D、双机TP通用测试脚本

```plaintext
#需要下载ShareGPT_V3_unfiltered_cleaned_split.json 数据集 到 /tmp 目录下
#!/bin/bash
set -euo pipefail
 
# ===================== 基础配置 =====================
MODEL_PATH="/data2/models/Kimi-K2.5_w4a8" # 需要修改模型路径
TS="$(date +%Y%m%d_%H%M%S)"
RESULT_BASE="./bench_results/Kimi-K2.5_w4a8_gsp_${TS}"
HOST="127.0.0.1"
PORT="30000"
 
# GSP (Generated Shared Prefix) 数据集参数
GSP_SYSTEM_PROMPT_LEN=54000
GSP_QUESTION_LEN=15000
GSP_OUTPUT_LEN=500
GSP_NUM_GROUPS=4
RANDOM_RANGE_RATIO=1
 
# 预热配置
WARMUP_ISL=128
WARMUP_OSL=128
WARMUP_NUM=8
WARMUP_CONCURRENCY=8
 
# ===================== 清理服务端 KV Cache =====================
# 通过 sglang 的 /flush_cache API 清理服务端的 prefix cache
# 确保每个 case 的第一个请求是冷启动（完整 prefill）
flush_server_cache() {
  echo ">>> 清理服务端 KV Cache..."
  local resp
  resp=$(curl -s -X POST "http://${HOST}:${PORT}/flush_cache" 2>/dev/null)
  echo ">>> 服务端 KV Cache 已清理 ✓ (${resp})"
  sleep 2  # 等待服务端完成清理
}
 
# ===================== 预热函数 =====================
# 说明：在正式测试前发送一批小请求，让引擎完成 CUDA Graph 捕获、KV Cache 分配等初始化
do_warmup() {
  echo ""
  echo "=========================================="
  echo " 开始预热 (warmup) - 使用 generated-shared-prefix 数据集"
  echo " ISL=${WARMUP_ISL} OSL=${WARMUP_OSL} num=${WARMUP_NUM} concurrency=${WARMUP_CONCURRENCY}"
  echo "=========================================="
 
  python3 -m sglang.bench_serving \
    --host "${HOST}" \
    --port "${PORT}" \
    --backend sglang \
    --model "${MODEL_PATH}" \
    --tokenizer "${MODEL_PATH}" \
    --dataset-name generated-shared-prefix \
    --random-range-ratio "${RANDOM_RANGE_RATIO}" \
    --gsp-num-groups "${GSP_NUM_GROUPS}" \
    --gsp-prompts-per-group "${WARMUP_NUM}" \
    --gsp-system-prompt-len "${WARMUP_ISL}" \
    --gsp-question-len "${WARMUP_OSL}" \
    --gsp-output-len 50 \
    --num-prompts "${WARMUP_NUM}" \
    --max-concurrency "${WARMUP_CONCURRENCY}" \
    --output-file "/dev/null" \
    --disable-tqdm \
    --seed 42
 
  echo ">>> 预热完成，等待 5 秒让引擎稳定..."
  sleep 5
}
# ===================== 核心测试函数 =====================
# 使用 generated-shared-prefix 数据集
# $1: 并发数  $2: num_prompts  $3: gsp_prompts_per_group
run_gsp_case() {
  # 每个 case 执行前清理服务端 KV Cache，确保第一个请求是冷启动
  # 注意：不清理客户端 pkl 缓存，复用已生成的测试数据以节省时间
  flush_server_cache
 
  local concurrency="$1"
  local num_prompts="$2"
  local prompts_per_group="$3"
 
  local case_name="gsp_68.7k_0.5k"
  local out_dir="${RESULT_BASE}/${case_name}"
  mkdir -p "${out_dir}"
 
  local log_file="${out_dir}/kimi_k2.5_gsp_concurrency_${concurrency}.log"
 
  echo ""
  echo "=== GSP Test: concurrency=${concurrency} num_prompts=${num_prompts} prompts_per_group=${prompts_per_group} ==="
  echo "    system_prompt_len=${GSP_SYSTEM_PROMPT_LEN} question_len=${GSP_QUESTION_LEN} output_len=${GSP_OUTPUT_LEN}"
 
  python3 -m sglang.bench_serving \
    --host "${HOST}" \
    --port "${PORT}" \
    --backend sglang \
    --model "${MODEL_PATH}" \
    --tokenizer  "${MODEL_PATH}" \
    --max-concurrency "${concurrency}" \
    --num-prompts "${num_prompts}" \
    --dataset-name generated-shared-prefix \
    --random-range-ratio "${RANDOM_RANGE_RATIO}" \
    --gsp-num-groups "${GSP_NUM_GROUPS}" \
    --gsp-prompts-per-group "${prompts_per_group}" \
    --gsp-system-prompt-len "${GSP_SYSTEM_PROMPT_LEN}" \
    --gsp-question-len "${GSP_QUESTION_LEN}" \
    --gsp-output-len "${GSP_OUTPUT_LEN}" \
    --output-file "${out_dir}/concurrency_${concurrency}.jsonl" \
    --seed 1 \
    |& tee "${log_file}"
 
}
 
# ===================== 执行流程 =====================
echo "============================================================"
echo " SGLang Benchmark - Kimi-K2.5 W4A8 (GSP 68.7k/0.5k)"
echo " 结果目录: ${RESULT_BASE}"
echo " 服务地址: http://${HOST}:${PORT}"
echo " 数据集: generated-shared-prefix"
echo "   system_prompt_len = ${GSP_SYSTEM_PROMPT_LEN}"
echo "   question_len      = ${GSP_QUESTION_LEN}"
echo "   output_len        = ${GSP_OUTPUT_LEN}"
echo "============================================================"
 
# 检查服务是否可用
echo ">>> 检查 sglang 服务是否就绪..."
HEALTH_RESP=$(curl -s --connect-timeout 5 "http://${HOST}:${PORT}/v1/models" 2>/dev/null || true)
if [ -z "${HEALTH_RESP}" ] || echo "${HEALTH_RESP}" | grep -q '"detail"'; then
  echo "ERROR: sglang 服务未就绪，请先启动服务！"
  echo "  地址: http://${HOST}:${PORT}"
  exit 1
fi
echo ">>> 服务就绪 ✓"
echo ">>> 模型信息: ${HEALTH_RESP}"
 
# 第一步：预热
do_warmup
# 测试case
run_gsp_case  1   64   16
run_gsp_case  2   64   16
run_gsp_case  3   64   16
run_gsp_case  4   64   16
run_gsp_case  8   64   16
run_gsp_case  16  64   16
run_gsp_case  32  64   16
 
# 输出汇总表格
echo ""
echo "============================================================"
echo " 全部测试完成！结果保存在: ${RESULT_BASE}"
echo "============================================================
```


 
性能数据参考

<image token="ObD9bU8nyoWAP5xgZJ3cOaPVn8f" width="1272" height="1198"/>

#  
 
#

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/并发|并发]]（metric）
- [[飞书知识图谱/实体/精度问题|精度问题]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/CUDA Graph|CUDA Graph]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/kimi-2.5|kimi-2.5]]（model）
- [[飞书知识图谱/实体/kimi-2.5-w4a16|kimi-2.5-w4a16]]（model）
- [[飞书知识图谱/实体/kimi-2.5-w4a8|kimi-2.5-w4a8]]（model）
- [[飞书知识图谱/实体/kimi-2.5-w8a8|kimi-2.5-w8a8]]（model）
- [[飞书知识图谱/实体/Kimi-K2.5|Kimi-K2.5]]（model）
- [[飞书知识图谱/实体/Kimi-K2.5_w4a8|Kimi-K2.5_w4a8]]（model）
- [[飞书知识图谱/实体/Kimi-K2.5_w4a8_gsp_|Kimi-K2.5_w4a8_gsp_]]（model）
- [[飞书知识图谱/实体/Kimi-K2.5_w4a8.tar.gz|Kimi-K2.5_w4a8.tar.gz]]（model）
- [[飞书知识图谱/实体/Kimi-K2.5_w8a8|Kimi-K2.5_w8a8]]（model）
- [[飞书知识图谱/实体/kimi-k2.5-w8a8.tar.gz|kimi-k2.5-w8a8.tar.gz]]（model）
- [[飞书知识图谱/实体/Kimi-K25-eagle3|Kimi-K25-eagle3]]（model）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/Prefix Cache|Prefix Cache]]（concept）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）
- [[飞书知识图谱/实体/RDMA|RDMA]]（concept）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TBO|TBO]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

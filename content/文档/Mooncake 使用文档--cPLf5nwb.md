---
title: Mooncake 使用文档
tags:
  - AI
  - LLM
  - Benchmark
model:
  - GLM5-test
  - Qwen3-32B
  - Qwen3-8B
framework:
  - Mooncake
  - SGLang
  - vLLM
issue_type:
  - 超时
  - OOM
source: feishu
feishu_token: VAKFw0aLbig82EkRieHcPLf5nwb
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/VAKFw0aLbig82EkRieHcPLf5nwb
last_synced_at: 2026-06-07T17:07:53.393Z
---

# Mooncake 使用文档

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/VAKFw0aLbig82EkRieHcPLf5nwb)

# 关于 Mooncake

Mooncake 是一个面向大语言模型（LLM）推理场景的 KV Cache 传输与缓存卸载框架，核心目标是解决 PD 分离架构下 KV Cache 在 Prefill 和 Decode 节点间的高效传输问题，以及将 KV Cache 从 GPU 显存卸载到主机内存或 SSD 以降低推理成本。Mooncake 的核心能力包括：

- **多协议传输**：支持 RDMA、TCP 等多种传输协议，在 DCU 环境下还可启用 HIP IPC（节点内）和 HIP RPC（跨节点）传输，灵活适配不同网络硬件
- **多级缓存池**：在 GPU 显存（L1）、主机 DRAM（L2）、SSD（L3）上构建分层缓存，平衡推理速度与容量
- **即插即用**：已深度集成至 SGLang 和 vLLM，通过 `--kv-transfer-config` 或 `--disaggregation-mode` 即可启用 PD 分离
- **缓存后端**：可作为 LMCache 的远端存储后端（Mooncake Store），支持内存卸载与 SSD 卸载

官方文档：[Welcome to Mooncake — Mooncake](https%3A%2F%2Fkvcache-ai.github.io%2FMooncake%2F)

# 下载和安装

## mooncake 安装

- dcu mooncake 代码仓库：[http://42.228.13.241:10068/dcutoolkit/deeplearing/mooncake](http%3A%2F%2F42.228.13.241%3A10068%2Fdcutoolkit%2Fdeeplearing%2Fmooncake) （develop 分支，基于官方 0.3.10.post1）
- dcu mooncake whl 包：
	- `mooncake_transfer_engine`：正常版本，普通网卡，不带 hylink 支持 [http://pypi.sourcefind.cn:666/das_nightly/dtk2604-rc4-mooncake/+f/79c/add379d74452d/mooncake_transfer_engine-0.3.10.post1+das.opt1.dtk2604.2605131137.gd34f6f-cp310-cp310-manylinux_2_35_x86_64.whl](http%3A%2F%2Fpypi.sourcefind.cn%3A666%2Fdas_nightly%2Fdtk2604-rc4-mooncake%2F%2Bf%2F79c%2Fadd379d74452d%2Fmooncake_transfer_engine-0.3.10.post1%2Bdas.opt1.dtk2604.2605131137.gd34f6f-cp310-cp310-manylinux_2_35_x86_64.whl) 
	- `mooncake_transfer_engine_rpc`：带 hylink 支持的版本，普通网卡[http://pypi.sourcefind.cn:666/das_nightly/dtk2604-rc4-mooncake/+f/4aa/b1d2c2d1653e9/mooncake_transfer_engine_rpc-0.3.10.post1+das.opt1.dtk2604.2605131408.gd34f6f-cp310-cp310-manylinux_2_35_x86_64.whl](http%3A%2F%2Fpypi.sourcefind.cn%3A666%2Fdas_nightly%2Fdtk2604-rc4-mooncake%2F%2Bf%2F4aa%2Fb1d2c2d1653e9%2Fmooncake_transfer_engine_rpc-0.3.10.post1%2Bdas.opt1.dtk2604.2605131408.gd34f6f-cp310-cp310-manylinux_2_35_x86_64.whl)
	- `mooncake_transfer_engine_shca`：支持天龙网卡的版本，不带 hylink 支持 [http://pypi.sourcefind.cn:666/das_nightly/dtk2604-rc4-mooncake/+f/2e2/14988dbb22475/mooncake_transfer_engine_shca-0.3.10.post1+das.opt1.dtk2604.2605131044.gd34f6f-cp310-cp310-manylinux_2_35_x86_64.whl](http%3A%2F%2Fpypi.sourcefind.cn%3A666%2Fdas_nightly%2Fdtk2604-rc4-mooncake%2F%2Bf%2F2e2%2F14988dbb22475%2Fmooncake_transfer_engine_shca-0.3.10.post1%2Bdas.opt1.dtk2604.2605131044.gd34f6f-cp310-cp310-manylinux_2_35_x86_64.whl)
- 安装 mooncake：`pip install mooncake_transfer_engine*.whl`
- <text bgcolor="light-yellow">安装前请先卸载旧版本</text>：`pip uninstall mooncake_transfer_engine`

## 天龙网卡驱动安装

在天龙网卡环境中需要先确保安装天龙网卡对应驱动。
```bash {wrap}
# 查看驱动是否正常
# 检测环境中是否有适配dcu网卡的工具
# 检查 shca 驱动 & shca-tools 是否正常加载
dpkg -l | grep shca
ibv_devinfo

#下载linux快传客户端，若有无需重复下载
wget https://zzefile.scnet.cn:65011/efile/share/fasttrans-client.tar.gz

#下载执行脚本
rayfile-c -a zzefile.scnet.cn -P 65012 -u jsyadmin -w 9380ec0f4b92b4ccb1-f638-44e3-b420-3c52876c6bc0 -no-meta -symbolic-links follow -retry 10 -retrytimeout 30 -o download -s '/L4/yongdan/project/GLM5-test/mlxtoshca.sh' -d <请输入下载目标全路径并替换尖括号及本内容>

#下载驱动安装包
rayfile-c -a zzefile.scnet.cn -P 65012 -u jsyadmin -w 9380ec0f4b0e868171-0d49-4ff3-9fa3-5618bb17dd91 -no-meta -symbolic-links follow -retry 10 -retrytimeout 30 -o download -s '/L4/yongdan/shca-tools_2.500.4.B059-Ubuntu22.04_amd64.deb' -d <请输入下载目标全路径并替换尖括号及本内容>

bash mlxtoshca.sh

# 验证安装
ibv_devinfo 
```

使用时请关注mooncake初始化log，以确认是否正确发现网卡，以及实际使用的Transport是否为RDMA。

执行程序指定网卡时天龙网卡通常为`shca_x`，可通过`ibv_devices`查看网卡名称。

其它组件(nccl/deepep)等可能也需要对应的天龙网卡版本以正确启用相应的RDMA支持。

<image token="Df7nbG9stoKKOrxO9HCcOhuynXf" width="2887" height="966" align="center"/>

# 使用方法

## 环境变量

### LOG

- `MC_LOG_LEVEL`（该选项可以设置成`TRACE`/`INFO`/`WARNING`/`ERROR`，默认`INFO`）
```bash
export MC_LOG_LEVEL=TRACE
```

### RDMA 环境变量

- 握手失败时，可以通过设置 host ip 解决。
```bash
# sglang 握手失败时可以通过设置 host ip 解决
export SGLANG_HOST_IP=${HOST_IP}
# vllm 握手失败时可以通过设置 host ip 解决
export VLLM_HOST_IP=${HOST_IP}
```

- 存在跨 sm ib nic transfer 问题时，可以尝试启用设备亲和性。
```bash
# 启用后，Transfer Engine 将优先选择和本地网卡同名的远端网卡进行通信。
# 默认值为 false。存在跨sm ib nic transfer 问题时，可尝试设置。
# 双平面需要设置
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
```

- 同一交换机内通信，可以尝试切换 GID。
```bash
# 默认为3（global 路由），让 RDMA 通信走带全局路由头 GRH 的 GID 地址，交换机识别后按 GID 跨三层转发
# 0（link-local），只能在同一交换机/本地子网进行通信，不支持跨交换机、跨子网通信
export MC_IB_GID_INDEX=0
```

- 只有某些网卡可见时，可以设置。
```bash
export MC_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
```

- 显式启用`MC_IB_PCI_RELAXED_ORDERING`时，可能会导致`Failed to register memory`，此时不支持该特性。

### HIP IPC/RPC 环境变量

- 启用 hip ipc transport (仅节点内)
```bash
# 使用 HIP IPC Transport 作为主要传输方式
export MC_FOCRE_MNNVL=1
```

- 启用hip rpc transport (节点内和节点间，需要安装`mooncake_transfer_engine_rpc`)
```bash
# 使用 HIP RPC Transport 作为主要传输方式
export MC_FOCRE_MNNVL=1
export MC_USE_HIP_IPC=0

# 修改 copy kernel cu number
export MC_HIP_COPY_BLOCKS=xxx
```

###  Topology 相关

查看自动发现 Topology，自动发现会将和 CPU 或 GPU 同 NUMA 的 NIC 放 preferred_hca，其余放 avail_hca；对于 GPU，若没有同 NUMA 的 NIC，则会将距离最近（两个 PCI 设备在 Linux PCI 树上的拓扑距离估计，并非NUMA distance）的 NIC 放 preferred_hca。

有关 topo 的信息请参阅`显存与计算拓扑`和`主机内存与拓扑`部分。
```bash {wrap}
transfer_engine_topology_dump
```

当多个 GPU 都优先使用同一张 NIC 时，可能导致该网卡负载过大，或者需要限定使用的 NIC 时，可以手动设定 Topo。

自定义拓扑JSON文件路径 `MC_CUSTOM_TOPO_JSON`

JSON 格式是：
```bash {wrap}
 {
   "<storage_type>": [
     ["preferred_hca..."],
     ["avail_hca..."]
   ]
 }
```

也就是每个 key 的 value 都是长度为 2 的数组：

1. 第一项：优先使用的 RDMA 网卡列表  
2. 第二项：可回退使用的网卡列表

示例：
```json {wrap}
 {
   "cpu:0":  [["mlx5_2"], []],
   "cpu:1":  [["mlx5_3"], []],
   "cpu:2":  [["mlx5_4"], []],
   "cpu:3":  [["mlx5_5"], []],
   "cpu:4":  [["mlx5_6"], []],
   "cpu:5":  [["mlx5_7"], []],
   "cpu:6":  [["mlx5_8"], []],
   "cpu:7":  [["mlx5_9"], []],
   "hip:0":  [["mlx5_2"], []],
   "hip:1":  [["mlx5_3"], []],
   "hip:2":  [["mlx5_4"], []],
   "hip:3":  [["mlx5_5"], []],
   "hip:4":  [["mlx5_6"], []],
   "hip:5":  [["mlx5_7"], []],
   "hip:6":  [["mlx5_8"], []],
   "hip:7":  [["mlx5_9"], []]
 }
```

## Mooncake 测试

### Transfer engine bench

transfer_engine_bench 路径：“pip show mooncake-transfer-engine” ，transfer_engine_bench 在安装路径下。
```bash
# local_host_ip: node 1 ip
transfer_engine_bench --mode=target --auto_discovery --protocol=rdma \
    --metadata_server=P2PHANDSHAKE --gpu_id=-1 \
    --local_server_name=${local_host_ip}
```

```bash
# local_host_ip: node 2 ip
# remote_host_ip: node 1 ip
# port: 从 node1 log 里找到 ZZZ，Transfer Engine RPC using XXX, listening on YYY:ZZZ）
transfer_engine_bench --mode=initiator --auto_discovery --protocol=rdma \
    --metadata_server=P2PHANDSHAKE --gpu_id=-1 \
    --local_server_name=${local_host_ip} --segment_id=${remote_host_ip}:${port}
```

测试最大 throughput 推荐参数：
```bash
--buffer_size=4294967296 --threads=12 --batch_size=128 --block_size=2097152
MC_SLICE_SIZE 可以调整为 131072，甚至可以更大。
```

NMZ 测试结果如下：

<image token="ZlKHbStlCorqYrxShmxcW8ucnbe" width="951" height="324" align="center"/>

block size 小于128kb 时对 throughput 结果影响较大。

<image token="MPfobvIHUoB5dqxdbLoc5G8Sn2c" width="1560" height="980" align="center"/>

## SGLang PD 分离

### 单节点测试
```bash
python -m sglang.launch_server \
    --model-path=/model/Qwen3-8B \
    --disaggregation-mode prefill \
    --port 30000 \
    --attention-backend=fa3 \
    --page-size=64
```

```bash
# --base-gpu-id` 的功能类似于环境变量 `CUDA_VISIBLE_DEVICES`，用于避免重用第 0 个 GPU 卡。不同之处在于它用于指定 GPU 的起始编号。
# 如果设置为 2，将跳过第一和第二张卡，直接使用第三张卡。
python -m sglang.launch_server \
    --model-path=/model/Qwen3-8B \
    --disaggregation-mode decode \
    --port 30001 \
    --base-gpu-id 1 \
    --attention-backend=fa3 \
    --page-size=64
```

```bash
python -m sglang_router.launch_router \
    --pd-disaggregation \
    --prefill http://127.0.0.1:30000 \
    --decode http://127.0.0.1:30001 \
    --host 0.0.0.0 --port 8000
```

```json
curl http://127.0.0.1:8000/v1/chat/completions   -H "Content-Type: application/json"   -d '{
    "model": "default",
    "messages": [
      {"role": "user", "content": "你好，介绍一下你自己"}
    ],
    "max_tokens": 500
  }'
```

### 双节点测试
```python
python -m sglang.launch_server \
    --model-path=/model/Qwen3-8B \
    --disaggregation-mode prefill \
    --attention-backend=fa3 \
    --page-size=64 \
    --host 10.16.1.58  \
    --port 30002 \
    --dist-init-addr 10.16.1.58:5000 \
    --nnodes 1 \
    --node-rank 0 \
    --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9 
```

```python
python -m sglang.launch_server \
    --model-path=/model/Qwen3-8B \
    --disaggregation-mode decode \
    --base-gpu-id 1 \
    --attention-backend=fa3 \
    --page-size=64 \
    --host 10.16.1.60  \
    --port 30002 \
    --dist-init-addr 10.16.1.58:5000 \
    --nnodes 1 \
    --node-rank 0 \
    --disaggregation-ib-device mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9 
```

```python
python -m sglang_router.launch_router \
    --pd-disaggregation \
    --prefill http://10.16.1.58:30002 \
    --decode http://10.16.1.60:30002 \
    --host 0.0.0.0 --port 8000
```

```python
curl http://127.0.0.1:8000/v1/chat/completions   -H "Content-Type: application/json"   -d '{
    "model": "default",
    "messages": [
      {"role": "user", "content": "中国的首都是哪里"}
    ],
    "max_tokens": 500
  }'
```

## vLLM PD 分离

### 单节点测试
```bash
vllm serve Qwen3/Qwen3-8B \  
    --port 8010 \  
    --kv-transfer-config '{"kv_connector":"MooncakeConnector","kv_role":"kv_producer"}' 
```

```bash
HIP_VISIBLE_DEVICES=1 vllm serve Qwen3/Qwen3-8B \  
    --port 8020 \  
    --kv-transfer-config '{"kv_connector":"MooncakeConnector","kv_role":"kv_consumer"}'
```

```bash
# mooncake_connector_proxy.py 在 vllm 源码中：
# https://developer.sourcefind.cn/codes/OpenDAS/vllm/-/tree/v0.18.1/examples/online_serving/disaggregated_serving/mooncake_connector
python3 vllm/examples/online_serving/disaggregated_serving/mooncake_connector/mooncake_connector_proxy.py \  
    --prefill "http://0.0.0.0:8010" "8998" \  
    --decode "http://0.0.0.0:8020" \  
    --port 8000
```

```json
curl http://localhost:8000/v1/completions \  
    -H "Content-Type: application/json" \  
    -d '{    
        "model": "Qwen/Qwen3-8B",    
        "messages": [      
            {"role": "user", "content": "Tell me a long story about artificial intelligence."}    
        ]  
    }'
```

### 双节点测试（P：DP2TP4  D：DP2TP4）
```python
vllm serve Qwen3/Qwen3-8B \  
    --port 8010 \  
    --data-parallel-size 2 --tensor-parallel-size 4 \  
    --kv-transfer-config '{"kv_connector":"MooncakeConnector","kv_role":"kv_producer"}' 
```

```python
vllm serve Qwen3/Qwen3-8B \  
    --port 8020 \  
    --data-parallel-size 2 --tensor-parallel-size 4 \  
    --kv-transfer-config '{"kv_connector":"MooncakeConnector","kv_role":"kv_consumer"}'
```

```python
# mooncake_connector_proxy.py 在 vllm 源码中：
# https://developer.sourcefind.cn/codes/OpenDAS/vllm/-/tree/v0.18.1/examples/online_serving/disaggregated_serving/mooncake_connector
python3 vllm/examples/online_serving/disaggregated_serving/mooncake_connector/mooncake_connector_proxy.py \  
    --prefill "http://10.63.60.113:8010" "8998" \  
    --decode "http://10.63.60.114:8020" \  
    --port 8000
```

```python
curl http://localhost:8000/v1/completions \  
    -H "Content-Type: application/json" \  
    -d '{    
        "model": "Qwen/Qwen3-8B",    
        "messages": [      
            {"role": "user", "content": "Tell me a long story about artificial intelligence."}    
        ]  
    }'
```

## SGLang Hicache + Mooncake Store

在启动服务前，必须清晰掌握节点内的计算、网络与内存布局。

### 显存与计算拓扑

- **查看显存容量及状态**：
```plaintext
hy-smi --showmeminfo vram
```

- **查看 DCU 间拓扑连接**：
```plaintext
hy-smi --showtopo
```

### 主机内存与拓扑

- **查看系统内存与共享内存现状**：
```plaintext
free -h
df -h /dev/shm
cat /proc/meminfo
```

- **查看物理硬件层级拓扑**：
```plaintext
lstopo
```

- **查看 NUMA 节点状态**：
```plaintext
numactl -H
numastat -m
```

Linux 内核会尽可能利用空闲内存作为缓存使用，以提升系统整体 I/O 性能。因此，在系统长期运行、大量文件读写、RDMA、共享内存（shmem/memfd）、大模型推理等场景下，经常会出现：`free -h`显示：`free` 很小，`buff/cache` 很大，但实际上并不一定意味着系统真正缺少可用内存。

不过，在以下场景中，缓存可能会导致实际内存分配失败：

- NUMA 本地节点内存耗尽
- RDMA pinned memory 分配失败
- mmap(MAP_POPULATE) 失败
- GPU DMA 注册失败
- 大块连续内存申请失败
- Mooncake / tmpfs / memfd 长时间占用缓存页
- OOM Killer 被触发

此时可以手动释放 Linux 页缓存。
```bash {wrap}
sync
echo 3 > /proc/sys/vm/drop_caches
```

执行后会导致短期<text bgcolor="light-yellow"> I/O 性能下降</text>，因为文件缓存被清空，后续文件读取需要重新从磁盘加载。

建议在以下情况使用：启动新的超大型任务前，出现 OOM 或内存申请失败时，NUMA 节点 free memory 极低时。OOM信息可能需要在`dmesg`中查看。

不建议频繁执行。

### Mooncake Master Server 配置

Mooncake 作为 L3 Cache 后端，依赖元数据服务器进行节点发现与 Cache 寻址，负责协调整个集群的逻辑存储空间池，管理L3 KV 缓存空间分配与淘汰。

- **启动命令**：
```plaintext {wrap}
mooncake_master --enable_http_metadata_server=true \--http_metadata_server_host=<your_node_ip> \--http_metadata_server_port=8080
```

当 Mooncake 使用 `MooncakeHostMemAllocator` 分配共享内存（shm）时，为了追求极致带宽，它默认会在**网卡（NIC）关联的 NUMA 节点**上优先分配。

- **现象描述**：

如果系统拥有 8 个 NUMA 节点（Node 0-7），但只插了 4 张网卡（分别挂在 Node 0, 1, 4, 5 下），Mooncake 会试图将所有的 Cache 缓冲区压入这 4 个节点。即使总内存充足，这 4 个节点也会因为空间耗尽触发 **局部 OOM**。

- **快速查看网卡绑定的 NUMA 节点**：
```plaintext
# 以 shca_0 为例
cat /sys/class/infiniband/shca_0/device/numa_node
```

### 跨节点内存交错（Interleave）

当发现 `numastat -m` 中节点负载极度不均（如 Node 0 剩余 1G，Node 2 剩余 50G）时，必须打破局部性限制，允许内存分配散布到全节点。跨节点访问可能会略有性能下降。

- **启动方式**：

在 SGLang 启动命令前添加 `numactl --interleave=all`。
```plaintext
numactl --interleave=all python3 -m sglang.launch_server <args...>
```

### 动态扩容

无需重启系统即可即时调整shmem容量限制：
```plaintext
# 假设将上限调整至 400G
sudo mount -o remount,size=400G /dev/shm
```

- **MAP_POPULATE 影响**：Mooncake 开启了该标志，申请时会立即“占满”物理内存以换取传输时的零中断性能。建议保留 15%-20% 的内存冗余给操作系统内核。

---

**HiCache CPU 内存使用：**

使用 HiCache 时，默认情况下，L2 主机 DRAM（CPU 内存）中用于 KV 缓存的大小是 L1 设备内存（GPU 内存）中 KV 缓存大小的 **2 倍**。

如果模型较小但 GPU 内存很大，特别是在多 TP（tensor parallel）场景下，这可能导致 L1 KV 缓存变得很大，继而消耗过多 CPU DRAM。

在这种情况下，应根据你的硬件手动配置合适的 L2 缓存大小。

可以通过设置 `--hicache-ratio` 或 `--hicache-size` 实现。

## LMCache + Mooncake Store

LMCache 官网：https://docs.lmcache.ai/kv_cache/storage_backends/mooncake.html

### 创建本地盘目录
```bash
mkfs.ext4 /dev/nvme0n1
mount /dev/nvme0n1 /mnt/mooncake   # 这个路径需要透传到docker内
```

### 启动 mooncake
```plaintext {wrap}
nohup mooncake_master --enable_http_metadata_server=1  --rpc_port=50051  --enable-offload true > mooncake.log 2>&1 &
```

支持三种后端模式：

- bucket_storage_backend：几个对象文件一起写在一个文件桶里
- file_per_key_storage_backend：每个对象一个文件
- offset_allocator_storage_backend：所有对象写在一个大文件里，根据偏移去找对象，不支持进程重启后cache文件复用
```sql {wrap}
export MOONCAKE_OFFLOAD_FILE_STORAGE_PATH=/llm/mooncake
export MOONCAKE_OFFLOAD_STORAGE_BACKEND_DESCRIPTOR=bucket_storage_backend

mooncake_client \
    --master_server_address=127.0.0.1:50051 \
    --host=10.17.95.253 \
    --protocol="tcp"  \
    --device_names=enp97s0f0 \
    --port=50052 \
    --global_segment_size="5368709120" \
    --enable_offload=true  \
    --metadata_server="P2PHANDSHAKE"
```

### 编写 lmcache 配置文件
```yaml
# LMCache Configuration
local_cpu: False
remote_url: "mooncakestore://127.0.0.1:50051/"
max_local_cpu_size: 10  # small local buffer
numa_mode: "auto"      # reduce tail latency with multi-NUMA/multi-NIC
pre_caching_hash_algorithm: sha256_cbor_64bit

# Mooncake Configuration (via extra_config)
extra_config:
  use_exists_sync: true
  save_chunk_meta: False  # Enable chunk metadata optimization
  local_hostname: "10.17.95.253"
  metadata_server: "P2PHANDSHAKE"
  protocol: "tcp"
  device_name: ""        # leave empty; autodetect device(s)
  global_segment_size: 5368709120    # 20 GiB per worker
  master_server_address: "127.0.0.1:50051"
  local_buffer_size: 0    # rely on LMCache local_cpu as the buffer
  mooncake_prefer_local_alloc: true  # prefer local segment if available
```

### 启动 vllm
```bash {wrap}
export VLLM_LOGGING_LEVEL=INFO
export LMCACHE_LOG_LEVEL=INFO
export PYTHONHASHSEED=0
export LMCACHE_CONFIG_FILE=lmcache_config.yaml

nohup vllm serve /llm/models/Qwen3-32B \
    --max-log-len 64 \
    --tensor-parallel-size 2 \
    --port 8000 \
    --gpu-memory-utilization 0.85 \
    --kv-transfer-config '{"kv_connector":"LMCacheConnectorV1","kv_role":"kv_both"}' \
    --trust-remote-code \
    --disable-custom-all-reduce \
    --disable-log-requests \
    --pipeline-parallel-size 1 \
    --served-model-name "Qwen3-32B" \
> 20260414_175623_vllm_running.log 2>&1 &

```

执行推理后，查看挂载路径下是否有缓存文件保存

<image token="PnOTbAfIkoEYthxTdDGcL0Xdnhd" width="875" height="202" align="center"/>

# TransferEngine API

## 使用流程

- 创建引擎
```python
from mooncake.engine import TransferEngine

engine = TransferEngine()
```

- 初始化
```python
engine.initialize(
    "localhost",       # 本机地址（必须唯一）
    "P2PHANDSHAKE",    # metadata server
    "rdma",            # 协议：rdma / tcp
    ""                 # device（可选）
)
```

使用基本配置初始化传输引擎。

参数：

- `local_hostname`: 本地服务器的主机名和端口（例如 "127.0.0.1:12345"）
- `metadata_server`: 元数据服务器的连接字符串（例如 "127.0.0.1:2379" 或 "etcd://127.0.0.1:2379"）
- `protocol`: 使用的传输协议（例如 "rdma"、"tcp" 等）
- `device_name`:用于筛选设备的名称列表（用逗号分隔）；如果为空字符串，则表示使用所有设备
- 注册内存
```python
import numpy as np

buf = np.ones(1024, dtype=np.uint8)
addr = buf.ctypes.data
size = buf.nbytes

engine.register_memory(addr, size)
```

注册内存区域以供  Transfer Engine 访问，未注册的内存无法进行传输

- 数据传输（同步写）
```python
ret = engine.transfer_sync_write(
    target_hostname,
    local_buffer_addr,
    remote_buffer_addr,
    length
)
```

执行同步写操作，将数据从本地缓冲区传输到远程缓冲区。

参数：

- `target_hostname` (str): 目标服务器的主机名
- `buffer` (int): 本地缓冲区地址
- `peer_buffer_address` (int): 远程缓冲区地址
- `length` (int): 要传输的字节数
- 注销内存
```python
ret = engine.unregister_memory(ptr)
```

注销之前注册的内存区域，使其不再被 Transfer Engine 访问。

## Sample Demo
```python
import zmq
import torch
from mooncake.engine import TransferEngine
import os
# from mooncake.allocator import NVLinkAllocator

# use cross-node hip rpc transport, need hardware support
# os.environ["MC_FORCE_MNNVL"] = "1" # or os.environ["MC_FORCE_HIP"] = "1"
# os.environ["MC_USE_HIP_IPC"] = "0"

def main():
    torch.cuda.set_device(0) # Use GPU 0 for server
    # allocator = NVLinkAllocator.get_allocator(0)
    # custom_mem_pool = torch.cuda.MemPool(allocator.allocator())
    # Initialize ZMQ context and socket
    context = zmq.Context()
    send_socket = context.socket(zmq.PUSH)
    send_socket.bind("tcp://*:5555")  # Bind to port 5555 for buffer info

    recv_socket = context.socket(zmq.PULL)
    recv_socket.bind("tcp://*:5556")  # Bind to port 5556

    HOSTNAME = "localhost" # localhost for simple demo, replace with actual IP if needed
    METADATA_SERVER = "P2PHANDSHAKE" # [ETCD_SERVER_URL, P2PHANDSHAKE, ...]
    PROTOCOL = "rdma" # [rdma, tcp, ...]
    DEVICE_NAME = "" # auto discovery if empty
    
    # Initialize server engine
    server_engine = TransferEngine()
    ret_value = server_engine.initialize(
        HOSTNAME,
        METADATA_SERVER,
        PROTOCOL,
        DEVICE_NAME
    )
    assert ret_value == 0, "Failed to initialize Mooncake server engine."
    session_id = f"{HOSTNAME}:{server_engine.get_rpc_port()}"
    
    # with torch.cuda.use_mem_pool(custom_mem_pool):
    # Allocate memory on server side (1MB buffer)
    server_buffer = torch.full((1024 * 1024,), 77, dtype=torch.uint8, device="cuda:0")
    server_ptr = server_buffer.data_ptr()
    server_len = server_buffer.nbytes 
    
    torch.cuda.synchronize(0)  # Ensure memory is allocated and initialized before registration
    # Register memory with Mooncake
    ret_value = server_engine.register_memory(server_ptr, server_len)
    if ret_value != 0:
        print("Mooncake memory registration failed.")
        raise RuntimeError("Mooncake memory registration failed.")

    print(f"Server initialized with session ID: {session_id}")
    print(f"Server buffer address: {server_ptr}, length: {server_len}")
    
    # Send buffer info to client
    buffer_info = {
        "session_id": session_id,
        "ptr": server_ptr,
        "len": server_len
    }
    send_socket.send_json(buffer_info)
    print("Buffer information sent to client")
    
    # Keep server running
    transfer_status = recv_socket.recv_json()
    if transfer_status.get("status") == "transfer_complete":
        print("Data transfer from client to server completed.")

    expect_val = 92
    is_correct = torch.all(server_buffer == expect_val).item()
    if is_correct:
        print("Data verification successful! All values are correct.")
    else:
        print("Data verification failed! Buffer values do not match expected value.")
    # Cleanup
    ret_value = server_engine.unregister_memory(server_ptr)
    if ret_value != 0:
        print("Mooncake memory deregistration failed.")
        raise RuntimeError("Mooncake memory deregistration failed.")

    send_socket.close()
    recv_socket.close()
    context.term()

if __name__ == "__main__":
    main() 
 
```

```python
import torch
import zmq
from mooncake.engine import TransferEngine
import os
# from mooncake.allocator import NVLinkAllocator

# use cross-node hip rpc transport, need hardware support
# os.environ["MC_FORCE_MNNVL"] = "1" # or os.environ["MC_FORCE_HIP"] = "1"
# os.environ["MC_USE_HIP_IPC"] = "0"

def main():
    torch.cuda.set_device(1) # Use GPU 1 for client
    # allocator = NVLinkAllocator.get_allocator(1)
    # custom_mem_pool = torch.cuda.MemPool(allocator.allocator())
    # Initialize ZMQ context and socket
    context = zmq.Context()
    recv_socket = context.socket(zmq.PULL)
    recv_socket.connect(f"tcp://localhost:5555")

    send_socket = context.socket(zmq.PUSH)
    send_socket.connect(f"tcp://localhost:5556")
    
    # Wait for buffer info from server
    print("Waiting for server buffer information...")
    buffer_info = recv_socket.recv_json()
    server_session_id = buffer_info["session_id"]
    server_ptr = buffer_info["ptr"]
    server_len = buffer_info["len"]
    print(f"Received server info - Session ID: {server_session_id}")
    print(f"Server buffer address: {server_ptr}, length: {server_len}")
    
    # Initialize client engine
    HOSTNAME = "localhost" # localhost for simple demo, replace with actual IP if needed
    METADATA_SERVER = "P2PHANDSHAKE" # [ETCD_SERVER_URL, P2PHANDSHAKE, ...]
    PROTOCOL = "rdma" # [rdma, tcp, ...]
    DEVICE_NAME = "" # auto discovery if empty

    client_engine = TransferEngine()
    client_engine.initialize(
        HOSTNAME,
        METADATA_SERVER,
        PROTOCOL,
        DEVICE_NAME
    )
    session_id = f"{HOSTNAME}:{client_engine.get_rpc_port()}"

    # with torch.cuda.use_mem_pool(custom_mem_pool):
    # Allocate and initialize client buffer (1MB)
    client_buffer = torch.full((1024 * 1024,), 92, dtype=torch.uint8, device="cuda:1")
    client_ptr = client_buffer.data_ptr()
    client_len = client_buffer.nbytes 

    torch.cuda.synchronize(1) # Ensure data is ready before transfer

    # Register memory with Mooncake
    ret_value = client_engine.register_memory(client_ptr, client_len)
    if ret_value != 0:
        print("Mooncake memory registration failed.")
        raise RuntimeError("Mooncake memory registration failed.")

    print(f"Client initialized with session ID: {session_id}")

    # Transfer data from client to server
    print("Transferring data to server...")
    ret = client_engine.transfer_sync_write(
        server_session_id,
        client_ptr,
        server_ptr,
        min(client_len, server_len)  # Transfer minimum of both lengths
    )

    if ret >= 0:
        print("Transfer successful!")
    else:
        print("Transfer failed!")

    # notify server after transfer is complete
    send_socket.send_json({"status": "transfer_complete"})
    # Cleanup
    ret_value = client_engine.unregister_memory(client_ptr)
    if ret_value != 0:
        print("Mooncake memory deregistration failed.")
        raise RuntimeError("Mooncake memory deregistration failed.")

    send_socket.close()
    recv_socket.close()
    context.term()

if __name__ == "__main__":
    main() 
```

---

# Mooncake Store

Mooncake Store 是面向 LLM 推理场景的分布式 KV cache 存储引擎。它提供类似 key-value store 的对象接口，例如 `put`、`get`、`remove`，但它的目标不是通用缓存，而是高吞吐、低开销地存储和传输 KV cache、tensor buffer、序列化块等大对象。

核心特点：

- Master 只负责元数据、空间分配、副本位置和对象状态，不承载数据面传输。
- Client 既可以作为请求发起方，也可以贡献本机内存作为分布式存储池的一部分。
- 数据传输由 Transfer Engine 完成。
- 支持普通`put/get` API，也支持 `put_from` / `get_into` 等 zero-copy API。
- 支持 batch 操作、对象副本、软固定/硬固定、内存 eviction 和 SSD offload。

## 组件和角色

Mooncake Store 主要包含两个角色：

- `mooncake_master`：集群控制面。负责对象元数据、内存 segment 管理、分配策略、eviction、任务调度等。
- `MooncakeDistributedStore` / `mooncake_client`：应用侧使用的 Store client。它可以发起 `put/get` 等请求，也可以贡献本机内存给集群。

一个 Store client 的两个关键容量参数是：

- `global_segment_size`：贡献给集群的存储空间。大于 0 时，该 client 可能被 Master 选为对象副本所在节点。
- `local_buffer_size`：发起普通 `put/get` 时使用的本地临时传输 buffer，普通 API 会使用它；zero-copy API 使用用户注册的 buffer。

## 启动与部署

常见模式

- **Embedded Real Client**：应用进程内直接嵌入 Real Client，既发请求，也可贡献内存。
- **Standalone Real Client**：独立 `mooncake_client `管理内存、RPC、SSD offload；应用进程通过 DummyClient 连接它，或者将`global_segment_size`设置为0，只作请求发起，不贡献空间。

**启动 Master**
```bash
mooncake_master \
  --enable_http_metadata_server=true \
  --http_metadata_server_host=0.0.0.0 \
  --http_metadata_server_port=8080 \
  --rpc_port=50051
```

**启动 Standalone real client**
```bash {wrap}
mooncake_client \
  --master_server_address=127.0.0.1:50051 \
  --host=<machine IP> \
  --protocol=rdma \
  --device_names=<NIC> \
  --port=50052 \
  --global_segment_size="4 GB" \
  --metadata_server=http://192.168.140.73:8085/metadata
```

**启动** **Embedded Real Client**
```python {wrap}
from mooncake.store import MooncakeDistributedStore

store = MooncakeDistributedStore()
store.setup(
    local_hostname="<machine IP>",
    metadata_server="http://127.0.0.1:8080/metadata",
    global_segment_size=4 * 1024 * 1024 * 1024,  # 4 GB
    local_buffer_size=512 * 1024 * 1024,  # 512 MB
    protocol="rdma",
    rdma_devices="",
    master_server_addr="127.0.0.1:50051"
)
```

将`global_segment_size`设置为 0 时，只作为请求发起方。

## 常用对象操作

**put / get**

`put` 写入一个不可变对象。如果 key 已存在，当前语义不会覆盖已有对象。
```python
store.put("key1", b"value1")
data = store.get("key1")
```

**batch_put / batch_get**

batch 接口可以减少 Master RPC 次数，适合一次处理多个对象：
```python
keys = ["k1", "k2", "k3"]
values = [b"v1", b"v2", b"v3"]

store.put_batch(keys, values)
results = store.get_batch(keys)
```

batch 结果按 key 独立成功或失败。调用方应检查每个返回值，而不是假设整批完全成功。

**is_exist / remove**
```python
if store.is_exist("k1"):
    store.remove("k1")
```

对于批量清理，可使用 `batch_remove`、`remove_by_regex` 或 `remove_all`。

## Zero-copy 用法

zero-copy API 指 `register_buffer` 后直接读写：

- `put_from(key, buffer_ptr, size)`
- `get_into(key, buffer_ptr, size)`
- `batch_put_from(keys, buffer_ptrs, sizes)`
- `batch_get_into(keys, buffer_ptrs, sizes)`
- `batch_put_from_multi_buffers(keys, all_buffer_ptrs, all_sizes)`
- `batch_get_into_multi_buffers(keys, all_buffer_ptrs, all_sizes)`

普通 `put/get` 会使用 Mooncake 内部 local buffer。对于大 tensor 或高频 KV cache 传输，可以使用 zero-copy API，直接从用户 buffer 写入或直接读入用户 buffer。

zero-copy 前必须注册 buffer：
```python
import numpy as np
from mooncake.store import MooncakeDistributedStore

store = MooncakeDistributedStore()
store.setup(
    "localhost",
    "http://localhost:8080/metadata",
    512 * 1024 * 1024,
    128 * 1024 * 1024,
    "tcp",
    "",
    "localhost:50051",
)

src = np.arange(1024, dtype=np.float32)
dst = np.empty_like(src)

src_ptr = src.ctypes.data
dst_ptr = dst.ctypes.data
size = src.nbytes

store.register_buffer(src_ptr, size)
store.register_buffer(dst_ptr, size)

rc = store.put_from("tensor", src_ptr, size)
if rc != 0:
    raise RuntimeError(f"put_from failed: {rc}")

read_bytes = store.get_into("tensor", dst_ptr, size)
if read_bytes != size:
    raise RuntimeError(f"get_into failed: {read_bytes}")

assert np.array_equal(src, dst)

store.unregister_buffer(src_ptr)
store.unregister_buffer(dst_ptr)
store.close()
```

## 副本和放置策略

`put/put_from`等的目标节点由 Master 在 PutStart 阶段分配，调用方 client 不直接决定。只有挂载了 memory segment 的 client/store 实例才可能被选中，也就是 global_segment_size > 0 的实例。global_segment_size = 0 的实例只是请求方，不贡献存储空间，不会成为 put 目标节点。

可以通过 `ReplicateConfig` 控制副本数量、pin 策略和放置偏好：
```python
from mooncake.store import ReplicateConfig

config = ReplicateConfig()
config.replica_num = 2
config.with_soft_pin = True

store.put("important_key", b"value", config)
```

常用字段：

- `replica_num`：对象副本数，默认 1。
- `with_soft_pin`：软固定。对象在 eviction 中优先保留，但必要时仍可能被回收。
- `with_hard_pin`：硬固定。对象不会被 eviction 回收，需谨慎使用，避免占满内存。
- `preferred_segment`：倾向把对象放到指定 segment。
- `prefer_alloc_in_same_node`：部分 batch zero-copy 场景使用，倾向在同节点分配。

Master 默认 allocation strategy 是 `random`。

- 先处理 ReplicateConfig.preferred_segment 或 preferred_segments。
- preferred segment 有空间就优先分配。
- 如果副本数还没满足，再从所有 segment 中随机选一个起点。
- 从随机起点开始顺序扫描 segment，跳过已经使用过或被排除的 segment。
- 对每个候选 segment 尝试分配对象空间。
- 至少分配到 1 个 replica 才成功；如果完全分配不到，返回 NO_AVAILABLE_HANDLE。

如果集群节点容量差异较大，可以启动 Master 时使用：
```bash
mooncake_master \
  --allocation_strategy=free_ratio_first \
  --enable_http_metadata_server=true \
  --http_metadata_server_port=8080
```

策略会变成：

- 仍然先尝试 preferred segment。
- 对剩余副本，从随机起点采样一批候选 segment。
- 按 free_bytes / capacity 从高到低排序。
- 优先选择空闲比例更高的节点。
- 如果仍不够，再 fallback 到 random 扫描。

## SSD offload 简述

SSD offload 用于把被回收的内存对象异步落到本地 SSD，扩大有效缓存容量。它不改变普通 `put` 的热路径：`put` 仍然先写 memory replica；之后 Master 将对象加入 offload queue，由持有对象的 real client 后台 heartbeat 线程写入 SSD。

读取时，如果 Master 返回的是 memory replica，请求方走普通 Transfer Engine read；如果只剩 `LOCAL_DISK` replica，请求方会联系持有 SSD 数据的目标 client，目标 client 先从 SSD 读到 staging buffer，再由请求方通过 Transfer Engine 拉取到目标 buffer。

### 启用 SSD offload 

Master：
```bash
mooncake_master \
  --rpc_port=50051 \
  --enable_offload=true
```

Embedded Python：
```python
store.setup(
    ...,
    enable_ssd_offload=True,
    ssd_offload_path="/nvme/mooncake_offload",
)
```

Standalone real client：
```bash
export MOONCAKE_OFFLOAD_FILE_STORAGE_PATH=/nvme/mooncake_offload
export MOONCAKE_OFFLOAD_STORAGE_BACKEND_DESCRIPTOR=bucket_storage_backend

mooncake_client \
  --master_server_address=127.0.0.1:50051 \
  --host=<machine IP> \
  --protocol=rdma \
  --device_names=<NIC> \
  --port=50052 \
  --global_segment_size="4 GB" \
  --enable_offload=true \
  --metadata_server=P2PHANDSHAKE
```

## 常用配置建议

**local_buffer_size**

普通 `put/get/batch_put/batch_get` 依赖 `local_buffer_size`。如果对象大小超过可分配 local buffer，普通接口会失败。大对象建议：

- 增大 `local_buffer_size`。
- 或使用 `put_from/get_into`，直接使用用户注册 buffer。

**global_segment_size**

`global_segment_size` 决定本进程贡献多少存储空间：

- 应用进程也作为存储节点：设置为大于 0。
- 只作为请求方：设置为 0。
- 独立 store service 负责存储：应用侧 client 可设置为 0。

**eviction**

Master 通过以下参数控制内存回收：
```bash
--eviction_high_watermark_ratio=0.95
--eviction_ratio=0.05
--allow_evict_soft_pinned_objects=true
```

高水位触发后，Master 会选择部分对象回收内存。开启 SSD offload 时，被 offload 成功的对象仍可从 SSD 回读。

## Mooncake Master 参数说明

### 1. 基础服务参数

<lark-table rows="10" cols="3" column-widths="244,117,330">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--rpc_address`
    </lark-td>
    <lark-td>
      `0.0.0.0`
    </lark-td>
    <lark-td>
      RPC 服务绑定地址
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--rpc_port`
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      RPC 监听端口，优先于 `port`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--port`
    </lark-td>
    <lark-td>
      `50051`
    </lark-td>
    <lark-td>
      旧版 RPC 端口参数（已废弃）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--rpc_thread_num`
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      RPC 线程数
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--max_threads`
    </lark-td>
    <lark-td>
      `4`
    </lark-td>
    <lark-td>
      旧版线程数参数（已废弃）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--rpc_interface`
    </lark-td>
    <lark-td>
      `""`
    </lark-td>
    <lark-td>
      指定用于 RPC 的网卡名
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--rpc_conn_timeout_seconds`
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      RPC 连接超时
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--rpc_enable_tcp_no_delay`
    </lark-td>
    <lark-td>
      `true`
    </lark-td>
    <lark-td>
      是否开启 TCP_NODELAY，降低延迟
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--config_path`
    </lark-td>
    <lark-td>
      `""`
    </lark-td>
    <lark-td>
      配置文件路径
    </lark-td>
  </lark-tr>
</lark-table>

---

### 2. Client / 心跳相关

<lark-table rows="5" cols="3" column-widths="303,106,276">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--client_ttl`
    </lark-td>
    <lark-td>
      `10`
    </lark-td>
    <lark-td>
      Client 心跳超时时间（秒）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--nof_heartbeat_interval_sec`
    </lark-td>
    <lark-td>
      `10`
    </lark-td>
    <lark-td>
      NoF Segment 心跳检测周期
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--nof_heartbeat_probe_timeout_ms`
    </lark-td>
    <lark-td>
      `1000`
    </lark-td>
    <lark-td>
      NoF 心跳 probe 超时
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--nof_heartbeat_failures_threshold`
    </lark-td>
    <lark-td>
      `3`
    </lark-td>
    <lark-td>
      连续失败多少次后认为 segment 挂掉
    </lark-td>
  </lark-tr>
</lark-table>

---

### 3. KV Cache / Lease 参数

<lark-table rows="4" cols="3" column-widths="311,124,250">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--default_kv_lease_ttl`
    </lark-td>
    <lark-td>
      `5000`
    </lark-td>
    <lark-td>
      KV lease 默认 TTL
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--default_kv_soft_pin_ttl`
    </lark-td>
    <lark-td>
      `1800000`
    </lark-td>
    <lark-td>
      soft pin 生命周期
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--allow_evict_soft_pinned_objects`
    </lark-td>
    <lark-td>
      `true`
    </lark-td>
    <lark-td>
      是否允许驱逐 soft pinned 对象
    </lark-td>
  </lark-tr>
</lark-table>

概念说明

Lease TTL表示：KV 对象在多长时间内被认为“有效”

超过时间后：

- master 可回收
- 可触发 eviction
- client 需要续租

---

Soft Pin

表示：

<quote-container>

“尽量别驱逐”，但不是绝对不能驱逐。
</quote-container>

适合：

- 热 KV Cache
- 高频 token block
- 正在 decode 的 cache

---

### 4. Eviction 参数

内存驱逐

<lark-table rows="3" cols="3" column-widths="302,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--eviction_high_watermark_ratio`
    </lark-td>
    <lark-td>
      `0.95`
    </lark-td>
    <lark-td>
      内存使用超过该比例开始驱逐
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--eviction_ratio`
    </lark-td>
    <lark-td>
      `0.05`
    </lark-td>
    <lark-td>
      每次驱逐比例
    </lark-td>
  </lark-tr>
</lark-table>

---

NoF SSD 驱逐

<lark-table rows="3" cols="2" column-widths="336,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--nof_eviction_high_watermark_ratio`
    </lark-td>
    <lark-td>
      `0.95`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--nof_eviction_ratio`
    </lark-td>
    <lark-td>
      `0.05`
    </lark-td>
  </lark-tr>
</lark-table>

用于：

- SSD tier
- NoF backend

---

### 5. Offload / Promotion

Offload

<lark-table rows="4" cols="3" column-widths="200,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_offload`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      是否启用 offload
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--offload_on_evict`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      eviction 时再 offload
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--offload_force_evict`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      超过上限强制驱逐
    </lark-td>
  </lark-tr>
</lark-table>

---

Promotion

<lark-table rows="4" cols="3" column-widths="303,127,218">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--promotion_on_hit`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      读命中时提升到 MEMORY
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--promotion_admission_threshold`
    </lark-td>
    <lark-td>
      `2`
    </lark-td>
    <lark-td>
      命中多少次才允许 promotion
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--promotion_queue_limit`
    </lark-td>
    <lark-td>
      `50000`
    </lark-td>
    <lark-td>
      最大 promotion 队列
    </lark-td>
  </lark-tr>
</lark-table>

---

### 6. CXL 参数

<lark-table rows="5" cols="3" column-widths="200,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_cxl`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      启用 CXL 内存
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--cxl_path`
    </lark-td>
    <lark-td>
      `/dev/dax0.0`
    </lark-td>
    <lark-td>
      DAX 设备
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--cxl_size`
    </lark-td>
    <lark-td>
      `8589934592`
    </lark-td>
    <lark-td>
      CXL 大小（8GB）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--allocation_strategy`
    </lark-td>
    <lark-td>
      `random`
    </lark-td>
    <lark-td>
      segment 分配策略
    </lark-td>
  </lark-tr>
</lark-table>

---

### 7.  allocation_strategy

<lark-table rows="4" cols="2" column-widths="200,200">

  <lark-tr>
    <lark-td>
      策略
    </lark-td>
    <lark-td>
      含义
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `random`
    </lark-td>
    <lark-td>
      随机分配
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `free_ratio_first`
    </lark-td>
    <lark-td>
      优先空闲比例大的
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `cxl`
    </lark-td>
    <lark-td>
      优先 CXL
    </lark-td>
  </lark-tr>
</lark-table>

---

### 8. Snapshot / HA

Snapshot

<lark-table rows="6" cols="3" column-widths="281,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_snapshot`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      开启快照
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--snapshot_interval_seconds`
    </lark-td>
    <lark-td>
      `600`
    </lark-td>
    <lark-td>
      快照周期
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--snapshot_retention_count`
    </lark-td>
    <lark-td>
      `2`
    </lark-td>
    <lark-td>
      保留快照数量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_snapshot_restore`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      启动恢复
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--snapshot_backup_dir`
    </lark-td>
    <lark-td>
      `""`
    </lark-td>
    <lark-td>
      本地备份目录
    </lark-td>
  </lark-tr>
</lark-table>

---

Snapshot Store

<lark-table rows="3" cols="2" column-widths="281,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--snapshot_object_store_type`
    </lark-td>
    <lark-td>
      local / s3
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--snapshot_catalog_store_type`
    </lark-td>
    <lark-td>
      embedded / redis
    </lark-td>
  </lark-tr>
</lark-table>

---

### 9. HA 参数

<lark-table rows="5" cols="3" column-widths="200,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_ha`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      开启高可用
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--ha_backend_type`
    </lark-td>
    <lark-td>
      `etcd`
    </lark-td>
    <lark-td>
      HA backend
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--ha_backend_connstring`
    </lark-td>
    <lark-td>
      `""`
    </lark-td>
    <lark-td>
      backend 连接串
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--etcd_endpoints`
    </lark-td>
    <lark-td>
      `""`
    </lark-td>
    <lark-td>
      etcd 地址
    </lark-td>
  </lark-tr>
</lark-table>

支持：

- etcd
- redis
- k8s

---

### 10. HTTP Metadata Server

<lark-table rows="4" cols="2" column-widths="281,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_http_metadata_server`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--http_metadata_server_host`
    </lark-td>
    <lark-td>
      `0.0.0.0`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--http_metadata_server_port`
    </lark-td>
    <lark-td>
      `8080`
    </lark-td>
  </lark-tr>
</lark-table>

作用：

- 替代 etcd
- 提供 metadata 管理

---

### 11. Metrics

<lark-table rows="3" cols="2" column-widths="281,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_metric_reporting`
    </lark-td>
    <lark-td>
      `true`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--metrics_port`
    </lark-td>
    <lark-td>
      `9003`
    </lark-td>
  </lark-tr>
</lark-table>

---

### 12. Storage Backend

<lark-table rows="4" cols="3" column-widths="256,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--root_fs_dir`
    </lark-td>
    <lark-td>
      `""`
    </lark-td>
    <lark-td>
      存储根目录
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--quota_bytes`
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      存储配额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_disk_eviction`
    </lark-td>
    <lark-td>
      `true`
    </lark-td>
    <lark-td>
      启用磁盘驱逐
    </lark-td>
  </lark-tr>
</lark-table>

---

### 13. Memory Allocator

<lark-table rows="4" cols="3" column-widths="256,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--memory_allocator`
    </lark-td>
    <lark-td>
      `offset`
    </lark-td>
    <lark-td>
      segment allocator
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--use_mmap_arena_allocator`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      mmap arena allocator
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--mmap_arena_pool_size`
    </lark-td>
    <lark-td>
      `8GB`
    </lark-td>
    <lark-td>
      mmap arena pool
    </lark-td>
  </lark-tr>
</lark-table>

---

### 14. Task 管理

<lark-table rows="4" cols="2" column-widths="263,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--max_total_pending_tasks`
    </lark-td>
    <lark-td>
      `10000`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--max_total_processing_tasks`
    </lark-td>
    <lark-td>
      `10000`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--max_total_finished_tasks`
    </lark-td>
    <lark-td>
      `10000`
    </lark-td>
  </lark-tr>
</lark-table>

---

### Timeout

<lark-table rows="5" cols="2" column-widths="287,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--pending_task_timeout_sec`
    </lark-td>
    <lark-td>
      `300`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--processing_task_timeout_sec`
    </lark-td>
    <lark-td>
      `300`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--put_start_discard_timeout_sec`
    </lark-td>
    <lark-td>
      `30`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--put_start_release_timeout_sec`
    </lark-td>
    <lark-td>
      `600`
    </lark-td>
  </lark-tr>
</lark-table>

---

## Mooncake Client 参数说明

### 1. 基础服务参数

<lark-table rows="6" cols="3" column-widths="200,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--host`
    </lark-td>
    <lark-td>
      `0.0.0.0`
    </lark-td>
    <lark-td>
      Client 绑定地址
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--port`
    </lark-td>
    <lark-td>
      `50052`
    </lark-td>
    <lark-td>
      Client 服务端口
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--threads`
    </lark-td>
    <lark-td>
      `1`
    </lark-td>
    <lark-td>
      Client 工作线程数
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--protocol`
    </lark-td>
    <lark-td>
      `tcp`
    </lark-td>
    <lark-td>
      通信协议
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--device_names`
    </lark-td>
    <lark-td>
      `""`
    </lark-td>
    <lark-td>
      RDMA/网络设备名
    </lark-td>
  </lark-tr>
</lark-table>

---

### 2. Master / Metadata Server

<lark-table rows="3" cols="3" column-widths="249,275,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--master_server_address`
    </lark-td>
    <lark-td>
      `127.0.0.1:50051`
    </lark-td>
    <lark-td>
      Mooncake Master 地址
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--metadata_server`
    </lark-td>
    <lark-td>
      `http://127.0.0.1:8080/metadata`
    </lark-td>
    <lark-td>
      Metadata Server 地址
    </lark-td>
  </lark-tr>
</lark-table>

---

架构理解
```plaintext
Client
   ↓
Master
   ↓
Metadata Server
```

其中：Master负责：

- segment 管理
- eviction
- task 调度
- promotion
- HA

---

Metadata Server负责：

- KV 元数据
- object location
- segment metadata

类似：

- etcd
- placement service
- metadata catalog

---

### 3. Global Segment

<lark-table rows="2" cols="3" column-widths="200,108,240">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--global_segment_size`
    </lark-td>
    <lark-td>
      `4 GB`
    </lark-td>
    <lark-td>
      Client 注册的全局 segment 大小
    </lark-td>
  </lark-tr>
</lark-table>

---

### 4. Offload 参数

<lark-table rows="3" cols="3" column-widths="257,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_offload`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      是否启用 offload
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--start_offload_rpc_server`
    </lark-td>
    <lark-td>
      `true`
    </lark-td>
    <lark-td>
      是否启动 offload RPC 服务
    </lark-td>
  </lark-tr>
</lark-table>

---

### 5. start_offload_rpc_server

用于：
```plaintext
其他节点读取当前节点上的 disk-tier object
```

主要接口：

- `batch_get_offload_object`
- `release_offload_buffer`

---

### 6. HTTP Server

<lark-table rows="3" cols="3" column-widths="200,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--enable_http_server`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      启用 HTTP 服务
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--http_port`
    </lark-td>
    <lark-td>
      `9300`
    </lark-td>
    <lark-td>
      HTTP 端口
    </lark-td>
  </lark-tr>
</lark-table>

---

一般用于：

- health check
- metrics
- Prometheus exporter

---

### 7. Memory Allocator

<lark-table rows="3" cols="3" column-widths="245,200,200">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--use_mmap_arena_allocator`
    </lark-td>
    <lark-td>
      `false`
    </lark-td>
    <lark-td>
      启用 mmap arena allocator
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `--mmap_arena_pool_size`
    </lark-td>
    <lark-td>
      `8GB`
    </lark-td>
    <lark-td>
      mmap arena pool 大小
    </lark-td>
  </lark-tr>
</lark-table>

---

### 8. mmap arena allocator

普通 mmap：
```plaintext
频繁 mmap/munmap
    ↓
锁竞争
    ↓
碎片
```

arena allocator：
```plaintext
预申请大块 mmap
    ↓
内部切分
    ↓
减少 syscall
```

适合：

- 高频 KV alloc/free
- token block 分配
- 大规模 cache

---

### 9. device_names

指定：
```plaintext
RDMA / IB 网卡
```

例如：
```bash
--device_names=mlx5_0
```

多个设备：
```bash
--device_names=mlx5_0,mlx5_1
```

---

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/超时|超时]]（issue）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM5-test|GLM5-test]]（model）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OOM|OOM]]（issue）
- [[飞书知识图谱/实体/PD 分离|PD 分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/Prometheus|Prometheus]]（tool）
- [[飞书知识图谱/实体/Qwen3-32B|Qwen3-32B]]（model）
- [[飞书知识图谱/实体/Qwen3-8B|Qwen3-8B]]（model）
- [[飞书知识图谱/实体/RDMA|RDMA]]（concept）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

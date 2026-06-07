---
title: DeepEP通信库使用文档
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-V3
framework:
  - PyTorch
issue_type: []
source: feishu
feishu_token: KzZ2dfkv2oayHhxzgJQcgCfmnnd
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/KzZ2dfkv2oayHhxzgJQcgCfmnnd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# DeepEP通信库使用文档

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/KzZ2dfkv2oayHhxzgJQcgCfmnnd)

# DeepEP 简介

DeepEP 是一个专为混合专家模型（MoE）和专家并行（EP）设计的通信库。它提供了高吞吐、低延迟的 All-to-All GPU 核函数，这些核函数也被称为 MoE 分发（Dispatch）与合并（Combine）。该库还支持包括 FP8 在内的低精度运算。

为了与 DeepSeek-V3 论文中提出的组限制门控算法保持一致，DeepEP 提供了一组针对非对称域带宽转发进行优化的核函数，例如将数据从 NVLink 域转发到 RDMA 域。这些核函数能够提供高吞吐量，非常适合训练和推理预填充任务。此外，它们还支持流多处理器（SM）数量控制。对于延迟敏感的推理解码场景，DeepEP 包含了一组使用纯 RDMA 的低延迟核函数，以最大限度地减少延迟。该库还引入了一种基于钩子（Hook）的通信-计算重叠方法，这种方法不占用任何 SM 资源。

DeepEP将MOE/EP通信的两个All-to-All过程优化为dispatch和combine操作，dispatch 操作的核心作用是将输入token从当前GPU路由到承载其目标专家的其他GPU上，combine 操作是 dispatch的逆过程，它的核心作用是将分散在各个专家GPU上计算完成的输出结果，重新收集并聚合回原始的token位置

### 此通信库包含以下两种模式：

1. **高吞吐模式**

专为需要处理大量数据、最大化整体处理效率的场景而设计，是训练任务和推理预填充（Prefill）阶段的理想选择。此模式的核心优化目标是确保在单位时间内能够处理尽可能多的数据量。

1. Dispatch (分发)

在训练或推理预填充（Prefill）阶段，处理大批量（Batch）的输入数据，包含分发+permute操作。目标是最大化数据传输的总带宽，确保大量 token 能够被高效地分发到各个专家节点，加速整体模型的收敛或处理速度。

1. Combine (合并)
  在训练或预填充阶段，将大量专家计算后的结果高效地聚合起来，包含all-reduce+unpermute操作。目标是最大化数据回传的总带宽，确保聚合操作不会成为处理大批量数据的瓶颈。
1. **低延迟模式**

专注于满足对实时性要求极高的任务需求，特别是推理过程中的解码（Decode）阶段。此模式的核心目标是将单次操作的延迟降至最低，以保证流畅、即时的响应体验。

1. Dispatch (分发)
  服务于推理解码（Decode）阶段。其核心目标是最小化单次分发的端到端延迟，确保实时生成任务的流畅性。
1. Combine (合并)
  在推理解码阶段，快速地将专家计算出的单个 token 结果聚合起来，以便生成下一个 token。核心目标是最小化合并操作的延迟，与dispatch共同保证端到端的实时性能。

# 编译安装

### 环境依赖： 

Pytorch版本要求：pytorch2.1/pytoch2.3/pytoch2.5/pytoch2.7/pytoch2.9/

DTK版本要求：dtk24.10/dtk25.04/dtk26.04

网卡要求：mlnx 网卡或者天龙网卡的库文件和头文件，需编译对应的代码分支。

1. [源码](http%3A%2F%2F42.228.13.241%3A10068%2Fdcutoolkit%2Fdeeplearing%2FDeepEP)编译

编译脚本：

 bash build.sh rocshmem (dtk version>=24.10)

 编译好whl包之后，安装包在dist目录

1. whl包下载：

https://das.sourcefind.cn:55011/portal/#/home

### 安装验证

如果您想验证DeepEP 是否安装正常，可通过以下命令进行验证：
```plaintext
python -c "from deep_ep import version; print(version.__version__)" #若成功打印版本信息则安装成功。
```

# 811 环境注意事项

[[飞书知识图谱/文档/811天龙网卡环境配置和DeepEP使用事项--cqre9nGf|811天龙网卡环境配置和DeepEP使用事项]]

# 环境变量设置
```plaintext
export ROCSHMEM_GDA_NUM_QPS_DEFAULT_CTX=288  # 只用于低延迟，设置大于等于专家数（高吞吐不设置）
export ROCSHMEM_MAX_NUM_CONTEXTS=48          # 不设置默认32，但节点间高吞吐需大于等于设置的SM数
export ROCSHMEM_HEAP_SIZE=3737418240         # 预申请的对称堆大小，默认1G，需大于实际使用size
export ROCSHMEM_TOPO_FILE_FORCE=topo.config  # topo设置
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9
                                             # 可使用的网卡名，可过滤掉不可用网卡

topo.config 文件内容: 第一列 gpubusid，`hy-smi --showbus`可显示。第二列 网卡名，第三列网卡号（跟网卡名后缀对应）
0000:9f:00.0 mlx5_2 2
0000:56:00.0 mlx5_3 3
0000:5d:00.0 mlx5_4 4
0000:05:00.0 mlx5_5 5
0000:e5:00.0 mlx5_6 6
0000:c1:00.0 mlx5_7 7
0000:ca:00.0 mlx5_8 8
0000:b1:00.0 mlx5_9 9

这个是强制指定busid对应gpu 使用gpu最近的网卡，可提升性能。

注意：如果是天龙网卡，需要修改：
export ROCSHMEM_ALLOWED_IBV_DEVICES=shca_0,shca_1,shca_2,shca_3  # 你环境中使用的网卡名
topo.config 也需要修改对应的gpu busid，网卡名和网卡号

其他：
ROCSHMEM_IB_GID_INDEX 设置交换机 GID_INDEX，根据自己环境交换机ROCE版本配置，IB不需要配置。
ROCSHMEM_GDR_DISABLE_XDP 设置是否不使用XDP HEAP memory进行性能加速。
ROCSHMEM_DISABLE_HDP_FLUSH 设置是否不使用HDP 刷新。
```

# 性能测试

单测脚本：

<view type="1">

  <file token="LDExbkQ8moHFTKxEc3dczoWSnEe" name="tests.zip"/>

</view>

注意事项：

1. 单测节点内高吞吐：python test_intranode.py
1. 单测节点内低延迟：ROCSHMEM_HEAP_SIZE=3737418240 python test_low_latency.py
1. 单测节点间高吞吐和低延迟：需要把脚本中ip设置为对应ip，并打开或者注释对应的测试命令

node1：bash 1.sh

node2：bash 2.sh

# Buffer 类 API 参考

**初始化**
```sql
buffer = Buffer(
    group,                          # torch.distributed.ProcessGroup    
    num_nvl_bytes=0,                # NVLink 缓冲区大小（字节）    
    num_rdma_bytes=0,               # RDMA 缓冲区大小（字节）    
    low_latency_mode=False,         # 是否启用低延迟模式    
    num_qps_per_rank=24,            # 每个 rank 的 QP 数量（低延迟模式需等于本地专家数）    
    allow_nvlink_for_low_latency_mode=True,  # 低延迟模式是否允许 NVLink 流量    
    allow_mnnvl=False,              # 是否允许 MNNVL    
    explicitly_destroy=False,       # 是否需要显式调用 destroy()    
    enable_shrink=False,            # 是否启用 shrink 模式（动态屏蔽 ranks）    
    enable_dispatch_ll_layered=False,  # 是否启用分层低延迟分发算子            
    enable_combine_overlap=False,   # 是否使 deepgemm DOWN gemm 与 combine 发送重叠
)
```

参数说明

<sheet token="OE42saZ7qhcrM4tv8N6ci5AQnoc_ffC5ny"/>

#### `Buffer.num_sms`

类属性，用于设置高吞吐 kernel 使用的 SM（Streaming Multiprocessor）数量。默认值为 24，必须是偶数，也需要是3的倍数。目前高吞吐节点内可设置最大60；节点间可设置最大48，但需要设置rocshmem 环境变量大于等于SM数（ROCSHMEM_MAX_NUM_CONTEXTS）。可通过 `Buffer.set_num_sms()` 修改。

#### `set_num_sms(new_num_sms: int) -> None`

设置 SM 数量。

#### `get_low_latency_rdma_size_hint(...) -> int`

获取低延迟模式下 RDMA 缓冲区的最小建议大小（单位：字节）。计算基于 BF16 数据类型。

参数：

- `num_max_dispatch_tokens_per_rank`：每个 rank 最大分发 token 数
- `hidden`：隐藏层维度
- `num_ranks`：EP 组内 rank 数量
- `num_experts`：总专家数
- `enable_dispatch_ll_layered`：是否启用分层低延迟分发
- `quant_group_size`：量化分组大小（0 表示逐 token）

#### `destroy() -> None`

显式销毁 C++ 运行时并释放资源。仅在 `explicitly_destroy=True` 时可用。

#### `get_comm_stream() -> torch.Stream`

返回 Buffer 内部使用的通信流（CUDA stream）。

#### `get_local_buffer_tensor(dtype, size=None, offset=0, use_rdma_buffer=False) -> torch.Tensor`

将内部缓冲区（NVLink 或 RDMA）的一部分作为 PyTorch Tensor 返回。可用于自定义数据读写。

- `dtype`：返回 tensor 的数据类型
- `size`：需要的元素数量（可选，默认为整个缓冲区）
- `offset`：起始元素偏移
- `use_rdma_buffer`：`True` 返回 RDMA 缓冲区，`False` 返回 NVLink 缓冲区

#### `get_dispatch_config(num_ranks) -> Config`

获取针对给定 rank 数量推荐的分发配置（`Config` 对象，包含 SM 数量、块大小等调优参数）。

#### `get_combine_config(num_ranks) -> Config`

获取推荐的合并配置。

#### `get_dispatch_layout(...)`

计算分发所需的布局信息（不实际发送数据）。返回：

- `num_tokens_per_rank`：每个 rank 需要发送的 token 数量
- `num_tokens_per_rdma_rank`：每个 RDMA rank（相同 GPU 索引）需要发送的 token 数量（节点内为 `None`）
- `num_tokens_per_expert`：每个专家接收的 token 数量
- `is_token_in_rank`：布尔张量 `[num_tokens, num_ranks]`，标记每个 token 是否要发送到某个 rank
- `event`：异步完成事件（若 `async_finish=True`）

#### `dispatch(...)`

执行分发操作。根据节点内/节点间自动选择 kernel。支持缓存模式（传入 `handle`）和非缓存模式。

参数（非缓存模式关键参数）：

- `x`：输入 token 张量（BF16）或包含量化 scales 的 tuple
- `topk_idx` / `topk_weights`：每个 token 的专家选择索引和权重
- `num_tokens_per_rank`、`is_token_in_rank`、`num_tokens_per_expert`：由 `get_dispatch_layout` 得到
- `expert_alignment`：每个本地专家接收 token 数量的对齐值
- `num_worst_tokens`：若 >0，则使用 CUDA Graph 兼容模式（仅 intranode）
- `config`：性能调优配置
- `async_finish`：是否异步执行
- `allocate_on_comm_stream`：是否在通信流上分配返回的张量

返回值：

- `recv_x`：接收到的 token（与 `x` 类型一致）
- `recv_topk_idx` / `recv_topk_weights`：接收到的专家索引和权重
- `num_recv_tokens_per_expert`：每个本地专家接收的 token 数量
- `handle`：通信句柄，可用于后续 `combine` 或缓存模式 `dispatch`
- `event`：异步事件

#### `combine(...)`

执行合并操作。必须传入 `dispatch` 返回的 `handle`。

参数：

- `x`：本地专家计算后的输出张量
- `handle`：分发句柄
- `topk_weights`：可选的权重张量，用于加权归约
- `bias`：可选的偏置项（单个 tensor 或两个 tensor 的 tuple）
- `config`、`previous_event`、`async_finish`、`allocate_on_comm_stream`

返回值：

- `recv_x`：归约后的 token 张量
- `recv_topk_weights`：归约后的权重（若有输入）
- `event`：异步事件

#### `internode_dispatch` / `internode_combine`

节点间模式的显式调用接口。通常不应直接调用，使用 `dispatch` / `combine` 即可，它们会自动判断。

#### `clean_low_latency_buffer(...)`

清空低延迟缓冲区，使其归零。在执行过低延迟分发/合并之前，如果缓冲区被其他操作（如高吞吐分发）污染，则需要调用此方法。

#### `low_latency_dispatch(...)`

低延迟分发（基于 IBGDA）。仅支持 BF16 输入，支持量化。

关键参数：

- `x`：BF16 token 张量 `[num_tokens, hidden]`
- `topk_idx`：专家索引 `[num_tokens, num_topk]`
- `num_max_dispatch_tokens_per_rank`：每个 rank 最大分发 token 数（需所有 rank 一致）
- `num_experts`：总专家数
- `quant_type`：量化类型（0=无，1=INT8，2=FP8_E4M3，3=UE8M0，4=FP8_E5M2）
- `quant_group_size`：量化分组大小（0=per‑token，128=per‑group）
- `fp8_round_scale`：是否将 FP8 缩放因子取整为 2 的幂（零硬件开销）
- `return_recv_hook`：若为 `True`，返回一个可调用对象，调用该对象才能确保数据到达

返回值：

- `recv_x`：接收到的数据（根据量化类型可能是 tuple）
- `recv_count`：每个本地专家接收的 token 数量
- `handle`：用于 `low_latency_combine` 的句柄
- `event`：异步事件
- `hook`（可选）：接收钩子函数

#### `low_latency_combine(...)`

低延迟合并（基于 IBGDA）。必须传入 `low_latency_dispatch` 返回的 `handle`。

关键参数：

- `x`：本地计算后的张量（形状与 `low_latency_dispatch` 返回的 `recv_x` 相同）
- `topk_idx`、`topk_weights`：原始分发的索引和权重
- `handle`：分发句柄
- `zero_copy`：是否跳过拷贝（若已通过 `get_next_low_latency_combine_buffer` 直接填充缓冲区）
- `use_logfmt`：是否使用内部 LogFMT 格式（10 bits）
- `out`：可选的输出张量（in‑place）
- `combine_wait_recv_cost_stats`：统计张量 `[num_ranks, num_ranks]`，用于收集等待时间

返回值：

- `combined_x`：合并后的 token 张量
- `event`：异步事件
- `hook`（可选）：接收钩子函数

#### `get_next_low_latency_combine_buffer(handle) -> torch.Tensor`

获取下一个低延迟合并的原始 RDMA 缓冲区（BF16 张量）。用户可直接填充该缓冲区，然后在 `low_latency_combine` 中设置 `zero_copy=True`，避免一次额外的拷贝。

## 注意事项

1. 资源释放

若 `explicitly_destroy=True`，务必在不再使用 Buffer 时调用 `destroy()`，否则可能造成资源泄漏或异常处理挂起。

1. 缓冲区大小

高吞吐模式下，`num_nvl_bytes` 和 `num_rdma_bytes` 应足够大以容纳单次通信的最大数据量。可使用 `get_low_latency_rdma_size_hint` 辅助计算低延迟缓冲区大小。

1. 低延迟模式的限制
	- 所有 rank 必须支持 IBGDA 且通过 RDMA 互联。
	- 同时最多持有 2 个低延迟操作的返回张量（因为内部使用双缓冲）。
	- 使用量化时，需确保所有 rank 的 `quant_type` 和 `quant_group_size` 一致。
1. NVLink 连接检查

初始化时会调用 `check_nvlink_connections(group)`，确保节点内 NVLink 拓扑正确。若检查失败，请检查硬件连接或禁用 NVLink 相关特性。

1. CUDA Graph 兼容性

高吞吐 intranode 分发支持 `num_worst_tokens > 0` 模式，此时不需要 CPU 同步，可用于 CUDA Graph。但需提前预知最坏情况下的 token 数量。

1. 性能调优

可通过 `Buffer.set_num_sms()` 调整 SM 数量，或通过 `get_dispatch_config` / `get_combine_config` 获取推荐配置并传入 `dispatch` / `combine` 的 `config` 参数。

## 常见问题

Q: 低延迟分发返回的 `recv_x` 中有些 token 是无效的，如何过滤？

A: 根据 `recv_count` 张量，每个本地专家仅前 `recv_count[i]` 个 token 是有效的，其余为未使用空间。

Q: 能否混合使用高吞吐和低延迟模式？

A: 可以，但切换模式前必须调用 `clean_low_latency_buffer()` 清空 RDMA 缓冲区，避免数据残留。

Q: 为什么 `dispatch` 需要传入 `num_tokens_per_rank` 等张量？

A: 这些张量可通过 `get_dispatch_layout` 预先计算并复用，避免在每次 `dispatch` 中重复计算布局，提升性能。若传入 `handle`（缓存模式），则可省略这些参数。

Q: 是否支持多节点 NVLink（MNNVL）？

A: 暂不支持。

**常见使用报错：**

Q: Error: modify_qp (RTR): Connection timed out (110) at RocSHMEM

A: 这个一般是交换机不是IB交换机，需设置ROCSHMEM_IB_GID_INDEX 为0（Roce V1）或者1（Roce V2），或者环境中没有可用网卡。

Q：Error: hipGetSymbolAddress(reinterpret_cast(&print_lock_addr), HIP_SYMBOL(print_lock))

A: 这个一般是编译时，选项只指定了更高或较低的gfx架构，而运行在较低或更高的gfx 架构机器上。

Q: no DV library could dlopen for IONIC, BNXT, or MLX5 GDA support.

A: 天龙网卡环境中遇到的问题，一般需要升级环境中的天龙库版本shca-tools.

Q: Failed to lock memory pool：

A: 这个问题可以通过export HSA_USE_SVM=0 来解决，后续runtime会修复。

Q：Failure to open HSA connection: 0x1008

A：manylinux 安装包问题，需要重新出包manylinux包，在出包时把网卡相关库排除掉。

Q：Error: ibv_create_cq_ex: Cannot allocate memory (12) at RocSHMEM::

A：ulimit -a 查看环境中max lock memory值，太小需要环境重新设置，重新建容器，添加 --cap-add=IPC_LOCK --ulimit memlock=-1 选项。

Q: ImportError: libhcoll.so.1: cannot open shared object file: No such file or directory

A：镜像环境中的mpi问题，可以重新编译新mpi解决。

## 关联资源

- [[飞书知识图谱/文档/811天龙网卡环境配置和DeepEP使用事项--cqre9nGf|811天龙网卡环境配置和DeepEP使用事项]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/Z0hBd7Z5coO5oSxeK15cqre9nGf)

## 关联知识

- [[飞书知识图谱/实体/吞吐量|吞吐量]]（metric）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/CUDA Graph|CUDA Graph]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-V3|DeepSeek-V3]]（model）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/RDMA|RDMA]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

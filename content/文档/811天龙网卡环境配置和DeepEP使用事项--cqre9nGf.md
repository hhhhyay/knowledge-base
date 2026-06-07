---
title: 811天龙网卡环境配置和DeepEP使用事项
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type:
  - 卡死
source: feishu
feishu_token: Z0hBd7Z5coO5oSxeK15cqre9nGf
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/Z0hBd7Z5coO5oSxeK15cqre9nGf
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 811天龙网卡环境配置和DeepEP使用事项

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/Z0hBd7Z5coO5oSxeK15cqre9nGf)

# 一、天龙网卡环境配置：

docker环境配置：

**需确认，物理机上和docker内，网卡驱动版本都必须在B068或以上!!!**

**如果物理机上驱动版本不对，请联系李柱。**
```plaintext
查看天龙网卡版本：需要是新版本B068
docker内
readelf --string-dump=shcaVersionInfo /usr/lib/x86_64-linux-gnu/libshca.so 
物理机
readelf --string-dump=shcaVersionInfo /usr/lib64/libshca.so
或 modinfo shca_ib

查看天龙网卡软件包动态库是否符合要求：
nm /usr/lib/x86_64-linux-gnu/libshca.so | grep shca_dv_init_obj


查询安装的deb包： dpkg -l | grep shca
卸载deb包：dpkg -P shca
安装deb包：dpkg -i shca-tools_2.500.4.B068-Ubuntu22.04_amd64.deb # 注意版本 B068

注：如果安装有问题，可能需要卸载ibverbs-utils和infiniband-diags
```

# 二、DeepEP使用事项：

### 1、使用环境：

**Docker中使用：**

1. 由于SHCA网卡软件包与MLNX_OFED软件包存在冲突，二者不能同时安装在一个OS中，因此，在安装SHCA网卡软件包前，需要检查是否存在MLNX_OFED软件包，如果存在，将其卸载。
1. 安装SHCA网卡软件依赖。
1. 安装shca-tools软件包（docker内不需要安装驱动）。
1. 下载DeepEP 天龙网卡版本安装。或者下载[源码](http%3A%2F%2F42.228.13.241%3A10068%2Fdcutoolkit%2Fdeeplearing%2FDeepEP)编译：bash build.sh rocshmem BUILD_SHCA=ON

**Conda中使用：**

1. 不能使用在release docker中或者物理环境中编译的DeepEP。
1. 需要创建基础manylinux 环境docker，再按上面docker配置之后编译的DeepEP版本。

### 2、环境变量：
```plaintext
网络topo
L3交换机可过：
0000:09:00.0 shca_0 0
0000:36:00.0 shca_1 1
0000:55:00.0 shca_0 0
0000:77:00.0 shca_1 1
0000:85:00.0 shca_2 2
0000:b5:00.0 shca_3 3
0000:d5:00.0 shca_2 2
0000:f5:00.0 shca_3 3
L3交换机不可过：目前只能使用2个网卡，后续方案正在开发中
0000:09:00.0 shca_0 0
0000:36:00.0 shca_0 0
0000:55:00.0 shca_0 0
0000:77:00.0 shca_0 0
0000:85:00.0 shca_3 3
0000:b5:00.0 shca_3 3
0000:d5:00.0 shca_3 3
0000:f5:00.0 shca_3 3
```

# 四、mori-ep：低延迟 L3交换机不可过替代方案

Mori-ep 通过 走纯同轨传输达到两个目的：

- 绕过 811环境双平面限制导致 Deepep 低延迟卡死问题；
- 使用 全部4张 RDMA 网卡获取 更高带宽 和 更低延迟。

## 编译流程：

1. clone rocshmem 库（天龙网卡库）：git clone -b rocshmem_shca http://42.228.13.241:10068/dcutoolkit/deeplearing/rocshmem.git
1. 更新 天龙网卡 shca 库：上传改文件到 容器中，然后执行下面的命令。

<view type="1">

  <file token="L4BPbGAQ7oY11bxiSEPc5LuunDd" name="shca-tools_2.500.4.B068-Ubuntu22.04_amd64.deb"/>

</view>

dpkg -i shca-tools_2.500.4.B068-Ubuntu22.04_amd64.deb

1. 建立 /root/rocm 文件夹存储 编译后的 rocshmem 文件
1. 编译rocshmem 库：
	1. cd /rocshmem
	1. mkdir build && cd build
	1. 安装脚本增加可执行权限： `chmod +x  ../scripts/build_configs/gda_mlx5
	1. 编译 `rocshmem`:

 MPI_ROOT=/opt/mpi UCX_ROOT=/opt/ucx INSTALL_PREFIX=/root/rocm/rocshmem-install CMAKE_PREFIX_PATH="/opt/dtk:$CMAKE_PREFIX_PATH" ../scripts/build_configs/gda_shca

1. 编译 mori-ep
	1. git clone -b mori-ep http://42.228.13.241:10068/dcutoolkit/deeplearing/mori.git
	1. 指定 mori-ep 使用 `rocshmem` 库: 修改 mori-ep/build.sh 中 SHMEM_INSTALL_PREFIX 为 `roshmem` 编译路径。
	1. 编译mori-ep：`./build.sh rocshmem`
	1. 安装 whl 包：cd ./dist  然后安装生成的 whl 包

## 性能测试

1. 拉取 mori-ep 代码到 多个节点的共享文件夹中`git clone -b mori-ep ``http://42.228.13.241:10068/dcutoolkit/deeplearing/mori.git`
1. 进入 测试文件夹：
  2node:   mori/tests/2nodes
  4node:   mori/test/4nodes
1. 修改 所有 sh 文件中 `--master-addr=`  参数为 0 号 Rank 的IP地址。
1. 每个节点按照索引顺序，分别执行各自的 sh文件：
          2node: 
                  0 node:  ./0.sh
                  1 node:  ./1.sh 
          4node: 
                  0 node:  ./0.sh
                  1 node:  ./1.sh 
                  2 node:  ./2.sh 
                  3 node:  ./3.sh  
1. 观察各个 节点上的 终端输出。

# 五、常见报错问题：

Q: Error: modify_qp (RTR): Connection timed out (110) at RocSHMEM

A: 这个一般是交换机不是IB交换机，需设置ROCSHMEM_IB_GID_INDEX 为0（Roce V1）或者1（Roce V2）。

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

# 六、DeepEP使用文档参考

[[飞书知识图谱/文档/DeepEP通信库使用文档--cgCfmnnd|DeepEP通信库使用文档]]

## 关联资源

- [[飞书知识图谱/文档/DeepEP通信库使用文档--cgCfmnnd|DeepEP通信库使用文档]] · [在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/KzZ2dfkv2oayHhxzgJQcgCfmnnd)

## 关联知识

- [[飞书知识图谱/实体/卡死|卡死]]（issue）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/RDMA|RDMA]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

---
title: 自动化选优脚本使用说明(持续更新)
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: PdupdKnAjo6Dh9x3OaUcHDsUnwd
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/PdupdKnAjo6Dh9x3OaUcHDsUnwd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 自动化选优脚本使用说明(持续更新)

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/PdupdKnAjo6Dh9x3OaUcHDsUnwd)

## 一、功能说明

只需提供GEMM size文件，以及输入脚本执行所需的参数，脚本会自动进行选优并完成配置文件的更新。

**自动化流程**：拆分size → 进行kernel选优 → 使用原始库和选优库对size进行测试及性能对比 → 生成最优性能配置文件 -> 自动更新配置文件（可选）

### 工具所在目录

`dtk(rocblas-install)/lib/rocblas/auto_select_test/auto_select_tools`

目前建议使用飞书目录下的 auto_select_tools工具。相比 DTK 26.04 及之前版本，该工具已修复相关问题并补充了新功能，后续将在 26.04 之后的正式版本中合入。

### 注意事项

<text bgcolor="light-yellow">若用户需要将选优结果合入正式版本：由于不同 DTK 版本的内部 kernel 不完全一致，选优结果不具备跨版本通用性，BLAS小组仅接收新版本发布前一版本的选优结果。自 DTK 26.04 起，不再接收更早版本结果。</text>

---

## 二、快速开始

### 步骤1：准备输入文件

根据你的数据来源，将 size 文件放到对应目录：

- **CSV格式**：放到 `csv_size/csv/` 目录
- **Layer3格式**：放到 `rocblas_layer3_size/` 目录
- **Layer4格式**：放到 `rocblas_layer4_size/` 目录

详见下方"输入文件格式详解"部分。

### 步骤2：配置参数（必须修改）

编辑 `configs/tuning_args.txt`，**至少修改以下5个必填参数**：

- `mode：运行模式（user/developer），用户仅支持user模式`
- `source：数据来源（csv/layer3/layer4）`
- `hardware_type：硬件平台（ZF/ZF2/KM/KME/ECO/BW/NMZ，必须根据实际平台修改）`
- `gpu_ids：GPU卡号，支持多卡，用空格分隔`
- `cu_count：根据实际环境cu数量设置，用于匹配不同cu的配置文件`

详见下方"参数配置详解"部分。

### 步骤3：执行脚本
```bash
# 1. 进入工具目录
cd dtk(rocblas-install)/lib/rocblas/auto_select_test/auto_select_tools

# 2. 加载DTK环境（根据实际DTK版本路径调整）
source /public/dtk/dtk-26.04-centos8/env.sh

# 3. （可选）如果需要链接测试包
# export LD_LIBRARY_PATH=rocblas-install/lib:$LD_LIBRARY_PATH

# 4. 执行选优脚本（建议保存日志）
python3 Auto_tuning.py 2>&1 | tee tuning.log
```

**用户操作注意事项**：

- 执行前最好新开终端，避免加载多余环境变量
- <text bgcolor="light-yellow">对于不支持最新版DTK的环境，可使用 dtk_To_rocblas_install.sh 将最新版DTK转换为独立的rocblas-install 包。然后加载DTK环境，通过配置 LD_LIBRARY_PATH链接该包后进行选优；完成选优后同样可通过配置 LD_LIBRARY_PATH链接该包进行结果验证。</text>

<view type="2">

  <file token="JvRxbhYmJoq8FGxhVSQcvDp9nxf" name="dtk_To_rocblas_install.sh"/>

</view>

### 步骤4：查看结果

- **配置文件**：`optimization_configs/new/config/library_*/G*`
	- 详见下方"配置文件使用说明"部分。
- **最终结果csv**：`optimization_result/merged_results_*.csv`
	- **注意**：如果需要将优化合入到dtk版本中，需要保存这个目录下的csv文件，交由blas小组合入（需说明dcu型号及cu数量）。
	- 详见下方"结果文件说明"部分。

---

## 三、参数配置详解

执行脚本前，**必须**在 `configs/tuning_args.txt` 中根据需求修改参数。

### 必填参数（必须修改）

<lark-table rows="6" cols="5" column-widths="146,146,146,146,146">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      说明
    </lark-td>
    <lark-td>
      可选值
    </lark-td>
    <lark-td>
      示例
    </lark-td>
    <lark-td>
      注意事项
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `mode`
    </lark-td>
    <lark-td>
      运行模式
    </lark-td>
    <lark-td>
      `user` / `developer`
    </lark-td>
    <lark-td>
      `mode=user`
    </lark-td>
    <lark-td>
      **普通用户选择user,developer仅blas小组内部使用**
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `source`
    </lark-td>
    <lark-td>
      数据来源类型
    </lark-td>
    <lark-td>
      `layer3` / `layer4` / `csv`
    </lark-td>
    <lark-td>
      `source=csv`
    </lark-td>
    <lark-td>
      均支持多文件输入，文件名自定义
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `hardware_type`
    </lark-td>
    <lark-td>
      硬件平台类型
    </lark-td>
    <lark-td>
      `ZF` / `ZF2` / `KM` / `KME` / `ECO` / `BW` / `NMZ`
    </lark-td>
    <lark-td>
      `hardware_type=NMZ`
    </lark-td>
    <lark-td>
      **必须根据实际硬件平台修改**
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `gpu_ids`
    </lark-td>
    <lark-td>
      使用的GPU编号
    </lark-td>
    <lark-td>
      单个或多个，用空格分隔
    </lark-td>
    <lark-td>
      `gpu_ids=4` 或 `gpu_ids=0 1 2 3`
    </lark-td>
    <lark-td>
      确保GPU可用
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      cu_count
    </lark-td>
    <lark-td>
      cu数量
    </lark-td>
    <lark-td>
      **根据实际环境cu数量设置**，用于匹配不同cu的配置文件，0表示是用无后缀的兜底配置文件
    </lark-td>
    <lark-td>
      cu_count=80
    </lark-td>
    <lark-td>
      可通过rocminfo | grep Com查看cu数量
    </lark-td>
  </lark-tr>
</lark-table>

### 重要参数（建议根据需求调整）

<lark-table rows="5" cols="5" column-widths="146,146,146,146,146">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      说明
    </lark-td>
    <lark-td>
      可选值
    </lark-td>
    <lark-td>
      推荐值
    </lark-td>
    <lark-td>
      注意事项
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `strategy`
    </lark-td>
    <lark-td>
      配置生成策略
    </lark-td>
    <lark-td>
      `common/quick`
    </lark-td>
    <lark-td>
      `common`
    </lark-td>
    <lark-td>
      user模式仅支持common
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `should_clean`
    </lark-td>
    <lark-td>
      是否删除临时文件
    </lark-td>
    <lark-td>
      `0/1`
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      **设为0用于选优被中断接着跑；设为1用于选优新的size删除之前结果文件**
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `update_target_config`
    </lark-td>
    <lark-td>
      是否自动更新配置文件到测试包
    </lark-td>
    <lark-td>
      `0/1`
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      **设为0手动更新更安全，避免覆盖原配置**
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `enable_bestkernel_csv`
    </lark-td>
    <lark-td>
      是否启用根据原配置文件找最近的size对应的bestkernel中的MT参数进行校验，从而减少选优用到的kernel数量
    </lark-td>
    <lark-td>
      `0/1`
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      size比较大时建议启用
    </lark-td>
  </lark-tr>
</lark-table>

### 性能调优参数（可选，保持默认即可）

<lark-table rows="8" cols="4" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      参数
    </lark-td>
    <lark-td>
      说明
    </lark-td>
    <lark-td>
      默认值
    </lark-td>
    <lark-td>
      何时调整
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `chunk_size`
    </lark-td>
    <lark-td>
      每张卡每次选优的size数量
    </lark-td>
    <lark-td>
      `40`
    </lark-td>
    <lark-td>
      size较少时可减小
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `tune_warmup_times`
    </lark-td>
    <lark-td>
      选优预热次数
    </lark-td>
    <lark-td>
      `2`
    </lark-td>
    <lark-td>
      一般不需要调整
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `tune_loop_times`
    </lark-td>
    <lark-td>
      选优循环次数
    </lark-td>
    <lark-td>
      `4`
    </lark-td>
    <lark-td>
      一般不需要调整
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `bench_warmup_times`
    </lark-td>
    <lark-td>
      测试预热次数
    </lark-td>
    <lark-td>
      `2`
    </lark-td>
    <lark-td>
      一般不需要调整
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `bench_loop_times`
    </lark-td>
    <lark-td>
      测试循环次数
    </lark-td>
    <lark-td>
      `10`
    </lark-td>
    <lark-td>
      一般不需要调整
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `test_times`
    </lark-td>
    <lark-td>
      测试轮次
    </lark-td>
    <lark-td>
      `1`
    </lark-td>
    <lark-td>
      需要更准确结果时可增加
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `enable_mk_n_threshold`
    </lark-td>
    <lark-td>
      是否启用自动归类到infer
    </lark-td>
    <lark-td>
      `0`
    </lark-td>
    <lark-td>
      一般不需要调整
    </lark-td>
  </lark-tr>
</lark-table>

**重要注意事项**：

- `should_clean=1` 会删除 `optimization_configs` 和 `optimization_artifacts` 目录下的内容，**删除前请确保已备份结果配置文件**；
- 设为0用于选优被中断接着跑；设为1用于选优新的size删除之前所有过程和结果文件；
- `update_target_config=1` 会自动更新覆盖测试包中的原配置文件，**如果开启建议先备份原配置文件**，设为0手动更新更安全，避免覆盖原配置；

---

## 四、输入文件格式详解

### 1. CSV格式输入(支持tf32)

**输入目录**：`csv_size/csv/`

**文件格式**：CSV文件，表头必须包含：`gemmtype,TransA,TransB,M,N,B,K`

**完整示例文件**（`csv_size/csv/size.csv`）：
```plaintext
gemmtype,TransA,TransB,M,N,K,B
sgemm,N,N,1024,1024,1024,1
sgemm,N,T,2048,2048,2048,1
hpa,N,N,4096,4096,4096,1
bf16,T,N,512,512,512,1
tf32,T,N,512,512,512,1
```

**特殊用法：N值自动遍历**（仅限KME/ECO/BW/NMZ平台）

- **适用场景**：MK固定，N需要遍历取值
- **使用方法**：N值写为 `-`，脚本会自动遍历
- **限制条件**：
	- 仅支持 KME/ECO/BW/NMZ平台的 hpa/bf16/int8 类型
	- 仅支持 NN/TN 转置类型
	- B 必须为 1
- **示例**：
```plaintext
gemmtype,TransA,TransB,M,N,B,K
hpa,N,N,4096,-,1,4096    # N会自动遍历
bf16,T,N,4096,-,1,4096   # N会自动遍历
```

- **不满足限制条件的情况**：N必须是具体数值，不能使用 `-`

### 2. Layer3格式输入(支持tf32)

通过环境变量export ROCBLAS_LAYER=3打印

**输入目录**：`rocblas_layer3_size/`

**文件格式**：log或txt文件，每行一条rocblas-bench命令

**示例文件内容**（`rocblas_layer3_size/layer3.log`）：
```plaintext
./rocblas-bench -f gemm_ex --transposeA N --transposeB N -m 512 -n 16384 -k 512 --alpha 1 --a_type f16_r --lda 512 --b_type f16_r --ldb 512 --beta 0 --c_type f16_r --ldc 512 --d_type f16_r --ldd 512 --compute_type f32_r --algo 0 --solution_index 0 --flags 0
./rocblas-bench -f gemm_ex --transposeA N --transposeB T -m 1024 -n 1024 -k 1024 --alpha 1 --a_type f32_r --lda 1024 --b_type f32_r --ldb 1024 --beta 0 --c_type f32_r --ldc 1024 --d_type f32_r --ldd 1024 --compute_type f32_r --algo 0 --solution_index 0 --flags 0
./rocblas-bench -f gemm_ex --transposeA N --transposeB T -m 1024 -n 1024 -k 1024 --alpha 1 --a_type f32_r --lda 1024 --b_type f32_r --ldb 1024 --beta 0 --c_type f32_r --ldc 1024 --d_type f32_r --ldd 1024 --compute_type f32_r --algo 0 --solution_index 0 --flags 0 --math_mode 1
```

**重要**：

- 使用 layer3 抓取的 rocblas-bench 指令中，TF32 与 SGEMM 未做区分。若需进行 TF32 选优，需要在每条 rocblas-bench 命令后手动添加参数 “<text bgcolor="light-yellow">--math_mode 1</text>”。
- 必须保证每行只写一条指令，否则程序会报错。

### 3. Layer4格式输入（不支持tf32）

通过环境变量export ROCBLAS_LAYER=4打印

**输入目录**：`rocblas_layer4_size/`

**文件格式**：log或txt文件，每行一条YAML格式的配置

**示例文件内容**（`rocblas_layer4_size/layer4.log`）：
```plaintext
- { rocblas_function: "rocblas_gemm_ex", atomics_mode: atomics_allowed, a_type: "f16_r", b_type: "f16_r", c_type: "f16_r", d_type: "f16_r", compute_type: "f32_r", transA: 'N', transB: 'N', M: 108, N: 108, K: 108, alpha: 1.0, lda: 108, ldb: 108, beta: 0.0, ldc: 108, ldd: 108, algo: 0, solution_index: 0, flags: none, call_count: 1 }
- { rocblas_function: "rocblas_gemm_ex", atomics_mode: atomics_allowed, a_type: "f32_r", b_type: "f32_r", c_type: "f32_r", d_type: "f32_r", compute_type: "f32_r", transA: 'N', transB: 'T', M: 256, N: 256, K: 256, alpha: 1.0, lda: 256, ldb: 256, beta: 0.0, ldc: 256, ldd: 256, algo: 0, solution_index: 0, flags: none, call_count: 1 }
```

**重要**：必须保证每行只写一条指令，否则程序会报错。

---

## 五、GEMM/转置类型支持情况

### 硬件平台与支持的GEMM类型对照表

<lark-table rows="8" cols="3" column-widths="244,244,244">

  <lark-tr>
    <lark-td>
      hardware_type
    </lark-td>
    <lark-td>
      支持的gemmtype
    </lark-td>
    <lark-td>
      配置文件目录名
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `ZF`
    </lark-td>
    <lark-td>
      sgemm, hpa, hgemm
    </lark-td>
    <lark-td>
      `library`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `ZF2`
    </lark-td>
    <lark-td>
      sgemm, hpa, hgemm, dgemm
    </lark-td>
    <lark-td>
      `library_gpu2`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `KM`
    </lark-td>
    <lark-td>
      sgemm, hpa
    </lark-td>
    <lark-td>
      `library_gpu3`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `KME`
    </lark-td>
    <lark-td>
      sgemm, hpa, bf16, int8
    </lark-td>
    <lark-td>
      `library_gpu4`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `ECO`
    </lark-td>
    <lark-td>
      sgemm, hpa, bf16, int8
    </lark-td>
    <lark-td>
      `library_gpu4`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `BW`
    </lark-td>
    <lark-td>
      sgemm, hpa, dgemm, bf16, int8, bss, cgemm, zgemm,tf32
    </lark-td>
    <lark-td>
      `library_gpu5`
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `NMZ`
    </lark-td>
    <lark-td>
      sgemm, hpa, dgemm, bf16, int8, bss, cgemm, zgemm,tf32
    </lark-td>
    <lark-td>
      `library_gpu6`
    </lark-td>
  </lark-tr>
</lark-table>

### 转置类型支持情况

**标准转置类型**（sgemm/hpa/dgemm/bf16/int8/bss）：

- `NN`
- `NT`
- `TN`
- `TT`

**复数类型额外支持**（cgemm/zgemm，仅BW/NMZ平台）：

- `NN`、`NT`、`TN`、`TT`
- `NC`：A不转置，B共轭
- `CN`：A共轭，B不转置

### 重要提示

1. **自动过滤**：脚本会自动过滤不匹配的 gemm size
2. **错误处理**：如果输入的所有 size 都不匹配，程序会终止并报错
3. **检查方法**：执行前请确认：
	- `hardware_type` 参数与你的硬件平台匹配
	- 输入的 `gemmtype` 在你的平台上支持
	- 输入的转置类型在你的平台上支持

---

## 六、配置文件使用说明

### 配置文件位置

**路径**：`optimization_configs/new/config/library_*`

**说明**：

- `library_*` 的名称因平台不同而有所差异（具体对应关系见"GEMM/转置类型支持情况"表格）
- 这些是最终生成的配置文件，用于替换rocblas库中的配置文件

**示例**：

- NMZ平台：`optimization_configs/new/config/library_gpu6/G*`
- BW平台：`optimization_configs/new/config/library_gpu5/G*`
- KME平台：`optimization_configs/new/config/library_gpu4/G*`

### 三种使用方式

#### 方式1：手动替换配置文件（推荐，最安全）
```bash
# 复制配置文件到rocblas库目录
cp optimization_configs/new/config/library_*/G* Path/lib/rocblas/library_*

# 示例（NMZ平台）
cp optimization_configs/new/config/library_gpu6/G* /path/to/rocblas/lib/rocblas/library_gpu6/
```

**优点**：完全控制，不会意外覆盖

**缺点**：需要手动操作

#### 方式2：设置环境变量（临时使用）
```bash
# 设置环境变量指向新配置文件目录
export ROCBLAS_TENSILE_LIBPATH=$(pwd)/optimization_configs/new/config/library_*

# 示例（NMZ平台）
export ROCBLAS_TENSILE_LIBPATH=$(pwd)/optimization_configs/new/config/library_gpu6
```

**优点**：不需要修改原配置文件

**缺点**：仅当前终端会话有效，关闭终端后失效

#### 方式3：自动更新（需谨慎）

在 `configs/tuning_args.txt` 中设置：
```bash
update_target_config=1
```

**优点**：自动化，无需手动操作

**缺点**：**会自动覆盖源目录配置文件，请提前备份原配置文件**

## 七、结果文件说明

### 1. 最终选优结果

**文件路径**：`optimization_results/merged_results_{source_type}_{timestamp}.csv`

**说明**：

- **注意**：<text bgcolor="light-yellow">如果需要将优化合入到dtk版本中，需要保存这个目录下的csv文件，交由blas小组合入（需说明选优环境及cu数）。</text>
- 文件包含所有选优结果的整合数据
- 文件名包含来源类型（custom/infer/general）和时间戳
- **不会被后续执行删除**，可以安全保存

**示例文件名**：

- `optimization_results/merged_results_custom_20251224_143022.csv`
- `optimization_results/merged_results_infer_20251224_143022.csv`

### 2. csv文件表头说明

**表头**：`gemmtype,TransA,TransB,M,N,K,B,old_gflops,old_kernel,tune_gflops,tune_kernel,new_gflops,new_kernel,new_kernelSource,bestkernel,compare_kernel,gflops_ratio`

**字段说明**：

<lark-table rows="14" cols="2" column-widths="365,365">

  <lark-tr>
    <lark-td>
      字段
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `old_gflops`
    </lark-td>
    <lark-td>
      old库测试性能
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `old_us`
    </lark-td>
    <lark-td>
      old库测试耗时
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `old_kernel`
    </lark-td>
    <lark-td>
      old库使用的kernel
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `old_kernelSource`
    </lark-td>
    <lark-td>
      来源于old库中的哪个表（Add/AI/General/Origin...）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `tune_gflops`
    </lark-td>
    <lark-td>
      选优的性能
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `tune_kernel`
    </lark-td>
    <lark-td>
      选优选择的kernel名称
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `new_gflops`
    </lark-td>
    <lark-td>
      new库测试性能
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `new_us`
    </lark-td>
    <lark-td>
      new库测试耗时
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `new_kernel`
    </lark-td>
    <lark-td>
      new库使用的kernel
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `new_kernelSource`
    </lark-td>
    <lark-td>
      来源于new库中的哪个表
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `bestkernel`
    </lark-td>
    <lark-td>
      最优性能对应的kernel名称
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `compare_kernel`
    </lark-td>
    <lark-td>
      选优前后kernel是否一样
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      `gflops_ratio`
    </lark-td>
    <lark-td>
      性能对比比率 = new_gflops / old_gflops（>1表示性能提升）
    </lark-td>
  </lark-tr>
</lark-table>

**用途**：用于分析选优效果，查看性能提升情况, 制作配置文件

---

## 八、常见问题处理

### 问题1：程序中断后如何继续？

**解决方法**：

1. 将 `configs/tuning_args.txt` 文件中 `should_clean` 设为 `0`
2. 加载环境变量
3. 重新执行脚本：
```bash
python3 -u Auto_tuning.py
```

4. 脚本会从上次中断的地方继续执行

### 问题2：脚本报错"没有有效的 size"？

**可能原因**：

1. 输入的 gemmtype 与硬件平台不匹配
2. 输入的转置类型不支持
3. CSV 文件格式错误

**解决方法**：

1. 检查 `hardware_type` 参数是否正确
2. 查看支持的 gemmtype 和转置类型（见下文表格）
3. 检查 CSV 文件格式是否正确

### 问题3：如何基于上次选优结果继续优化出包？

**解决方法**：

需要将生成的配置文件替换到原rocblas库中：

**方法1**：如果 `update_target_config=0`（手动更新）
```bash
# 手动复制配置文件
cp optimization_configs/new/config/library_*/G* Path/lib/rocblas/library_*
# 然后重新执行选优脚本
python3 -u Auto_tuning.py
```

**方法2**：如果 `update_target_config=1`（自动更新）

- 配置文件会自动更新到测试包对应目录
- **注意**：会覆盖源目录配置文件，注意提前备份原配置文件
- 下次执行选优即可基于上次选优的配置文件继续优化

### 问题4：测试阶段执行失败检测功能

**功能说明**：

脚本在执行原库测试、新库测试、获取kernel等关键步骤时，会自动检测失败情况。如果检测到失败，会立即打印详细的错误信息并退出程序，帮助用户快速定位问题。

**失败检测覆盖的场景**：

1. **原库/新库测试失败**：测试任务执行失败时，会显示失败的日志文件路径
2. **测试结果合并失败**：CSV文件合并失败时，会显示预期的文件路径
3. **测试结果为空**：读取的CSV文件为空时，会显示文件路径
4. **获取kernel失败**：获取kernel任务失败时，会显示失败的CSV文件路径
5. **kernel结果无效**：kernel结果文件无效或缺少必要列时，会显示文件路径和数据信息
6. **数据合并失败**：最终数据合并失败时，会显示各数据源的行数信息

**错误信息格式**：

所有错误信息都包含具体的文件路径，格式如下：
```plaintext
错误: [错误描述]
[文件路径/目录路径]
[其他详细信息]
```

**示例错误信息**：
```plaintext
错误: 原库测试失败，2/6 个任务失败
失败的日志文件:
  - optimization_artifacts/custom/compare/log/old/task_0_test_round0.log
  - optimization_artifacts/custom/compare/log/old/task_1_test_round0.log
```

```plaintext
错误: 原库kernel测试结果无效（检查是否加入额外环境变量）
文件路径: optimization_artifacts/custom/compare/csv/kernel/sgemm_NT.csv
数据行数: 0, 列: ['M', 'N', 'K', 'B']
```

### 问题5：common选优任务失败检测功能

**功能说明**：

脚本在执行common策略的kernel选优过程中，会自动检测选优任务是否成功完成。如果检测到失败的任务，会立即打印详细的失败任务信息并退出程序，帮助用户快速定位问题。

**检测时机**：

所有common策略的选优任务完成后，统一检查是否有任务的结果文件缺失或为空。

**检测机制**：

- 检查任务结果文件是否存在（`task_{idx}.csv`）
- 检查结果文件是否为空（文件大小为0）
- 如果trained目录不存在，则认为该类型的所有任务都失败

**错误信息格式**：

失败任务会按类型分组显示，格式如下：
```plaintext
============================================================
错误: common策略选优阶段发现失败的任务
============================================================
失败任务列表:
  - [tune_type] [gemm_type] [transpose_type]：任务 [任务索引列表] (共N个)
程序终止
```

**示例错误信息**：
```plaintext
============================================================
错误: common策略选优阶段发现失败的任务
============================================================
失败任务列表:
  - custom sgemm NT：任务 [0, 2, 5] (共3个)
  - infer hpa NN：任务 [1] (共1个)
程序终止
```

**失败任务文件位置**：

`optimization_artifacts/{tune_type}/{gemm_type}/{transpose_type}/trained/task_{idx}.csv`

**排查建议**：

1. 查看错误信息中显示的任务索引，检查对应的结果文件是否存在
2. 检查 `optimization_artifacts/{tune_type}/{gemm_type}/{transpose_type}/log/` 目录下的日志文件

### 问题6：quick策略选优任务失败检测功能（blas小组内部使用）

**功能说明**：

脚本在执行quick策略的kernel选优过程中，会自动检测选优任务是否成功完成。quick策略分为两个阶段：样本选优和快速选优，每个阶段完成后都会进行失败检测。如果检测到失败的任务，会立即打印详细的失败任务信息并退出程序。

**检测时机**：

1. **样本选优阶段**：所有样本选优任务完成后，检查custom/infer/general三类选优任务是否成功
2. **快速选优阶段**：所有快速选优任务完成后，检查快速选优任务的结果文件是否成功生成

**检测机制**：

- **样本选优阶段**：检查 `task_{idx}.csv` 文件是否存在且不为空
- **快速选优阶段**：检查 `task_quick_{idx}.json` 文件是否存在且不为空
- 如果trained目录不存在，则认为该类型的所有任务都失败

**错误信息格式**：

失败任务会按类型分组显示，格式如下：
```plaintext
============================================================
错误: [阶段名称]阶段发现失败的任务
============================================================
失败任务列表:
  - [tune_type] [gemm_type] [transpose_type]：任务 [任务索引列表] (共N个)
程序终止
```

**示例错误信息**：

样本选优阶段失败：
```plaintext
============================================================
错误: 样本选优阶段发现失败的任务
============================================================
失败任务列表:
  - custom sgemm NT：任务 [0] (共1个)
  - infer bf16 TN：任务 [0, 1] (共2个)
程序终止
```

快速选优阶段失败：
```plaintext
============================================================
错误: 快速选优阶段发现失败的任务
============================================================
失败任务列表:
  - infer hpa NN：任务 [2, 3] (共2个)
  - general bf16 TN：任务 [1] (共1个)
程序终止
```

**失败任务文件位置**：

- **样本选优阶段**：`optimization_artifacts/{tune_type}/{gemm_type}/{transpose_type}/trained/task_{idx}.csv`
- **快速选优阶段**：`optimization_artifacts/{tune_type}/{gemm_type}/{transpose_type}/trained/task_quick_{idx}.json`

**排查建议**：

1. 查看错误信息中显示的任务索引和阶段名称，检查对应的结果文件是否存在
2. 检查 `optimization_artifacts/{tune_type}/{gemm_type}/{transpose_type}/log/` 目录下的日志文件

---

## 九、用户模式目录结构说明

### 完整目录结构
```plaintext
├── configs
│       ├── multinode_args.txt      (多节点结果整合参数输入文件，仅developer使用)
│       └── tuning_args.txt         **(选优参数输入文件)**
│       
│
├── rocblas_layer3_size
│       ├── layer3.log              **(source为layer3时输入目录)**
│       ├── csv
│       │   ├── hpa_NN.csv          (size拆分生成的过程文件) 
│       │   └── bf16_TN.csv
│       │
│       ├──hpa_NN_supplement.csv    (size拆分完成生成的文件)
│       └──bf16_TN_inference.csv    
│
│
├── rocblas_layer4_size
│       ├── layer4.log              **(source为layer4时输入目录)**
│       ├── csv
│       │   ├── hpa_NN.csv 
│       │   └── bf16_TN.csv
│       │  
│       ├──hpa_NN_supplement.csv
│       └──bf16_TN_inference.csv
│ 
│
├── csv_size
│       ├── csv
│       │   └── size.csv            **(source为size时输入目录)**
│       │ 
│       ├──hpa_NN_supplement.csv 
│       └──bf16_TN_inference.csv 
│                                
|
├── optimization_artifacts          (选优测试过程文件目录)
│       ├── infer                                     
│       │   ├── bf16                (kernel选优过程文件目录)
│       │   │    └── TN
│       │   │        ├── log        (选优日志)
│       │   │        ├── task_txt 
│       │   │        ├── trained
│       │   │        └── kernels_bf16_TN_dce1c742.txt
│       │   │
│       │   ├── compare             (测试过程文件目录)
│       │   │    ├── csv
│       │   │    ├── log
│       │   │    └── txt
│       │   │
│       │   └── result              (kernel选优结果目录)
│       │   │    ├── csv 
│       │   │    │   └── hpa_NN.csv 
│       │   │    └── json
│       │
│       ├── custom                  
│       │   ├── hpa
│       │   │    └── NN
│       │   │        ├── log
│       │   │        ├── task_txt
│       │   │        ├── trained
│       │   │        └── kernels_hpa_NN_dce1c742.txt
│       │   ├── compare
│       │   │    ├── csv
│       │   │    │   └── hpa_NN.csv
│       │   │    ├── log
│       │   │    └── txt
│       │   │
│       │   └── result
│       │   │    ├── csv 
│       │   │    │   └── hpa_NN.csv
│       │   │    └── json
│       │   
|       └── general
│
│                                
├── optimization_configs
|    ├── new
│    │   ├── config 
│    │   │   └── library_*            **(配置文件结果目录)**
│    │   └── csv          
│    │       └── library_*
│    │           ├── custom
│    │           └── infer
|    └── old
│        ├── config                   (原配置文件备份)
│        │   └── library_* 
│        └── csv
│            └── library_*
│                ├── infer
|                └── custom
│
├── optimization_results              **(最终选优结果csv，提供给BLAS小组)**
```

## 十、开发者模式(mode=developer，blas小组内部使用)

### 1.mode（user/developer）主要区别：

1. user和developer基本一样，但developer模式额外包含以下步骤：

a.跟user模式主要区别在于developer模式支持快速选优策略（strategy=quick）

        此时样本选优结果在csv目录下，根据样本进行选优的最终结果在quick_csv目录下。

b.user从配置文件中获取kernel无需手动添加，developer模式下需要手动添加选优所需的kernel，在developer_kernels目录下：
```plaintext
├── developer_kernels
│       ├── txt
│       │   └── hpl_lite_gpu4
│       │       └── kernels_bf16_NN.txt(首先检查是否存在，存在则kernel从txt读)
│       └── yaml
│           └── hpl_lite_gpu4 (yaml文件)
│              └── kongmingE_Cijk_Ailk_BIjk_HBH.yaml（不存在从yaml中读）
```

### 2.多节点结果整合快速开始
```plaintext
   a.快速开始
           步骤1：配置参数
                tune_type：custom|infer|general(选优结果存在于哪个表填哪个，支持多选)
                hardware_type：平台类型(ZF|ZF2|KM|KME|ECO|BW|NMZ)
                cu_count：根据实际环境cu数量设置，用于匹配不同cu的配置文件
                （必须修改configs/multinode_args.txt上述三个参数，其余用默认值即可）
           步骤2：准备原配置文件
                   需将原配置文件手动放入multi_node_configs/old/config/library_*
           步骤3：准备选优结果
                   详见下方"目录结构"部分。
           步骤4：执行脚本
                   python3 Auto_tuning.py -m
           步骤5：结果目录
                   新配置文件目录：multi_node_configs/new/config/library_*
   b.目录结构：
   ├── multi_node_configs
   │       ├── infer
   │       │   ├── compare (保存一些测试过程文件，默认不开启测试)
   │       │   │
   │       │   ├── result_csv
   │       │   │   ├── hpa_NN (创建名为gemmtype_transpose文件夹)
   │       │   │   │   ├──task_0.csv
   │       │   │   │   ├──task_2.csv
   │       │   │   │   └──task
   │       │   │   │      └── task.csv
   │       │   │   └── bf16_NN
   │       │   │       ├──task_0.csv
   │       │   │       └──task_2.csv
   │       │   │
   │       │   ├── hpa_NN.csv (如果已经有了gemmtype_transpose.csv直接放到这个目录)
   │       │   └── bf16_NN.csv
   │       │
   │       ├── custom
   │       │    └── 同上
   │       │      
   │       │
   │       ├── general
   │       │   └── 同上
   │       │
   │       ├── old
   │       │    ├── config
   │       │    │    └──library_gpu*(准备工作)
   │       │    └── csv
   │       └── new
   │       │    ├── config
   │       │    │    └──library_gpu*
   │       │    └── csv
  
```

- 准备选优结果举例：

   Ⅰ.如果是infer表的hpa-NN类型存在多个节点的选优结果，那么需要在infer/result_目录下创建名为hpa_NN的文件夹（文件夹名需严格按照{gemmtype}_{transpose}格式创建）。

   Ⅱ.创建hpa_NN后，将多节点的选优结果放入该目录（该目录下可任意命名子文件夹/子文件名），执行python3 Auto_tuning.py -m程序会遍历的将hpa_NN目录下的所有结果合并到infer/result_csv/hpa_NN.csv。

   Ⅲ.如果本身已经有了infer表hpa_NN.csv(文件名需严格按照{gemmtype}_{transpose}.csv格式创建)，直接放入infer/result_csv目录下，不需要进行上述操作。

   Ⅳ.如果存在多种gemm类型转置类型重复上述的操作放到对应目录即可，准备完成后执行脚本进行新配置文件的生成。

---

## 十一、总结

### 核心要点

1. **使用简便**：只需提供 size 文件，配置参数，执行脚本即可
2. **完全自动化**：size 拆分、kernel 选优、性能测试、配置文件生成全部自动化
3. **结果文件**：**最终结果在 `optimization_results/merged_results_*.csv`
4. **配置文件**：生成的配置文件在 `optimization_configs/new/config/library_*/G*`

### 快速检查清单

执行前请确认：

- 输入文件已放到正确目录
- `tuning_args.txt` 中的 `mode`、`source`、`hardware_type`、`gpu_ids`、`cu_count` 已正确配置
- 环境变量已加载
- GPU 可用

执行后请检查：

- `optimization_results/` 目录下有最终结果文件
- `optimization_configs/new/config/library_*/` 目录下有配置文件

### 注意事项

- 除少数特殊情况（如 kernel vmfault）外，脚本会自动生成更新后的配置文件
- 如有问题，请保留现场日志文件联系BLAS小组

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）

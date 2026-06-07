---
title: 20260421_GLM5 hipblaslt  rocblas size统计与tuning
tags:
  - AI
  - LLM
  - Benchmark
model:
  - GLM-5-W8A8
framework:
  - SGLang
  - vLLM
issue_type: []
source: feishu
feishu_token: VTfld4PyYozMwgxxgfVcTs79nkg
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/VTfld4PyYozMwgxxgfVcTs79nkg
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 20260421_GLM5 hipblaslt  rocblas size统计与tuning

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/VTfld4PyYozMwgxxgfVcTs79nkg)

**本文操作步骤整理自第 6 节所列参考文献。遇到问题可以查阅对应参考文献，或咨询参考文献负责人。**

# 1 **hipblaslt gemm size**

## 1.1 抓log

export HIPBLASLT_LOG_MASK=32

并且需要设置--api-server-count 1，否则后面没有打印。
```bash
export HIPBLASLT_LOG_MASK=32
vllm serve /models/GLM-5-W8A8  \
  --disable-log-requests \
  -q slimquant_marlin \
  --trust-remote-code \
  -dp 8 \
  -tp 1 \
  --enable-expert-parallel \
  --disable-custom-all-reduce \
  --dtype bfloat16 \
  --enable-chunked-prefill \
  --max-model-len 50000 \
   --max-num-batched-tokens 128 \
  --max-num-seqs 32 \
  --enable-prefix-caching \
  --block-size 64 \
  --gpu-memory-utilization 0.88 \
  --kv-cache-dtype fp8_ds_mla \
  -cc '{"inductor_compile_config":{"combo_kernels": false}}' \
--speculative_config '{"method":"mtp","num_speculative_tokens":3, "quantization": "slimquant_marlin"}'  \
--api-server-count 1  >&hipblas.txt
```

发送请求
```bash
vllm bench serve --dataset-name random --model /models/GLM-5-W8A8 \
--num-prompts 1 \
--random-input-len 32 \
--random-output-len 1 \
--random-range-ratio 0.0 \
--endpoint /v1/completions \
--tokenizer /models/GLM-5-W8A8 \
--temperature 0.0 \
--ignore-eos \
--host 12.12.12.55
```

log如下，
```bash
hipblaslt-bench --api_method c -m 28672 -n 512 -k 512 --lda 512 --ldb 512 --ldc 28672 --ldd 28672  --stride_a 14680064 --stride_b 262144 --stride_c 14680064 --stride_d 14680064  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 28672 -n 512 -k 512 --lda 512 --ldb 512 --ldc 28672 --ldd 28672  --stride_a 14680064 --stride_b 262144 --stride_c 14680064 --stride_d 14680064  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 28672 -n 512 -k 512 --lda 512 --ldb 512 --ldc 28672 --ldd 28672  --stride_a 14680064 --stride_b 262144 --stride_c 14680064 --stride_d 14680064  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 28672 -n 512 -k 512 --lda 512 --ldb 512 --ldc 28672 --ldd 28672  --stride_a 14680064 --stride_b 262144 --stride_c 14680064 --stride_d

......

hipblaslt-bench --api_method c -m 4096 -n 32 -k 6144 --lda 6144 --ldb 6144 --ldc 4096 --ldd 4096  --stride_a 25165824 --stride_b 196608 --stride_c 131072 --stride_d 131072  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  
hipblaslt-bench --api_method c -m 6144 -n 32 -k 2048 --lda 2048 --ldb 2048 --ldc 6144 --ldd 6144  --stride_a 12582912 --stride_b 65536 --stride_c 196608 --stride_d 196608  --alpha 1 --beta 0 --transA T --transB N --batch_count 1 --scaleA 2 --scaleB 2  --a_type i8_r --b_type i8_r --c_type i32_r --d_type bf16_r --scale_type f32_r --bias_type f32_r   --compute_type i32_r --activation_type none  

```

log刷屏打印的太多了，直接写个脚本提取size, python脚本如下
```python
cat > extract_hipblaslt_gemm_to_csv.py << 'PY'
#!/usr/bin/env python3
import argparse
import csv
import re
import sys
from collections import defaultdict, Counter
from pathlib import Path


def parse_args():
    p = argparse.ArgumentParser(
        description="Extract hipblaslt-bench GEMM M/N/K from log and export report+tuning CSV."
    )
    p.add_argument("-i", "--input", default="hipblas.txt", help="Input log file")
    p.add_argument("--detail-out", default="hipblaslt_detail.csv", help="Detail CSV output")
    p.add_argument("--report-out", default="hipblaslt_report.csv", help="Report CSV output (N_range)")
    p.add_argument("--tuning-out", default="hipblaslt_tuning.csv", help="Tuning CSV output (keep all N + count)")
    p.add_argument("--tp", default="", help="Optional tp value, e.g. 1")
    return p.parse_args()


def extract_records(text: str):
    cmd_pattern = re.compile(r"hipblaslt-bench\b.*?(?=hipblaslt-bench\b|$)", re.S)
    m_pat = re.compile(r"(?:^|\s)-m\s+(\d+)\b")
    n_pat = re.compile(r"(?:^|\s)-n\s+(\d+)\b")
    k_pat = re.compile(r"(?:^|\s)-k\s+(\d+)\b")
    ta_pat = re.compile(r"--transA\s+([TN])\b", re.I)
    tb_pat = re.compile(r"--transB\s+([TN])\b", re.I)

    records = []
    for block in cmd_pattern.findall(text):
        mm = m_pat.search(block)
        nn = n_pat.search(block)
        kk = k_pat.search(block)
        if not (mm and nn and kk):
            continue

        ta = ta_pat.search(block)
        tb = tb_pat.search(block)
        calc_type = f"{ta.group(1).upper()}{tb.group(1).upper()}" if (ta and tb) else ""

        records.append({
            "计算类型": calc_type,
            "M": int(mm.group(1)),
            "N": int(nn.group(1)),
            "K": int(kk.group(1)),
        })
    return records


def write_detail_csv(records, output_path: Path, tp_value: str):
    with output_path.open("w", newline="", encoding="utf-8-sig") as f:
        w = csv.writer(f)
        w.writerow(["序号", "计算类型", "M", "N", "K", "tp"])
        for i, r in enumerate(records, start=1):
            w.writerow([i, r["计算类型"], r["M"], r["N"], r["K"], tp_value])


def write_report_csv(records, output_path: Path, tp_value: str):
    # 报告版：N_range，不要次数
    groups = defaultdict(list)
    for r in records:
        key = (r["计算类型"], r["M"], r["K"], tp_value)
        groups[key].append(r["N"])

    with output_path.open("w", newline="", encoding="utf-8-sig") as f:
        w = csv.writer(f)
        w.writerow(["hipblaslt gemm size", "", "", "", ""])
        w.writerow(["计算类型", "M", "N_range", "K", "tp"])

        first = True
        for (calc_type, m, k, tp), n_list in sorted(
            groups.items(), key=lambda x: (x[0][0], x[0][1], x[0][2], x[0][3])
        ):
            n_min = min(n_list)
            n_max = max(n_list)
            n_range = f"{n_min}-{n_max}" if n_min != n_max else str(n_min)
            tp_cell = tp if first else ""
            w.writerow([calc_type, m, n_range, k, tp_cell])
            first = False


def write_tuning_csv(records, output_path: Path, tp_value: str):
    # 调优版：保留所有具体N，并统计次数
    counter = Counter((r["计算类型"], r["M"], r["N"], r["K"], tp_value) for r in records)

    with output_path.open("w", newline="", encoding="utf-8-sig") as f:
        w = csv.writer(f)
        w.writerow(["hipblaslt tuning sizes", "", "", "", "", ""])
        w.writerow(["计算类型", "M", "N", "K", "tp", "次数"])

        for (calc_type, m, n, k, tp), cnt in sorted(
            counter.items(), key=lambda x: (x[0][0], x[0][1], x[0][2], x[0][3], x[0][4])
        ):
            w.writerow([calc_type, m, n, k, tp, cnt])


def main():
    args = parse_args()
    in_path = Path(args.input)
    if not in_path.exists():
        print(f"[ERROR] Input file not found: {in_path}", file=sys.stderr)
        return 1

    text = in_path.read_text(encoding="utf-8", errors="ignore")
    records = extract_records(text)
    if not records:
        print("[ERROR] No valid hipblaslt-bench GEMM records found.", file=sys.stderr)
        return 2

    detail_out = Path(args.detail_out)
    report_out = Path(args.report_out)
    tuning_out = Path(args.tuning_out)

    write_detail_csv(records, detail_out, args.tp)
    write_report_csv(records, report_out, args.tp)
    write_tuning_csv(records, tuning_out, args.tp)

    print(f"[OK] Parsed records: {len(records)}")
    print(f"[OK] Detail CSV:  {detail_out}")
    print(f"[OK] Report CSV:  {report_out}")
    print(f"[OK] Tuning CSV:  {tuning_out}")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
PY
```

python3 extract_hipblaslt_gemm_to_csv.py -i hipblas.txt --tp 1

<image token="LBmqbR8u9oGGFEx0VP8cACrnnFj" width="1412" height="496" align="center"/>

## 1.2 hipblaslt gemm size--结论

<lark-table rows="12" cols="5" header-row="true" column-widths="100,100,100,100,100">

  <lark-tr>
    <lark-td colspan="5">
      **hipblaslt gemm size** {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      计算类型 {align="center"}
    </lark-td>
    <lark-td>
      M {align="center"}
    </lark-td>
    <lark-td>
      N_range {align="center"}
    </lark-td>
    <lark-td>
      K {align="center"}
    </lark-td>
    <lark-td>
      tp {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      128 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
    <lark-td rowspan="10">
      1 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      2624 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      4096 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      2048 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      4096 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      2048 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      12288 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      16384 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      16384 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      2048 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      24576 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      28672 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      512 {align="center"}
    </lark-td>
  </lark-tr>
</lark-table>

# 2 hipblaslt tuning

## 2.1 log去重

就还是用前面生成的那个log文件，然后用下面的脚本对文件进行去重，最终生成一个sample.sh文件。
```python
cat > extract_hipblaslt_dedup_from_log.py << 'PY'
#!/usr/bin/env python3
import argparse
import re
from pathlib import Path


def pick(pattern: str, text: str) -> str:
    m = re.search(pattern, text, flags=re.IGNORECASE)
    return m.group(1) if m else ""


def normalize_spaces(s: str) -> str:
    return re.sub(r"\s+", " ", s).strip()


def extract_candidates(raw_text: str):
    # 折叠空白，处理“整行粘连/换行丢失”
    flat = normalize_spaces(raw_text)
    # 从 hipblaslt-bench 开始，截到 --activation_type XXX 结束
    pattern = re.compile(r"(hipblaslt-bench\b.*?--activation_type\s+\S+)", re.IGNORECASE)
    return [normalize_spaces(x) for x in pattern.findall(flat)]


def build_key(cmd: str):
    # 用关键字段做去重键，避免空格差异导致重复
    return (
        pick(r"--api_method\s+(\S+)", cmd),
        pick(r"(?:^|\s)-m\s+(\d+)", cmd),
        pick(r"(?:^|\s)-n\s+(\d+)", cmd),
        pick(r"(?:^|\s)-k\s+(\d+)", cmd),
        pick(r"--lda\s+(\d+)", cmd),
        pick(r"--ldb\s+(\d+)", cmd),
        pick(r"--ldc\s+(\d+)", cmd),
        pick(r"--ldd\s+(\d+)", cmd),
        pick(r"--stride_a\s+(\d+)", cmd),
        pick(r"--stride_b\s+(\d+)", cmd),
        pick(r"--stride_c\s+(\d+)", cmd),
        pick(r"--stride_d\s+(\d+)", cmd),
        pick(r"--alpha\s+(\S+)", cmd),
        pick(r"--beta\s+(\S+)", cmd),
        pick(r"--transA\s+([TN])", cmd).upper(),
        pick(r"--transB\s+([TN])", cmd).upper(),
        pick(r"--batch_count\s+(\d+)", cmd),
        pick(r"--scaleA\s+(\S+)", cmd),
        pick(r"--scaleB\s+(\S+)", cmd),
        pick(r"--a_type\s+(\S+)", cmd),
        pick(r"--b_type\s+(\S+)", cmd),
        pick(r"--c_type\s+(\S+)", cmd),
        pick(r"--d_type\s+(\S+)", cmd),
        pick(r"--scale_type\s+(\S+)", cmd),
        pick(r"--bias_type\s+(\S+)", cmd),
        pick(r"--compute_type\s+(\S+)", cmd),
        pick(r"--activation_type\s+(\S+)", cmd),
    )


def is_valid_key(key):
    # 关键字段至少要有这些，否则视为截断命令
    required_idx = [0, 1, 2, 3, 14, 15, 19, 20, 23, 25, 26]
    return all(key[i] for i in required_idx)


def main():
    parser = argparse.ArgumentParser(
        description="从原始混合日志提取并去重 hipblaslt-bench 完整命令。"
    )
    parser.add_argument("-i", "--input", required=True, help="原始日志文件路径")
    parser.add_argument("-o", "--output", default="sample_dedup.sh", help="输出去重后的命令文件")
    args = parser.parse_args()

    input_path = Path(args.input)
    output_path = Path(args.output)

    if not input_path.exists():
        raise FileNotFoundError(f"Input file not found: {input_path}")

    raw_text = input_path.read_text(encoding="utf-8", errors="ignore")
    candidates = extract_candidates(raw_text)

    seen = set()
    uniq = []
    invalid = 0

    for cmd in candidates:
        key = build_key(cmd)
        if not is_valid_key(key):
            invalid += 1
            continue
        if key in seen:
            continue
        seen.add(key)
        uniq.append(cmd)

    output_path.write_text("\n".join(uniq) + ("\n" if uniq else ""), encoding="utf-8")

    print(f"[OK] candidates: {len(candidates)}")
    print(f"[OK] invalid/skipped: {invalid}")
    print(f"[OK] unique: {len(uniq)}")
    print(f"[OK] output: {output_path}")


if __name__ == "__main__":
    main()
PY
```

python3 extract_hipblaslt_dedup_from_log.py -i hipblas.txt -o sample.sh

## 2.2 生成config文件
```bash
chmod +x /opt/dtk-26.04-rc4-ubuntu20.04/lib/hipblaslt/benchmark_tool/hipblaslt-bench
chmod +x /opt/dtk-26.04-rc4-ubuntu20.04/lib/hipblaslt/benchmark_tool/hipblaslt_gentest.py
export PATH=/opt/dtk-26.04-rc4-ubuntu20.04/lib/hipblaslt/benchmark_tool:$PATH
HIPBLASLT_TUNING_FILE=hipblaslt.config   ./sample.sh
```

最终生成的config文件内容大约是

<view type="1">

  <file token="MNHZbMsQwofcskx4bSFcSseZnUh" name="hipblaslt.config"/>

</view>

```bash
root@NMZ36:/work/chenhw5# cat hipblaslt.config
Git Version: a6254b89-dirty
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,28672,512,512,1,512,14680064,0,512,262144,28672,14680064,28672,14680064,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,150538,99.8581,5223,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,2624,128,6144,1,6144,16121856,0,6144,786432,2624,335872,2624,335872,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,96525.9,42.7574,5195,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,128,128,6144,1,6144,786432,0,6144,786432,128,16384,128,16384,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,17115.7,11.7627,5162,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,4096,128,2048,1,2048,8388608,0,2048,262144,4096,524288,4096,524288,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,91920.2,23.3625,5172,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,16384,128,2048,1,2048,33554432,0,2048,262144,16384,2097152,16384,2097152,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,169225,50.7603,5223,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,6144,128,16384,1,16384,100663296,0,16384,2097152,6144,786432,6144,786432,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,154902,166.362,5214,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,24576,128,6144,1,6144,150994944,0,6144,786432,24576,3145728,24576,3145728,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,231391,167.053,5233,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,6144,128,12288,1,12288,75497472,0,12288,1572864,6144,786432,6144,786432,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,190864,101.262,5202,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,4096,128,6144,1,6144,25165824,0,6144,786432,4096,524288,4096,524288,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,141099,45.6591,5211,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,6144,128,2048,1,2048,12582912,0,2048,262144,6144,786432,6144,786432,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,110864,29.0557,5202,gfx938:sramecc+:xnack-,72
    transA,transB,grouped_gemm,batch_count,m,n,k,alpha,lda,stride_a,beta,ldb,stride_b,ldc,stride_c,ldd,stride_d,a_type,b_type,c_type,d_type,compute_type,scaleA,scaleB,scaleC,scaleD,amaxD,activation_type,bias_vector,bias_type,rotating_buffer,hipblaslt-Gflops,us,solution_index,gcnArchName,CUs
    T,N,0,1,2624,120,6144,1,6144,16121856,0,6144,737280,2624,314880,2624,314880,i8_r,i8_r,i32_r,bf16_r,i32_r,2,2,0,0,0,none,0,f32_r,512,89438,43.2618,5195,gfx938:sramecc+:xnack-,72
```

## 2.3 利用config文件重跑服务
```bash
export HIPBLASLT_TUNING_OVERRIDE_FILE=hipblaslt.config
export HIPBLASLT_LOG_LEVEL=4 #使用环境变量 HIPBLASLT_LOG_LEVEL=4,可观察效果是否生效

vllm serve /models/GLM-5-W8A8  \
  --disable-log-requests \
  -q slimquant_marlin \
  --trust-remote-code \
  -dp 8 \
  -tp 1 \
  --enable-expert-parallel \
  --disable-custom-all-reduce \
  --dtype bfloat16 \
  --enable-chunked-prefill \
  --max-model-len 50000 \
   --max-num-batched-tokens 128 \
  --max-num-seqs 32 \
  --enable-prefix-caching \
  --block-size 64 \
  --gpu-memory-utilization 0.88 \
  --kv-cache-dtype fp8_ds_mla \
  -cc '{"inductor_compile_config":{"combo_kernels": false}}' \
--speculative_config '{"method":"mtp","num_speculative_tokens":3, "quantization": "slimquant_marlin"}'  
```

会有这种log
```bash
[2026-04-18 08:58:34][HIPBLASLT][600][Trace][rocblaslt_matmul] A=0x7f4db5800000 Adesc=[type=R_8I rows=512 cols=28672 ld=512] B=0x7f4619280000 Bdesc=[type=R_8I rows=512 cols=512 ld=512] C=0x1 Cdesc=[type=R_32I rows=28672 cols=512 ld=28672] D=0x7f4565a00000 Ddesc=[type=R_16BF rows=28672 cols=512 ld=28672] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55d4db876f80
[2026-04-18 08:58:34][HIPBLASLT][600][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 08:58:34][HIPBLASLT][600][Info][problem_override_from_file] Find solution with index: 5223
[2026-04-18 08:58:34][HIPBLASLT][600][Trace][rocblaslt_matmul] A=0x7f4d5e200000 Adesc=[type=R_8I rows=512 cols=28672 ld=512] B=0x7f4619280000 Bdesc=[type=R_8I rows=512 cols=512 ld=512] C=0x1 Cdesc=[type=R_32I rows=28672 cols=512 ld=28672] D=0x7f4563a00000 Ddesc=[type=R_16BF rows=28672 cols=512 ld=28672] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55d4db876f80
[2026-04-18 08:58:34][HIPBLASLT][600][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 08:58:34][HIPBLASLT][600][Info][problem_override_from_file] Find solution with index: 5223
[2026-04-18 08:58:34][HIPBLASLT][600][Trace][rocblaslt_matmul] A=0x7f4d07200000 Adesc=[type=R_8I rows=512 cols=28672 ld=512] B=0x7f4619280000 Bdesc=[type=R_8I rows=512 cols=512 ld=512] C=0x1 Cdesc=[type=R_32I rows=28672 cols=512 ld=28672] D=0x7f4561a00000 Ddesc=[type=R_16BF rows=28672 cols=512 ld=28672] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55d4db876f80
[2026-04-18 08:58:34][HIPBLASLT][600][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 08:58:34][HIPBLASLT][600][Info][problem_override_from_file] Find solution with index: 5223


[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf69e00000 Adesc=[type=R_8I rows=6144 cols=2624 ld=6144] B=0x7fbe97a00000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=2624 cols=32 ld=2624] D=0x7fbe927c0000 Ddesc=[type=R_16BF rows=2624 cols=32 ld=2624] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5162
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcee0920000 Adesc=[type=R_8I rows=6144 cols=128 ld=6144] B=0x7fbe97a30000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=128 cols=32 ld=128] D=0x7fcee09f1000 Ddesc=[type=R_16BF rows=128 cols=32 ld=128] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5218
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf9f700000 Adesc=[type=R_8I rows=2048 cols=4096 ld=2048] B=0x7fcee03e1a00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=4096 cols=32 ld=4096] D=0x7fcc79ce1c00 Ddesc=[type=R_16BF rows=4096 cols=32 ld=4096] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5219
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcd86a00000 Adesc=[type=R_8I rows=2048 cols=16384 ld=2048] B=0x7fcebe3e1c00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=16384 cols=32 ld=16384] D=0x7fbe92040000 Ddesc=[type=R_16BF rows=16384 cols=32 ld=16384] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5166
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcd80600000 Adesc=[type=R_8I rows=16384 cols=6144 ld=16384] B=0x7fbe92640000 Bdesc=[type=R_8I rows=16384 cols=32 ld=16384] C=0x1 Cdesc=[type=R_32I rows=6144 cols=32 ld=6144] D=0x7fbe97a00000 Ddesc=[type=R_16BF rows=6144 cols=32 ld=6144] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5177
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcea5c00000 Adesc=[type=R_8I rows=6144 cols=4096 ld=6144] B=0x7fbe926a0000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=4096 cols=32 ld=4096] D=0x7fbe926d0000 Ddesc=[type=R_16BF rows=4096 cols=32 ld=4096] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5193
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf9aa00000 Adesc=[type=R_8I rows=2048 cols=6144 ld=2048] B=0x7fcebe3e1c00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=6144 cols=32 ld=6144] D=0x7fbe91a00000 Ddesc=[type=R_16BF rows=6144 cols=32 ld=6144] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5218
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf69e00000 Adesc=[type=R_8I rows=6144 cols=2624 ld=6144] B=0x7fbe97a29000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=2624 cols=32 ld=2624] D=0x7fbe97a00000 Ddesc=[type=R_16BF rows=2624 cols=32 ld=2624] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5162
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcee0920000 Adesc=[type=R_8I rows=6144 cols=128 ld=6144] B=0x7fcc79ce1c00 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=128 cols=32 ld=128] D=0x7fcee09f1000 Ddesc=[type=R_16BF rows=128 cols=32 ld=128] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5218
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf9f700000 Adesc=[type=R_8I rows=2048 cols=4096 ld=2048] B=0x7fcee03e1a00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=4096 cols=32 ld=4096] D=0x7fcebe200000 Ddesc=[type=R_16BF rows=4096 cols=32 ld=4096] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5219
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcd86a00000 Adesc=[type=R_8I rows=2048 cols=16384 ld=2048] B=0x7fcebe3e1c00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=16384 cols=32 ld=16384] D=0x7fbe92640000 Ddesc=[type=R_16BF rows=16384 cols=32 ld=16384] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5166
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcd80600000 Adesc=[type=R_8I rows=16384 cols=6144 ld=16384] B=0x7fbe91a00000 Bdesc=[type=R_8I rows=16384 cols=32 ld=16384] C=0x1 Cdesc=[type=R_32I rows=6144 cols=32 ld=6144] D=0x7fbe97a00000 Ddesc=[type=R_16BF rows=6144 cols=32 ld=6144] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5177
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcea5c00000 Adesc=[type=R_8I rows=6144 cols=4096 ld=6144] B=0x7fbe927c0000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=4096 cols=32 ld=4096] D=0x7fbe91aa0000 Ddesc=[type=R_16BF rows=4096 cols=32 ld=4096] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5193
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf9aa00000 Adesc=[type=R_8I rows=2048 cols=6144 ld=2048] B=0x7fcebe3e1c00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=6144 cols=32 ld=6144] D=0x7fbe92040000 Ddesc=[type=R_16BF rows=6144 cols=32 ld=6144] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5218
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf69e00000 Adesc=[type=R_8I rows=6144 cols=2624 ld=6144] B=0x7fbe91a00000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=2624 cols=32 ld=2624] D=0x7fbe927c0000 Ddesc=[type=R_16BF rows=2624 cols=32 ld=2624] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5162
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcee0920000 Adesc=[type=R_8I rows=6144 cols=128 ld=6144] B=0x7fbe91a30000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=128 cols=32 ld=128] D=0x7fcee09f1000 Ddesc=[type=R_16BF rows=128 cols=32 ld=128] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5218
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf9f700000 Adesc=[type=R_8I rows=2048 cols=4096 ld=2048] B=0x7fcee03e1a00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=4096 cols=32 ld=4096] D=0x7fbe97a00000 Ddesc=[type=R_16BF rows=4096 cols=32 ld=4096] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5219
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcd86a00000 Adesc=[type=R_8I rows=2048 cols=16384 ld=2048] B=0x7fcebe3e1c00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=16384 cols=32 ld=16384] D=0x7fbe92040000 Ddesc=[type=R_16BF rows=16384 cols=32 ld=16384] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5166
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcd80600000 Adesc=[type=R_8I rows=16384 cols=6144 ld=16384] B=0x7fbe92640000 Bdesc=[type=R_8I rows=16384 cols=32 ld=16384] C=0x1 Cdesc=[type=R_32I rows=6144 cols=32 ld=6144] D=0x7fbe91a00000 Ddesc=[type=R_16BF rows=6144 cols=32 ld=6144] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5177
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcea5c00000 Adesc=[type=R_8I rows=6144 cols=4096 ld=6144] B=0x7fbe92640000 Bdesc=[type=R_8I rows=6144 cols=32 ld=6144] C=0x1 Cdesc=[type=R_32I rows=4096 cols=32 ld=4096] D=0x7fbe92670000 Ddesc=[type=R_16BF rows=4096 cols=32 ld=4096] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
[2026-04-18 09:03:27][HIPBLASLT][601][Info][hipblasLtMatmulAlgoGetHeuristic] HIPBLASLT_TUNING_OVERRIDE_FILE is the correct setting.
[2026-04-18 09:03:27][HIPBLASLT][601][Info][problem_override_from_file] Find solution with index: 5193
[2026-04-18 09:03:27][HIPBLASLT][601][Trace][rocblaslt_matmul] A=0x7fcf9aa00000 Adesc=[type=R_8I rows=2048 cols=6144 ld=2048] B=0x7fcebe3e1c00 Bdesc=[type=R_8I rows=2048 cols=32 ld=2048] C=0x1 Cdesc=[type=R_32I rows=6144 cols=32 ld=6144] D=0x7fbe926b0000 Ddesc=[type=R_16BF rows=6144 cols=32 ld=6144] computeDesc=[computeType=COMPUTE_32I scaleType=R_32F transA=OP_T transB=OP_N epilogue=EPILOGUE_DEFAULT biasPointer=0x0] workSpace=0 workSpaceSizeInBytes=0 alpha=0 beta=0 stream=0x55e219cb2810
(EngineCore_DP0 pid=594) INFO 04-18 09:03:27 [core.py:273] init engine (profile, create kv cache, warmup model) took 282.95 seconds

```

# 3 rocmblas gemm size

## 3.1 抓log

容器重启一下
```sql
export ROCBLAS_LAYER=3 
export TENSILE_DB=0x8000 
vllm serve /models/GLM-5-W8A8  \
  --disable-log-requests \
  -q slimquant_marlin \
  --trust-remote-code \
  -dp 8 \
  -tp 1 \
  --enable-expert-parallel \
  --disable-custom-all-reduce \
  --dtype bfloat16 \
  --enable-chunked-prefill \
  --max-model-len 50000 \
   --max-num-batched-tokens 128 \
  --max-num-seqs 32 \
  --enable-prefix-caching \
  --block-size 64 \
  --gpu-memory-utilization 0.88 \
  --kv-cache-dtype fp8_ds_mla \
  -cc '{"inductor_compile_config":{"combo_kernels": false}}' \
--speculative_config '{"method":"mtp","num_speculative_tokens":3, "quantization": "slimquant_marlin"}'  \
--api-server-count 1  >&rocmblas.txt
```

```bash
vllm bench serve --dataset-name random --model /models/GLM-5-W8A8 \
--num-prompts 1 \
--random-input-len 32 \
--random-output-len 1 \
--random-range-ratio 0.0 \
--endpoint /v1/completions \
--tokenizer /models/GLM-5-W8A8 \
--temperature 0.0 \
--ignore-eos \
--host 12.12.12.55
```

```python
cat > extract_rocblas_gemm_to_csv.py << 'PY'
#!/usr/bin/env python3
import argparse
import csv
import re
import sys
from collections import defaultdict
from pathlib import Path


def parse_args():
    p = argparse.ArgumentParser(
        description="Extract rocBLAS GEMM M/N/K from log and export CSV."
    )
    p.add_argument("-i", "--input", default="rocblas.txt", help="Input log file")
    p.add_argument("--detail-out", default="rocblas_detail.csv", help="Detail CSV output")
    p.add_argument("--summary-out", default="rocblas_summary.csv", help="Summary CSV output")
    p.add_argument("--tp", default="", help="Optional tp value, e.g. 1")
    return p.parse_args()


def extract_from_bench_line(line: str):
    ta = re.search(r"--transposeA\s+([TN])\b", line, re.I)
    tb = re.search(r"--transposeB\s+([TN])\b", line, re.I)
    m = re.search(r"(?:^|\s)-m\s+(\d+)\b", line)
    n = re.search(r"(?:^|\s)-n\s+(\d+)\b", line)
    k = re.search(r"(?:^|\s)-k\s+(\d+)\b", line)
    if not (ta and tb and m and n and k):
        return None
    return {
        "计算类型": f"{ta.group(1).upper()}{tb.group(1).upper()}",
        "M": int(m.group(1)),
        "N": int(n.group(1)),
        "K": int(k.group(1)),
    }


def extract_from_api_line(line: str):
    m = re.search(
        r"rocblas_gemm(?:_strided_batched)?_ex,([TN]),([TN]),(\d+),(\d+),(\d+),",
        line,
        re.I,
    )
    if not m:
        return None
    return {
        "计算类型": f"{m.group(1).upper()}{m.group(2).upper()}",
        "M": int(m.group(3)),
        "N": int(m.group(4)),
        "K": int(m.group(5)),
    }


def extract_records(text: str):
    records = []
    for line in text.splitlines():
        line = line.strip()
        if not line:
            continue

        rec = None
        if "rocblas-bench" in line:
            rec = extract_from_bench_line(line)
        elif "rocblas_gemm_ex," in line or "rocblas_gemm_strided_batched_ex," in line:
            rec = extract_from_api_line(line)

        if rec is not None:
            records.append(rec)
    return records


def write_detail_csv(records, output_path: Path, tp_value: str):
    with output_path.open("w", newline="", encoding="utf-8-sig") as f:
        w = csv.writer(f)
        w.writerow(["序号", "计算类型", "M", "N", "K", "tp"])
        for i, r in enumerate(records, start=1):
            w.writerow([i, r["计算类型"], r["M"], r["N"], r["K"], tp_value])


def write_summary_csv(records, output_path: Path, tp_value: str):
    # 保留全部 M/K 组合，只把 N 做范围聚合
    groups = defaultdict(list)
    for r in records:
        key = (r["计算类型"], r["M"], r["K"], tp_value)
        groups[key].append(r["N"])

    with output_path.open("w", newline="", encoding="utf-8-sig") as f:
        w = csv.writer(f)
        # 标题行
        w.writerow(["rocmblas gemm size", "", "", "", ""])
        # 表头
        w.writerow(["计算类型", "M", "N_range", "K", "tp"])

        # 数据：tp 只在第一行显示，后续留空（视觉上接近合并）
        first = True
        for (calc_type, m, k, tp), n_list in sorted(
            groups.items(), key=lambda x: (x[0][0], x[0][1], x[0][2], x[0][3])
        ):
            n_min = min(n_list)
            n_max = max(n_list)
            n_range = f"{n_min}-{n_max}" if n_min != n_max else str(n_min)

            tp_cell = tp if first else ""
            w.writerow([calc_type, m, n_range, k, tp_cell])
            first = False


def main():
    args = parse_args()
    in_path = Path(args.input)
    if not in_path.exists():
        print(f"[ERROR] Input file not found: {in_path}", file=sys.stderr)
        return 1

    text = in_path.read_text(encoding="utf-8", errors="ignore")
    records = extract_records(text)
    if not records:
        print("[ERROR] No valid rocBLAS GEMM records found.", file=sys.stderr)
        return 2

    detail_out = Path(args.detail_out)
    summary_out = Path(args.summary_out)
    write_detail_csv(records, detail_out, args.tp)
    write_summary_csv(records, summary_out, args.tp)

    print(f"[OK] Parsed records: {len(records)}")
    print(f"[OK] Detail CSV: {detail_out}")
    print(f"[OK] Summary CSV: {summary_out}")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
PY
```

python3 extract_rocblas_gemm_to_csv.py -i rocmblas.txt --tp 1

<image token="ZnffbUhKQobV0sxFekccNhbMnNg" width="1443" height="897" align="center"/>

## 3.2 rocmblas gemm size--结论

<lark-table rows="8" cols="5" column-widths="100,100,100,100,100">

  <lark-tr>
    <lark-td colspan="5">
      rocmblas gemm size {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      计算类型 {align="center"}
    </lark-td>
    <lark-td>
      M {align="center"}
    </lark-td>
    <lark-td>
      N_range {align="center"}
    </lark-td>
    <lark-td>
      K {align="center"}
    </lark-td>
    <lark-td>
      tp {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      NN {align="center"}
    </lark-td>
    <lark-td>
      32 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
    <lark-td rowspan="6">
      1 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      NN {align="center"}
    </lark-td>
    <lark-td>
      256 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      512 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      NN {align="center"}
    </lark-td>
    <lark-td>
      256 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      512 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      192 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
    <lark-td>
      1-16384 {align="center"}
    </lark-td>
    <lark-td>
      12288 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TN {align="center"}
    </lark-td>
    <lark-td>
      154880 {align="center"}
    </lark-td>
    <lark-td>
      1-1024 {align="center"}
    </lark-td>
    <lark-td>
      6144 {align="center"}
    </lark-td>
  </lark-tr>
</lark-table>

<text color="red">**下面的是郭冰洁的log**</text>

root@NMZ43:/work/chenhw5# cat rocblas_summary.csv

计算类型,M,N_min,N_max,N_range,K,tp,次数

<text color="red">**NN,16,16,16,16,16,1,1                这个我没有**</text>

TN,32,1,128,1-128,6144,1,32

<text color="red">**TN,128,1,4096,1-4096,128,1,60   这个我没有**</text>

TN,256,1,128,1-128,512,1,32

TN,256,1,128,1-128,6144,1,32

TN,512,1,128,1-128,192,1,32

TN,6144,1,32,1-32,12288,1,20

TN,154880,1,128,1-128,6144,1,32

# 4 rocmblas tuning

export ROCBLAS_TENSILE_LIBPATH=/home/library_gpu6_glm5_int8

<view type="1">

  <file token="TaVLbxwC8o5WjIxMD51cUFjtnFc" name="library_gpu6_glm5_int8.zip"/>

</view>

上面是郭冰洁的，但是不一定能用，需要重新tuning，

/public/home/sglang/gbj/auto_select_tools/rocblas_layer3_size_glm5-int8_tp1/glm5_int8_ep8dp8_bs32_lmgemm_size.log

这是log，把他的log复制过来，然后在自己的环境中进行tuning，

用研发集群20节点。

[chenhw5@nmz20 ~]$ docker load -i vllm_015_glm5_ep_20260416_dev.tar

5ca673e8e92b: Loading layer [==================================================>]  1.724GB/1.724GB

a1f146e06a2b: Loading layer [==================================================>]  6.913GB/6.913GB

6086b41a6c15: Loading layer [==================================================>]  4.563GB/4.563GB

9fdc19ac6eac: Loading layer [==================================================>]  37.27GB/37.27GB

Loaded image: vllm_015_glm5_ep_20260416:dev

```bash
docker run --ipc=host --shm-size=256g --network=host --name=vllm_015_glm5_ep_20260421_chenhw5 \
--privileged \
--device=/dev/kfd \
--device=/dev/dri \
--group-add video \
--cap-add=SYS_PTRACE \
--security-opt seccomp=unconfined \
-v  /public/home/chenhw5/:/volume \
-v /public/opendas/DL_DATA/llm-models:/llm-models:ro \
-v /public5/dtk:/dtk:ro \
-v /module2:/module \
-v /opt/hyhal:/opt/hyhal:ro \
-v /usr/local/hyhal:/usr/local/hyhal:ro \
--ulimit stack=-1:-1 \
--ulimit memlock=-1:-1 \
-itd vllm_015_glm5_ep_20260416:dev bash
```


docker exec -it  vllm_015_glm5_ep_20260421_chenhw5  bash 

进入 
```bash
root@nmz20:/volume# cd /opt/dtk/lib/rocblas/auto_select_test/auto_select_tools

cd configs

vim tuning_args.txt

```

修改成下面的形式
```markdown
root@nmz20:/opt/dtk/lib/rocblas/auto_select_test/auto_select_tools/configs# cat tuning_args.txt
###########################
#      选优参数配置文件    #
###########################

# -------------------------
# 基础模式设置
# -------------------------
mode=user                      # 可选：user / developer
source=layer3                     # 数据来源：layer3 / layer4 / csv
hardware_type=NMZ               # 平台类型：ZF / ZF2 / KM / KME / ECO / BW / NMZ
strategy=common                # 配置生成策略：common / quick # user模式下仅支持common
gpu_ids=0 1 2 3 4 5 6 7        # 可以指定多个GPU编号

# -------------------------
# 其他设置
# -------------------------
should_clean=0                 # 是否删除optimization_configs和optimization_artifacts目录下内容,0表示不删除 # 注意：删除之前请确保备份结果配置文件
chunk_size=10                  # 每张卡每次选优的size数量，size较少时可按需调整
tune_warmup_times=2            # 选优预热次数
tune_loop_times=4              # 选优循环次数
bench_warmup_times=2           # 测试预热次数
bench_loop_times=10            # 测试循环次数
test_times=2                   # 测试轮次
enable_bestkernel_csv=0        # 是否启用“读取bestkernel CSV + MT校验过滤kernel”，0禁用(默认) / 1启用
cu_count=72                     # CU 数量，用于生成配置文件名称（如：72 表示生成 _72cu 后缀的文件名；设置为 0 则不添加 cu 后缀）

# -------------------------
# 配置更新设置
# -------------------------
update_target_config=0         # 是否将生成的配置文件更新到目标包的配置目录，0表示不更新，1表示更新
enable_mk_n_threshold=0        # 是否启用MK-N阈值检查（当同一MK组合的N值数量>=30时自动展开），0表示禁用，1表示启用


```

然后把包含size的日志文件放到这个目录

root@nmz20:/opt/dtk/lib/rocblas/auto_select_test/auto_select_tools/rocblas_layer3_size#

glm5_int8_ep8dp8_bs32_lmgemm_size.log

root@nmz20:/opt/dtk/lib/rocblas/auto_select_test/auto_select_tools/rocblas_layer3_size# **cp /volume/glm5_int8_ep8dp8_bs32_lmgemm_size.log  ./**

然后
```bash
# 1. 进入工具目录
cd /opt/dtk/lib/rocblas/auto_select_test/auto_select_tools

# 2. 加载DTK环境（根据实际DTK版本路径调整）
source /opt/dtk/env.sh

# 3. （可选）如果需要链接测试包
# export LD_LIBRARY_PATH=rocblas-install/lib:$LD_LIBRARY_PATH

# 4. 执行选优脚本（建议保存日志）
python3 Auto_tuning.py 2>&1 | tee tuning.log
```

将生成的拷贝出来，

 cp -drf  /opt/dtk-26.04-rc4-ubuntu20.04/lib/rocblas/auto_select_test/auto_select_tools/optimization_configs/new/config/library_gpu6  /volume/library_gpu6_glm5_int8_chenhw5

然后一会执行命令的时候，用

**export ROCBLAS_TENSILE_LIBPATH=/volume/library_gpu6_glm5_int8_chenhw5**

# 5 性能对比测试

/public/opendas/DL_DATA/llm-models/vllm-w8a8-models 拷贝到了/module2，容器内是/module

## 5.1 未tuning前的ep8 docoder性能

先测一下，没有修改tuning的性能，
```sql
vllm serve /module/GLM-5-W8A8  \
  --disable-log-requests \
  -q slimquant_marlin \
  --trust-remote-code \
  -dp 8 \
  -tp 1 \
  --enable-expert-parallel \
  --disable-custom-all-reduce \
  --dtype bfloat16 \
  --enable-chunked-prefill \
  --max-model-len 50000 \
   --max-num-batched-tokens 128 \
  --max-num-seqs 32 \
  --enable-prefix-caching \
  --block-size 64 \
  --gpu-memory-utilization 0.88 \
  --kv-cache-dtype fp8_ds_mla \
  -cc '{"inductor_compile_config":{"combo_kernels": false}}' \
--speculative_config '{"method":"mtp","num_speculative_tokens":3, "quantization": "slimquant_marlin"}'  
```


```bash
python decode_throughput.py \
  --base-url http://10.16.6.20:8000 \
  --model /module/GLM-5-W8A8 \
  --max-input-length-k 46 \
  --max-total-kvcache-length-k 500 \
  --max-batch-size 32 \
  --dp-size 8
```

<image token="AKktbW8icoElpoxKx4ZcQACHnDg" width="1000" height="500" align="center"/>

<image token="ZuXCbgNycoUBK5xQPS9cmlLHnte" width="1000" height="500" align="center"/>

<image token="NTrabz6jqoL2FHxbXbEc56Egn3b" width="1000" height="500" align="center"/>

<image token="LodKblCKVoeMpQxvrQ6ccOconTg" width="1000" height="500" align="center"/>

<image token="DWibbOK17oYo95xvQqwcsQRAnXc" width="1000" height="500" align="center"/>

## 5.2 tuning后的EP8 docoder性能
```sql
export HIPBLASLT_TUNING_OVERRIDE_FILE=/volume/hipblaslt.config
export ROCBLAS_TENSILE_LIBPATH=/volume/library_gpu6_glm5_int8_chenhw5

vllm serve /module/GLM-5-W8A8  \
  --disable-log-requests \
  -q slimquant_marlin \
  --trust-remote-code \
  -dp 8 \
  -tp 1 \
  --enable-expert-parallel \
  --disable-custom-all-reduce \
  --dtype bfloat16 \
  --enable-chunked-prefill \
  --max-model-len 50000 \
   --max-num-batched-tokens 128 \
  --max-num-seqs 32 \
  --enable-prefix-caching \
  --block-size 64 \
  --gpu-memory-utilization 0.88 \
  --kv-cache-dtype fp8_ds_mla \
  -cc '{"inductor_compile_config":{"combo_kernels": false}}' \
  --speculative_config '{"method":"mtp","num_speculative_tokens":3, "quantization": "slimquant_marlin"}'  
```

```bash
python decode_throughput.py \
  --base-url http://10.16.6.20:8000 \
  --model /module/GLM-5-W8A8 \
  --max-input-length-k 46 \
  --max-total-kvcache-length-k 500 \
  --max-batch-size 32 \
  --dp-size 8
```


<image token="Ci7ebzS3coeHCsxdPEGcwdQqnvb" width="1000" height="500" align="center"/>

<image token="D4B9bd4yHopsE1xnDOEcoKodnVe" width="1000" height="500" align="center"/>

# 6 参考文献

[[飞书知识图谱/文档/自动化选优脚本使用说明(持续更新)--cHDsUnwd|自动化选优脚本使用说明(持续更新)]]

[[飞书知识图谱/资源/hipblaslt-bench使用说明--cnhndnRb|hipblaslt-bench使用说明]]

[[飞书知识图谱/资源/选优脚本模型使用情况汇总--c2L54nCc|选优脚本模型使用情况汇总]]

## 关联资源

- [[飞书知识图谱/文档/自动化选优脚本使用说明(持续更新)--cHDsUnwd|自动化选优脚本使用说明(持续更新)]] · [在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/PdupdKnAjo6Dh9x3OaUcHDsUnwd)
- [[飞书知识图谱/资源/hipblaslt-bench使用说明--cnhndnRb|hipblaslt-bench使用说明]] · 飞书源链接不可用
- [[飞书知识图谱/资源/选优脚本模型使用情况汇总--c2L54nCc|选优脚本模型使用情况汇总]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/GLM-5-W8A8|GLM-5-W8A8]]（model）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/MTP|MTP]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

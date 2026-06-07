---
title: 性能分析及工具
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: R94NwfNPgiud5dkSsP9cIU0ZnEg
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/R94NwfNPgiud5dkSsP9cIU0ZnEg
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 性能分析及工具

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/R94NwfNPgiud5dkSsP9cIU0ZnEg)

# prof对比工具

用于对比DCU和NV的josn结果并做汇总统计
```plaintext {wrap}
python analyze_prof_excel.py -g -c a100.bs256.qq_onetrans.pt.trace.json bw151.bs256.qq_onetrans.0421.pt.trace.json --ops --ops-mapping
```

主要功能说明：

<lark-table rows="6" cols="2" column-widths="100,495">

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
      -f
    </lark-td>
    <lark-td>
      分析单个json文件
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      -c
    </lark-td>
    <lark-td>
      对比两个json文件结果，，例如-c A.json  B.json (默认第一个是DCU，第二个是NV）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      -g
    </lark-td>
    <lark-td>
      分组统计，主要针对triton生成的带数字后缀的算子
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      --ops
    </lark-td>
    <lark-td>
      针对op级别算子进行汇总，例如gemm、relu等
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      --ops-mapping
    </lark-td>
    <lark-td>
      生成算子统计分组详细信息，例如哪些算子归到gemm，哪些归到relu
    </lark-td>
  </lark-tr>
</lark-table>

<block-ref id="DOKSd2hfVoYwIjxk9T9cKsirn7f"/>

<file token="Im0DbsCD9oHQQ9x2bx8cKkTJnab" name="analyze_prof_excel.py"/>

结果展示：

csv中维对比结果

<lark-table rows="6" cols="2" column-widths="100,495">

  <lark-tr>
    <lark-td>
      类别
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      kernX
    </lark-td>
    <lark-td>
      算子种类数量
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      cntX
    </lark-td>
    <lark-td>
      调用次数
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      TotalX
    </lark-td>
    <lark-td>
      总耗时
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      PctX
    </lark-td>
    <lark-td>
      算子模型占比
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AvgX
    </lark-td>
    <lark-td>
      算子平均耗时
    </lark-td>
  </lark-tr>
</lark-table>

<view type="2">

  <file token="IGsNb1fVKoyy4exADgRcCdvvnUg" name="compare_a100.bs256.qq_onetrans.pt.trace_vs_bw151.bs256.qq_onetrans.0421.pt.trace.csv"/>

</view>

txt中详细展示各算子具体都包含哪些kernel

<block-ref id="To0Kdp7XnowmItxsuBycSqavnDc"/>

<file token="AlREbyNYboF53wxmTWicrBihnXb" name="compare_a100.bs256.qq_onetrans.pt.trace_ops_mapping.txt"/>

<block-ref id="I92bd7MLFoBez0xHGsZcyT0tnSf"/>

<file token="BEuDb3CpPoxFp2xM2r7celyjnbc" name="compare_bw151.bs256.qq_onetrans.0421.pt.trace_ops_mapping.txt"/>

<image token="HDUNb1b8io21Nbxe8f6cfcbnnng" width="2082" height="669" align="center"/>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）

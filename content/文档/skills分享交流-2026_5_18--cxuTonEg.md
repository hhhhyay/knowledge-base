---
title: skills分享交流-2026\5\18
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: EcsWwePbKiYZzOktPUncxuTonEg
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/EcsWwePbKiYZzOktPUncxuTonEg
last_synced_at: 2026-06-07T17:07:53.393Z
---

# skills分享交流-2026\5\18

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/EcsWwePbKiYZzOktPUncxuTonEg)

<quote-container>

本文档主要介绍团队开发使用的两个skill的结构和效果，分享创建的思路。另外，也介绍了国内体验国外大模型（Gemini 3 pro、claude opus 4.6\4.7、GPT-5.5）的方法，欢迎大家交流指正。
</quote-container>

# das-compare-prof skill

<view type="1">

  <file token="ITVwbMYbtoC2GTxavITch2nqnzd" name="das-compare-prof.zip"/>

</view>

```python
scripts/
├── parse_log.py                  # 解析日志，提取性能指标
├── query_baseline.py             # 查询 askflow 知识库，获取基线
├── compare_prof.py               # 对比性能，生成表格
├── feishu_push.py                # 飞书卡片与飞书表格推送
├── feishu_long_connection_server.py  # 飞书长连接服务（处理卡片按钮）
└── visualize.py                  # 生成可视化折线图

config.py                         # 集中配置文件（App ID、Chat ID、默认值等）
SKILL.md                           # 该skill的规范文件，定义触发词、交互流程等
```

- <text color="gray" bgcolor="light-gray">当前的问题：训练性能对比需要人工解析日志、查询知识库、手动制表</text>
- <text color="gray" bgcolor="light-gray">预期的目标：自动化性能对比，一键生成报告 + 飞书推送记录</text>
- 飞书开放平台：https://open.feishu.cn/app

<callout emoji="speech_balloon" background-color="light-orange" border-color="light-orange">

用户输入 log 目录 + 模型 + 卡型

    ↓

解析日志 + 查询知识库

    ↓

生成对比报告（stdout）

    ↓

询问：是否推飞书？

    ├─ 是 → 发送卡片（带按钮：写表格/生成图片/取消）
    └─ 否 → 结束
</callout>

# das-askflow skill

<view type="1">

  <file token="HCBib4RLBoqkMWxcylTc5pMxnib" name="das-askflow.zip"/>

</view>

```python
scripts/
├── config.py                 # 全局配置（KB_ROUTER、向量权重、字段定义）
├── retriever.py              # RAG HTTP 客户端（请求、重试、评分、来源加权）
├── search.py                 # 查询构造 + 路由 + 置信度分级 + 格式渲染
└── query_retrieval.py        # 命令行入口

references/
├── model_spec_table.md       # 推理模型-精度规格表（用于追问）
└── DAS1_8_Performance_Report.md  # DAS 1.8 宏观基准测试报告

README.md                         # 快速入门指南
SKILL.md                          # Skill 规范文件（触发词、交互流程、参数说明）
```

- <text color="gray" bgcolor="light-gray">用途：DCU 内部知识库问答专家，回答 DCU/DTK 相关技术问题、Bug 记录、性能数据、技术文档等。</text>
- <text bgcolor="light-gray">主要能力：</text>
	1. **Bug 排查** - 历史报错、已知限制、异常处理
	2. **性能查询** - 大模型推理/训练性能数据（吞吐、延迟、TFLOPs）
	3. **技术文档** - 接口配置、使用指南、参数说明
	4. **宏观分析** - 版本对比、基准总结、整体评估

<callout emoji="speech_balloon" background-color="light-orange" border-color="light-orange">

用户输入问题

    ↓

Step 1：宏观查询路由？

    ├─ 是 → 读取 DAS1_8_Performance_Report.md → 输出总结
    └─ 否 ↓

Step 2：信息确认与模型精度匹配

    ↓

Step 3：意图识别（train/infer）

    ↓

Step 4：分区路由（bug_list/perf_table/tech_docs）

    ↓

Step 5：调用 RAG API 检索

    ↓

Step 6：统一输出格式（结论+详细说明+参考来源）
</callout>

# 国外AI大模型使用方法

<quote-container>

推荐去<text bgcolor="light-red">咸鱼\淘宝</text>搜索相关的大模型登录方法，基本都有教程。

推荐使用类似谷歌账号密码登录，稳定且延迟低，不推荐使用中转站登录，延迟高还费token。
</quote-container>

## 网页端登录

### Gemini 

- 网页地址**:**[**gemini.google.com**](https%3A%2F%2Fgemini.google.com%2Fapp%3Futm_source%3Ddeepmind.google%26utm_medium%3Dreferral%26utm_campaign%3Dgdm%26utm_content%3D)
- 谷歌账号密码登录
- 普通用户每天可以无限使用快速和思考模式；pro模式使用次数有限。
- 升级为Google AI PRO会员后，pro模式够开发强度使用，基本不限制。

<image token="IlOTbMQd1oX3kcxq6rGc5PDCnpg" width="463" height="331" align="center"/>

### Claude 

- 网页地址**：****https://claude.ai/new**
- 谷歌账号密码登录
- 普通用户大概每天可以问2-4次复杂的问题，用的是Sonnet 4.6模型。

<image token="Cf80bQ6LboJy8Px8SJEcyJusnLI" width="1018" height="629" align="center"/>

## 桌面端 AI IDE 接入

### Cursor

- 下载地址：https://cursor.com/cn/download

<image token="SzUDb1zamoIK2vxWpIXcLxB8nHd" width="426" height="365" align="center"/>

### Windsurf

- 下载地址：https://windsurf.com/download/editor

<grid cols="2">

  <column width="50">
    <image token="P5J8bSWuHoCIXoxZzFPc60f7nzc" width="399" height="388" align="center"/>

  </column>
  <column width="50">
    <image token="EtuPbjsBuovYtUxn4SRcvANjnOb" width="380" height="400" align="center"/>

  </column>

</grid>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

---
title: 云一-异构 模型权重 cos上传，下载方式
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: LXK9djQg0olfucxlIAncJFSznJd
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/docx/LXK9djQg0olfucxlIAncJFSznJd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 云一-异构 模型权重 cos上传，下载方式

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/docx/LXK9djQg0olfucxlIAncJFSznJd)

# 注意事项

1. <text bgcolor="light-yellow">**如果上传的是模型权重，请联系下把模型名和对应的精度发给黄庆东查看，避免二次传输或覆盖**</text>
1. <text bgcolor="light-yellow">**上传的文件如果是文件夹，需要先打成一个压缩包！！！**</text>

# 环境安装

## 本地机器或者服务器安装cos命令

pip install coscmd

## 生成配置文件

cat << EOF >> ~/.cos.conf

[common]

secret_id =TENCENT_CLOUD_SECRET_ID_REDACTED

secret_key =KSVP6arvl3A4lEVZP2ShiW7d7ZWIxVwE

bucket = hygon-torch-third-party-1251001002

region = ap-shanghai

max_thread = 5

part_size = 1

retry = 5

timeout = 60

schema = https

verify = md5

anonymous = False

EOF

# 上传文件

<text bgcolor="light-yellow">**为方便查找，模型和镜像请按单个文件或压缩包的形式上传！！！！**</text>

## 模型上传

coscmd upload xxx external/haiguang/model/

## 下载：

wget https://hygon-torch-third-party-1251001002.cos.ap-shanghai.myqcloud.com/external/haiguang/model/xxx

xxx为上传文件的名字

## 镜像上传

coscmd upload xxx external/haiguang/image/

## 下载：

wget https://hygon-torch-third-party-1251001002.cos.ap-shanghai.myqcloud.com/external/haiguang/image/xxx

xxx为上传文件的名字

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/TP|TP]]（concept）

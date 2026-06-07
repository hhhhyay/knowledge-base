---
title: 大文件上传下载
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: Q11hwhJbKiuag9kMQFCcGT0onXL
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/Q11hwhJbKiuag9kMQFCcGT0onXL
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 大文件上传下载

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/Q11hwhJbKiuag9kMQFCcGT0onXL)

主要以ac平台作为中转，先上传到ac平台，在下载到需要的平台

## 先下载快传

在待需要上传下载文件的环境中
```bash {wrap}
#下载快传
wget https://ksefile.hpccube.com:65241/efile/share/fasttrans-client.tar.gz

#解压
tar -xf fasttrans-client.tar.gz
#进入快传文件夹添加权限
cd fasttrans
chmod +x rayfile-c
```

## 上传到AC

先进入要上传到ac平台的文件夹路径

<image token="IXyhbQ3vWoe7HLxxT8ycwbUOn1U" width="2527" height="1267" align="center"/>

复制命令，需要在fasttrans文件夹执行，

<text bgcolor="light-yellow">./</text>rayfile-c -a ksefile.hpccube.com -P 65245 -u guobj -w b1ed11352f7dc4190f-8c47-4226-9e63-c5ec3c58281c -tm -no-meta -symbolic-links follow -retry 10 -retrytimeout 30 -o upload -d /images-test -s <text bgcolor="light-yellow"><请输入上传目标文件夹全路径并替换尖括号及本内容></text>

需要在前面添加./，后面修改为要上传的文件路径

<image token="W8bdbBipOoigSix5YXUc6fU1n2e" width="2235" height="271" align="center"/>

上传和下载的速度都挺快的，自己觉得会比直接下载更快些

## 下载文件

下载也是一样，选择ac里要下载的文件>分享>快传（大文件）

<image token="FaYKbOxnGoSq2kxn1nRceE3TnAf" width="2619" height="997" align="center"/>

也是进入fasttrans文件夹，执行命令

<image token="JkXMbCT2cosgwcx3DC2cKe8wnae" width="2355" height="213" align="center"/>

### 注意：

在第一次执行命令时可能会遇到下面错误，重新执行命令就可以了

<image token="PyxgbFbq7oOk37x0CIhcYZ7TnNc" width="1765" height="143" align="center"/>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/TP|TP]]（concept）

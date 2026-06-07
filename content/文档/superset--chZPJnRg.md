---
title: superset
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: I2f3d1OV7o4471xyxlRchZPJnRg
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/I2f3d1OV7o4471xyxlRchZPJnRg
last_synced_at: 2026-06-07T17:07:53.393Z
---

# superset

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/I2f3d1OV7o4471xyxlRchZPJnRg)

下载镜像
```plaintext {wrap}
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/apache/superset:5.0.0
```

创建容器（4.13节点）
```bash {wrap}
docker run -it --network=host --name=gbj-superset --privileged --device=/dev/kfd --device=/dev/dri --ipc=host --shm-size=64G --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -u root --ulimit stack=-1:-1 --ulimit memlock=-1:-1 -v /opt/hyhal:/opt/hyhal:ro -v `pwd`:/mnt -v /public/opendas/DL_DATA/llm-models:/model:ro swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/apache/superset:5.0.0 bash
```

<image token="EhP0b8CDCojWvuxwtcIcTdXfnHe" width="925" height="221" align="center"/>

上面镜像vim什么都没有，于是用dcu0711镜像创建的环境，参考这个（https://blog.csdn.net/zuochang_liu/article/details/113617727）
```plaintext {wrap}
docker run -it --network=host --name=gbj-superset --privileged --device=/dev/kfd --device=/dev/dri --ipc=host --shm-size=64G --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -u root --ulimit stack=-1:-1 --ulimit memlock=-1:-1 -v /opt/hyhal:/opt/hyhal:ro -v `pwd`:/mnt -v /public/opendas/DL_DATA/llm-models:/model:ro 0711镜像 bash
```

```bash {wrap}
pip3 install apache-superset
```

需要SECRET_KEY，可以自己指定

通过openssl rand -base64 42生成强安全密钥（https://superset.apache.org/docs/configuration/configuring-superset/#specifying-a-secret_key）

<image token="AS4YbumWIoMOh5xXiFOcJqlpnhh" width="462" height="48" align="center"/>

```bash {wrap}
openssl rand -base64 42
CRybjfcOgz3apFDDLIg3Zsl0n7+aj4PhdAGn3ENtPEn2RYKynIic5pvf
```

账户：root  密码:superset

<image token="LLs6b8J7eosfcsxtecWcPuQsnmh" width="619" height="141" align="center"/>

初始化superset db upgrade时报类似错https://blog.csdn.net/s609727454/article/details/148082725
```bash {wrap}
pip3 install marshmallow==3.26.1
```

安装mysqlclient报类似错

https://blog.csdn.net/qq_34511096/article/details/137609441
```bash {wrap}
apt-get update
apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config 
```

汉化的话需要在superset_config.py添加下这个，环境变量指定下config.py文件，

export SUPERSET_CONFIG_PATH=/mnt/server_tool/superset/superset_config.py
```sql {wrap}
BABEL_DEFAULT_LOCALE = 'zh'
LANGUAGES = { 'en': {'flag': 'us', 'name': 'English'}, 'zh': {'flag': 'cn', 'name': 'Chinese'}, }

```


```plaintext {wrap}
superset run -h 0.0.0.0 -p 8088 --with-threads --reload --debugger
```

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/Superset|Superset]]（tool）
- [[飞书知识图谱/实体/TP|TP]]（concept）

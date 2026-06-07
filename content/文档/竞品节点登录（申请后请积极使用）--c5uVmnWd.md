---
title: 竞品节点登录（申请后请积极使用）
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: CSu0dGS8Ao3bPjxLuexc5uVmnWd
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/docx/CSu0dGS8Ao3bPjxLuexc5uVmnWd
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 竞品节点登录（申请后请积极使用）

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/docx/CSu0dGS8Ao3bPjxLuexc5uVmnWd)

# ~~910B1环境（不可用）~~

<text color="red">请严格按照以下分配使用节点，切勿抢占！！！</text>

<text color="red">登录后在/mnt/nvme/下建立自己名字的目录，/root的根目录下空间较小，太多文件会导致系统无法使用。</text>

（1）登录方式(跳转登录，跳转节点也有卡)：

       命令： ssh root@116.142.76.183 -p 1122
       密码：So4!vcQCzC
       其他节点直接ssh过去，例如 ssh node224

（2）节点分配：

<lark-table rows="5" cols="5" column-widths="213,149,100,188,100">

  <lark-tr>
    <lark-td>
      hostname
    </lark-td>
    <lark-td>
      内网ip
    </lark-td>
    <lark-td>
      使用人
    </lark-td>
    <lark-td>
      使用时长
    </lark-td>
    <lark-td>
      备注
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node138（默认登录节点）
    </lark-td>
    <lark-td>
      172.16.0.138
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node224
    </lark-td>
    <lark-td>
      172.16.0.224
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node216
    </lark-td>
    <lark-td>
      172.16.0.216
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node51
    </lark-td>
    <lark-td>
      172.16.0.51
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
</lark-table>

（3）使用
```plaintext
#smi管理，采用大小命令
npu-smi info
```

# ~~A100环境（不可用）~~ {folded="true"}

<text color="red">请严格按照以下分配使用节点，切勿抢占！！！</text>

<text color="red">登录后在</text><text color="green">**家目录**</text><text color="red">下建立自己名字的目录</text>

<text color="red">模型数据统一放在/public/DL_DATA/llm-models下</text>

（1）登录方式分为两种账号：

       shareuser>>共享存储账号（<text color="green">空间有限</text>，只放必要数据，定期删除）；
       localuser>>（本地用户）；
       1）命令：ssh shareuser@112.28.49.224 -p 33121
            密码：sugon@qazwsx
       2) 命令：ssh localuser@112.28.49.224 -p 33121
           命令：sugon@qazwsx

（2）节点分配

<lark-table rows="5" cols="7" column-widths="100,165,123,131,100,100,100">

  <lark-tr>
    <lark-td>
      hostname
    </lark-td>
    <lark-td>
      外网ip（可直接登录）
    </lark-td>
    <lark-td>
      port
    </lark-td>
    <lark-td>
      内网ip
    </lark-td>
    <lark-td>
      使用人
    </lark-td>
    <lark-td>
      使用时长
    </lark-td>
    <lark-td>
      备注
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      a121
    </lark-td>
    <lark-td>
      112.28.49.224
    </lark-td>
    <lark-td>
      33121
    </lark-td>
    <lark-td>
      192.168.33.121
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      a122
    </lark-td>
    <lark-td>
      112.28.49.224
    </lark-td>
    <lark-td>
      33122
    </lark-td>
    <lark-td>
      192.168.33.122
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      a124
    </lark-td>
    <lark-td>
      112.28.49.224
    </lark-td>
    <lark-td>
      33124
    </lark-td>
    <lark-td>
      192.168.33.124
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      a125
    </lark-td>
    <lark-td>
      112.28.49.224
    </lark-td>
    <lark-td>
      33125
    </lark-td>
    <lark-td>
      192.168.33.125
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
</lark-table>

# ~~H100环境（不可用）~~

<text color="red">请严格按照以下分配使用节点，切勿抢占！！！</text>

<text color="red">登录后在</text><text color="green">**家目录**</text><text color="red">下建立自己名字的目录</text>

<text color="red">模型数据统一放在/public/DL_DATA/llm-models下</text>

（1）登录方式(跳转登录，跳转节点无卡)：

       1）命令： ssh QinYunchangan0218@111.19.168.145
            密码：ois8qi120nco
       2）其他节点ssh过去，例如 
            ssh shareuser@10.10.1.40（共享存储账号（<text color="green">空间有限</text>，只放必要数据，定期删除））
            ssh localuser@10.10.1.40（本地用户）

（2）节点分配

<lark-table rows="5" cols="7" column-widths="100,100,268,132,100,100,100">

  <lark-tr>
    <lark-td>
      hostname
    </lark-td>
    <lark-td>
      内网ip
    </lark-td>
    <lark-td>
      用户名
    </lark-td>
    <lark-td>
      登录密码
    </lark-td>
    <lark-td>
      使用人
    </lark-td>
    <lark-td>
      使用时长
    </lark-td>
    <lark-td>
      备注
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node1
    </lark-td>
    <lark-td>
      10.10.1.40
    </lark-td>
    <lark-td>
      shareuser或者localuser
    </lark-td>
    <lark-td>
      sugon@qazwsx
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node2
    </lark-td>
    <lark-td>
      10.10.1.41
    </lark-td>
    <lark-td>
      shareuser或者localuser
    </lark-td>
    <lark-td>
      sugon@qazwsx
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node3
    </lark-td>
    <lark-td>
      10.10.1.42
    </lark-td>
    <lark-td>
      shareuser或者localuser
    </lark-td>
    <lark-td>
      sugon@qazwsx
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      node4
    </lark-td>
    <lark-td>
      10.10.1.43
    </lark-td>
    <lark-td>
      shareuser或者localuser
    </lark-td>
    <lark-td>
      sugon@qazwsx
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
</lark-table>

# ~~**AMD MI308x（不可用）**~~

乌镇：<text color="red">10.16.6.30</text> & <text color="red">10.16.6.31</text>节点

# ~~**Nvidia 5090（不可用）**~~

# ~~A~~~~1~~~~00 资源~~~~（~~~~不~~~~可用）~~

<text color="red">请严格按照以下分配使用节点，切勿抢占！！！</text>

<text color="red">登录后在</text><text color="green">**家目录**</text><text color="red">下建立自己名字的目录</text>

<text bgcolor="light-yellow">个人数据请及时保存，避免丢失和损坏！！！！！！！</text>

（1）登录方式

首先通过公网登录A125节点，然后通过内网访问A122节点

       1）公网登录A125节点<text color="red">需要联系管理员申请密钥（一个组共用一套密钥）</text>，进行免密登录。

命令： ssh public_user@112.28.49.224 -p 62125 -i 私钥文件路径

或者

ssh localuser@112.28.49.224 -p 62125 -i 私钥文件路径

如图所示：

<image token="Efk5bKELnoNGCnxpdOOcamZ1n5b" width="1392" height="237" align="center"/>

备注：public_user为公共账号，locauser为本地账号

       2）通过内网跳转其余节点，例如 
            ssh public_user@10.16.2.122（共享存储账号（<text color="green">空间有限</text>，只放必要数据，定期删除））

ssh localuser@10.16.2.122（家目录位于本地目录，不同节点间文件不共享）

(2) 节点分配

<lark-table rows="3" cols="8" column-widths="100,400,100,234,137,100,100,210">

  <lark-tr>
    <lark-td>
      hostname {align="center"}
    </lark-td>
    <lark-td>
      服务器配置
    </lark-td>
    <lark-td>
      内网ip {align="center"}
    </lark-td>
    <lark-td>
      用户名 {align="center"}
    </lark-td>
    <lark-td>
      登录密码 {align="center"}
    </lark-td>
    <lark-td>
      使用人 {align="center"}
    </lark-td>
    <lark-td>
      使用时长 {align="center"}
    </lark-td>
    <lark-td>
      备注 {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      a122 {align="center"}
    </lark-td>
    <lark-td rowspan="2">
      操作系统：Ubuntu 22.04.4 LTS
      CUDA:12.4
      CPU:Intel(R) Xeon(R) Platinum 8358P CPU @ 2.60GHz 
      内存：1.0T
      磁盘：894.3G SATA SSD+3.5T NVME SSD*4
      HCA:IB 200G*8+千兆网卡*1
      GPU: **NVIDIA** Corporation GA100 [A100 SXM4 80GB *8]
    </lark-td>
    <lark-td>
      10.16.2.122 {align="center"}
    </lark-td>
    <lark-td>
      public_user或者locauser {align="center"}
    </lark-td>
    <lark-td>
      {align="center"}
      使用秘钥
    </lark-td>
    <lark-td>
      质量组
    </lark-td>
    <lark-td>
      长期 {align="center"}
    </lark-td>
    <lark-td>
      <text color="red">勿占用，使用前请协调！</text> {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      a125 {align="center"}
    </lark-td>
    <lark-td>
      10.16.2.125 {align="center"}
    </lark-td>
    <lark-td>
      public_user或者locauser {align="center"}
    </lark-td>
    <lark-td>
      使用秘钥 {align="center"}
    </lark-td>
    <lark-td>
      HPC(负责人：陈永青)
    </lark-td>
    <lark-td>
      11月3号-12月3号 {align="center"}
    </lark-td>
    <lark-td>
      <text color="red">勿占用，使用前请协调！</text> {align="center"}
    </lark-td>
  </lark-tr>
</lark-table>

# H20资源（可用）

可使用共享用户或本地用户ssh远程连接，连接方式见下方环境信息表；

<text bgcolor="red">**所有H20节点禁止密码登录，请联系**</text><text bgcolor="red">**<mention-user id="ou_0692d99055bbbf56840a9c25b1d409c8"/>**</text><text bgcolor="red">**或**</text><text bgcolor="red">**<mention-user id="ou_00ab4987d9d418acfa897116dae1e1f2"/>**</text><text bgcolor="red">**获取私钥进行登录。**</text>

共享用户家目录位于共享存储/public/home，多节点间文件共享；

本地用户家目录位于本地目录/home，不同节点间文件不共享。

<text color="red">登录后在</text><text color="green">**家目录**</text><text color="red">下建立自己名字的目录</text>

---

目录结构：

---

/public/opendas可以存放共享文件

4用于本地存放模型文件

/data目录通常存放docker容器及镜像数据

<text color="red">请严格按照以下环境信息列表中占用情况及时长使用节点，切勿抢占！！！</text>

独占节点用户名是locauser，节点被独占了其他人就用不了了

locauser用户家目录独立于本地磁盘，多节点之间文件不共享，与public_user家目录文件也不共享

环境信息：

<sheet token="Uj0Nsz7rxhbHfEtdMY0cLs77nZ3_8KRlMf"/>


```plaintext {wrap}
mount -t ceph bjds@.cfs_ssd=/volumes/cfs/bjds/3c36c1e4-e332-42f2-8ac9-12030e3825ea /public -o secretfile=/etc/ceph/ceph.client.bjds.keyring,conf=/etc/ceph/ceph.conf,mon_addr=10.55.55.101:6789/10.55.55.103:6789/10.55.55.104:6789,recover_session=clean
```

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/EP|EP]]（concept）

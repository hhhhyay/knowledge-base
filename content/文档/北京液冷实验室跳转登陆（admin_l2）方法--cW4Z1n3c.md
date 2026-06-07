---
title: 北京液冷实验室跳转登陆（admin_l2）方法
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: KeK9wwmbpilld8kwgd3cW4Z1n3c
feishu_type: DOCX
source_url: https://r0ddbu55vzx.feishu.cn/wiki/KeK9wwmbpilld8kwgd3cW4Z1n3c
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 北京液冷实验室跳转登陆（admin_l2）方法

[在飞书中打开](https://r0ddbu55vzx.feishu.cn/wiki/KeK9wwmbpilld8kwgd3cW4Z1n3c)

一．跳转登陆方法：
 

<lark-table rows="5" cols="2" column-widths="100,673">

  <lark-tr>
    <lark-td>
      跳转节点（user）：
    </lark-td>
    <lark-td>
      IP 10.0.50.81
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
    </lark-td>
    <lark-td>
      用户名称：admin_l2
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
    </lark-td>
    <lark-td>
      密码：612_HPC@l2
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
    </lark-td>
    <lark-td>
      注意：登陆后请在 /mnt/lost+found/admin-l2/下建立自己的文件夹使用，避免后续混乱。
    </lark-td>
  </lark-tr>
</lark-table>

 
二．动态口令获取方法：

Google官方口令验证器可在相关平台（iso与android）的应用商店里找到，由于国内网络原因一般无法正常下载，这里介绍一个微信小程序用来关联动态口令令牌。

1.在微信小程序中搜索“运维密码”点击进入

<image token="Y5U6brQbro5pnBxtoPYcx1g0nYG" width="318" height="212" align="left"/>

 备注：也可用app:   Authenticator

<image token="N2XRbQy6uo3OAsxBoPQcH8pOnDe" width="429" height="817" align="left"/>

2.进入后点击“添加场景”，然后扫描之前生成的二维码或者从手机相册中选择之前保存的二维码，即可添加相关设备的动态口令

<image token="XRlabnbJeodbmXxJqowcCDPmn7f" width="343" height="385" align="left"/>

 3.添加场景-扫描下图二维码（由于拷贝home到/mnt/lost+found/admin-l2/home-admin原动态密钥出现异常，现更新二维码动态密钥如下，请重新获取密钥登陆。）

<image token="Xrv1bdOInol9ulxoQ4ncTm4qnBf" width="621" height="660" align="left"/>

4.确认地址和信息，绑定完成后效果如下，6位数字即为动态口令

<image token="QW7EbnMDPotT8Px4exZcPIbWnQh" width="340" height="458" align="left"/>

<image token="Wj3ObNAVEoOCHyxYyn0cOxSUnWg" width="336" height="455" align="left"/>

5.登录时，会提示先输入动态口令，然后再输入密码，两个口令均验证通过则正常登录

<image token="IC49bmpEuoHZxvxAiAwcqReInQe" width="524" height="319" align="left"/>

XShell设置:

预设用户名：admin_l2密码：612_HPC@l2 方法选择：Keybnoard

<image token="MdznbBaESoqoRTxtPeFcLoVun4b" width="892" height="838" align="left"/>

连接会话输入动态口令登陆

<image token="HBQ9biGuAoptVtxR22KcFICInPw" width="1333" height="817" align="left"/>

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

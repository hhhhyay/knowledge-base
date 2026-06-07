---
title: PD分离：deepseek-r1-fp8测试记录
tags:
  - AI
  - LLM
  - Benchmark
model:
  - DeepSeek-R1-Channel-FP8
  - deepseek-r1-fp8
framework:
  - Mooncake
  - SGLang
  - vLLM
issue_type:
  - OOM
source: feishu
feishu_token: M7BIwsGVwikaWYkCqhncztXnnl9
feishu_type: DOCX
source_url: https://bvjoh3z2qoz.feishu.cn/wiki/M7BIwsGVwikaWYkCqhncztXnnl9
last_synced_at: 2026-06-07T17:07:53.393Z
---

# PD分离：deepseek-r1-fp8测试记录

[在飞书中打开](https://bvjoh3z2qoz.feishu.cn/wiki/M7BIwsGVwikaWYkCqhncztXnnl9)

### 测试机信息

<image token="HxsgbppdsooIBdxYlkPcBhzXneg" width="2154" height="255" align="center"/>

### 测试要求及测试数据

[[飞书知识图谱/资源/JD PD分离测试要求--cu1jhned|JD PD分离测试要求]]

### 昆山测试环境
```bash {wrap}
docker load -i sglang_daily-20260507-1407.tar.gz
Loaded image: 10.16.1.254:5000/jenkins/model_test_env/sglang:daily-20260507-1407

images=10.16.1.254:5000/jenkins/model_test_env/sglang:daily-20260507-1407
cname=dpsk-r1-fp8
docker run -id \
--shm-size 1800g \
--network=host \
--name=$cname \
--privileged \
--device=/dev/kfd \
--device=/dev/dri \
--device=/dev/mkfd \
--ipc=host \
--group-add video \
--cap-add=SYS_PTRACE \
--security-opt seccomp=unconfined \
-u root --ulimit stack=-1:-1 --ulimit memlock=-1:-1 \
-v /opt/hyhal/:/opt/hyhal/:ro \
-v /model:/model \
-v /mnt:/mnt \
-v /public/home/public_user:/public/home/public_user \
$images /bin/bash

# 进⼊容器
docker exec -it $cname bash

```

#### pip环境
```bash {wrap}
# pip list
Package                           Version
--------------------------------- ------------------------------------------------------------
accelerate                        1.13.0
aiofiles                          25.1.0
aiohappyeyeballs                  2.6.1
aiohttp                           3.13.5
aiosignal                         1.4.0
airportsdata                      20260315
aiter                             0.1.2+das.opt1.dtk2604.torch290.2605071217.gcd1608
amdsmi                            24.5.3+02cbffb.dirty
annotated-doc                     0.0.4
annotated-types                   0.7.0
anthropic                         0.100.0
anyio                             4.13.0
apache-tvm-ffi                    0.1.0
astor                             0.8.1
asttokens                         3.0.1
async-timeout                     5.0.1
attrs                             26.1.0
backports.asyncio.runner          1.2.0
blake3                            1.0.8
blinker                           1.9.0
blobfile                          3.0.0
boto3                             1.43.5
botocore                          1.43.5
build                             1.5.0
cachetools                        7.1.1
cbor2                             6.0.1
certifi                           2026.4.22
cffi                              2.0.0
charset-normalizer                3.4.7
click                             8.3.3
cloudpickle                       3.1.2
cmake                             3.29.2
compressed-tensors                0.13.0
cryptography                      48.0.0
cuda-bindings                     12.9.6
cuda-pathfinder                   1.5.4
cuda-python                       12.9.0
cupy                              12.3.0
datasets                          4.8.5
dbus-python                       1.2.18
decorator                         5.2.1
deep_ep                           1.0.0+das.opt1.dtk2604.torch290.202604031828.ga42ecd
deepgemm                          2.1.0+das.dtk2604.torch290.2604281619.ge4c5ee
depyf                             0.20.0
dill                              0.4.1
diskcache                         5.6.3
distro                            1.7.0
dnspython                         2.8.0
docstring_parser                  0.18.0
eft                               0.0.7
einops                            0.8.2
email-validator                   2.3.0
exceptiongroup                    1.3.1
executing                         2.2.1
fastapi                           0.136.1
fastapi-cli                       0.0.24
fastapi-cloud-cli                 0.17.1
fastar                            0.11.0
fastrlock                         0.8.3
filelock                          3.29.0
flash_attn                        2.8.3+das.opt1.dtk2604.torch290.2604300943.gc37c96
flash_mla                         1.0.0+das.opt1.dtk2604.torch290.2604100931.g261aa8
Flask                             3.1.3
frozenlist                        1.8.0
fsspec                            2026.2.0
future                            1.0.0
gguf                              0.19.0
google-api-core                   2.30.3
google-auth                       2.50.0
google-cloud-core                 2.5.1
google-cloud-storage              3.10.1
google-crc32c                     1.8.0
google-resumable-media            2.8.2
googleapis-common-protos          1.74.0
greenlet                          3.5.0
grpcio                            1.80.0
grpcio-health-checking            1.80.0
grpcio-reflection                 1.80.0
grpcio-tools                      1.80.0
h11                               0.16.0
h2                                4.3.0
hf-xet                            1.5.0
hiredis                           3.3.1
hpack                             4.1.0
httpcore                          1.0.9
httplib2                          0.20.2
httptools                         0.7.1
httpx                             0.28.1
httpx-sse                         0.4.3
huggingface_hub                   1.6.0
humanize                          4.15.0
Hypercorn                         0.18.0
hyperframe                        6.1.0
idna                              3.13
ijson                             3.5.0
importlib-metadata                4.6.4
iniconfig                         2.3.0
interegular                       0.3.3
ipython                           8.39.0
itsdangerous                      2.2.0
jedi                              0.20.0
jeepney                           0.7.1
Jinja2                            3.1.6
jiter                             0.14.0
jmespath                          1.1.0
jsonschema                        4.26.0
jsonschema-specifications         2025.9.1
keyring                           23.5.0
lark                              1.2.2
launchpadlib                      1.10.16
lazr.restfulclient                0.14.4
lazr.uri                          1.0.6
libnacl                           2.1.0
lightop                           0.6.0+das.dtk2604.torch290.2605071031.g99c3f7
llguidance                        0.7.30
llvmlite                          0.44.0
lm-format-enforcer                0.11.3
lmslim                            0.3.1+das.opt4.dtk2604.torch290.2604281437.g61fdfe
loguru                            0.7.3
lxml                              6.1.0
markdown-it-py                    4.1.0
MarkupSafe                        3.0.3
matplotlib-inline                 0.2.1
mcp                               1.27.0
mdurl                             0.1.2
mistral_common                    1.11.2
ml_dtypes                         0.5.4
model-hosting-container-standards 0.1.15
modelscope                        1.36.3
mooncake-transfer-engine          0.3.7.post2+das.opt1.dtk2604.torch290
more-itertools                    8.10.0
mpmath                            1.3.0
msgpack                           1.1.2
msgspec                           0.21.1
multidict                         6.7.1
multiprocess                      0.70.19
nest-asyncio                      1.6.0
networkx                          3.4.2
ninja                             1.11.1
numa                              1.4.6
numba                             0.61.2
numpy                             1.25.0
nvidia-cutlass-dsl                4.4.2
nvidia-cutlass-dsl-libs-base      4.4.2
nvidia-ml-py                      13.595.45
oauthlib                          3.2.0
openai                            2.6.1
openai-harmony                    0.0.4
opencv-python-headless            4.13.0.92
orjson                            3.11.9
outlines                          0.1.11
outlines_core                     0.1.26
packaging                         26.2
pandas                            2.3.3
parso                             0.8.7
partial-json-parser               0.2.1.1.post7
peft                              0.19.1
pexpect                           4.9.0
pillow                            12.2.0
pip                               26.0.1
pluggy                            1.6.0
priority                          2.0.0
prometheus_client                 0.25.0
prometheus-fastapi-instrumentator 7.1.0
prompt_toolkit                    3.0.52
propcache                         0.4.1
proto-plus                        1.27.2
protobuf                          6.33.6
psutil                            7.2.2
ptyprocess                        0.7.0
pure_eval                         0.2.3
py-cpuinfo                        9.0.0
py-spy                            0.4.2
pyarrow                           24.0.0
pyasn1                            0.6.3
pyasn1_modules                    0.4.2
pybase64                          1.4.3
pybind11                          3.0.4
pycountry                         26.2.16
pycparser                         3.0
pycryptodomex                     3.23.0
pydantic                          2.13.4
pydantic_core                     2.46.4
pydantic-extra-types              2.11.1
pydantic-settings                 2.14.0
Pygments                          2.20.0
PyGObject                         3.42.1
PyHive                            0.7.0
PyJWT                             2.12.1
PyMySQL                           1.1.3
pyparsing                         2.4.7
pyproject_hooks                   1.2.0
pytest                            9.0.3
pytest-asyncio                    1.3.0
python-apt                        2.4.0+ubuntu4.1
python-dateutil                   2.9.0.post0
python-dotenv                     1.2.2
python-json-logger                4.1.0
python-multipart                  0.0.27
pytz                              2026.2
PyYAML                            6.0.3
pyzmq                             27.1.0
quack-kernels                     0.4.1
Quart                             0.20.0
ray                               2.55.1
redis                             7.4.0
referencing                       0.37.0
regex                             2026.4.4
requests                          2.33.1
rich                              15.0.0
rich-toolkit                      0.19.7
rignore                           0.7.6
rpds-py                           0.30.0
runai-model-streamer              0.15.3
runai-model-streamer-gcs          0.15.3
runai-model-streamer-s3           0.15.3
s3transfer                        0.17.0
safetensors                       0.7.0
scipy                             1.15.3
SecretStorage                     3.3.1
sentencepiece                     0.2.1
sentry-sdk                        2.59.0
setproctitle                      1.3.7
setuptools                        79.0.1
setuptools-scm                    10.0.5
sglang                            0.5.10rc0+das.opt2.alpha.dtk2604.torch290.2605071347.g3c1dba
sglang-kernel                     0.4.0+das.opt1.dtk2604.torch290.2605071347.g3c1dba
sglang-router                     0.3.2+das.opt1.dtk2604.torch290.2605071347.g3c1dba
shellingham                       1.5.4
six                               1.16.0
smg-grpc-proto                    0.4.7
smg-grpc-servicer                 0.5.2
sniffio                           1.3.1
soundfile                         0.13.1
SQLAlchemy                        2.0.49
sse-starlette                     3.4.2
stack-data                        0.6.3
starlette                         0.52.1
supervisor                        4.3.0
sympy                             1.14.0
taskgroup                         0.2.2
tensorizer                        2.10.1
threadpoolctl                     3.6.0
tiktoken                          0.12.0
tilelang                          0.1.7.post3+das.opt1.dtk2604.torch290.2604281919.gb3deab
timm                              1.0.16
tokenizers                        0.22.2
tomli                             2.4.1
torch                             2.9.0+das.opt1.dtk2604.2604281634.ged188f
torch_c_dlpack_ext                0.1.5
torchaudio                        2.9.0+das.opt1.dtk2604.torch290.202604011034.geaa9e4
torchvision                       0.24.0+das.opt1.dtk2604.torch290.2604091152.g66069c
tqdm                              4.67.3
traitlets                         5.15.0
transformers                      5.3.0
triton                            3.5.1+das.opt1.dtk2604.torch290.2604281605.g3a3ab3
typer                             0.25.1
typer-slim                        0.24.0
typing_extensions                 4.15.0
typing-inspection                 0.4.2
tzdata                            2026.2
urllib3                           2.6.3
uvicorn                           0.46.0
uvloop                            0.22.1
vcs-versioning                    1.1.1
vllm                              0.15.1+das.opt1.alpha.dtk2604.torch290.2605061705.gca9ce1
wadllib                           1.3.6
watchfiles                        1.1.1
wcwidth                           0.7.0
websockets                        16.0
Werkzeug                          3.1.8
wheel                             0.37.1
wsproto                           1.3.2
xgrammar                          0.1.32
xxhash                            3.7.0
yarl                              1.23.0
z3-solver                         4.16.0.0
zipp                              1.0.0

```

### 查看网络配置
```bash
# config使用dcu bus地址
# hy-smi --showbus

================================= System Management Interface ==================================
================================================================================================
HCU[0]          : PCI Bus: 0000:05:00.0 --> SN: 2500238601000586
HCU[1]          : PCI Bus: 0000:2b:00.0 --> SN: 2500238601000600
HCU[2]          : PCI Bus: 0000:14:00.0 --> SN: N/A
HCU[3]          : PCI Bus: 0000:1a:00.0 --> SN: 2500238601000537
HCU[4]          : PCI Bus: 0000:86:00.0 --> SN: N/A
HCU[5]          : PCI Bus: 0000:b8:00.0 --> SN: 2500238601000467
HCU[6]          : PCI Bus: 0000:c0:00.0 --> SN: N/A
HCU[7]          : PCI Bus: 0000:9e:00.0 --> SN: 2500238601000574
================================================================================================
======================================== End of SMI Log ========================================

# 使用的是DCU的地址，非网卡地址
# lspci -D | grep -i mellanox 
0000:06:00.0 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
0000:06:00.1 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
0000:17:00.0 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
0000:17:00.1 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
0000:83:00.0 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
0000:83:00.1 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
0000:c1:00.0 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
0000:c1:00.1 Ethernet controller: Mellanox Technologies MT2910 Family [ConnectX-7]
root@S1:/public/home/public_user/pd-test/net# ibdev2netdev
mlx5_0 port 1 ==> ens30f0np0 (Up)
mlx5_1 port 1 ==> ens30f1np1 (Up)
mlx5_2 port 1 ==> enp23s0f0np0 (Up)
mlx5_3 port 1 ==> enp23s0f1np1 (Up)
mlx5_4 port 1 ==> enp131s0f0np0 (Up)
mlx5_5 port 1 ==> enp131s0f1np1 (Up)
mlx5_6 port 1 ==> enp193s0f0np0 (Up)
mlx5_7 port 1 ==> enp193s0f1np1 (Up)

$ show_gids | grep 192 | grep v2
mlx5_0  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0101 192.168.1.1     v2      ens30f0np0
mlx5_1  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0201 192.168.2.1     v2      ens30f1np1
mlx5_2  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0301 192.168.3.1     v2      enp23s0f0np0
mlx5_3  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0401 192.168.4.1     v2      enp23s0f1np1
mlx5_4  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0501 192.168.5.1     v2      enp131s0f0np0
mlx5_5  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0601 192.168.6.1     v2      enp131s0f1np1
mlx5_6  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0701 192.168.7.1     v2      enp193s0f0np0
mlx5_7  1       3       0000:0000:0000:0000:0000:ffff:c0a8:0801 192.168.8.1     v2      enp193s0f1np1

# lspci -t -v
-+-[0000:cb]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
 |           +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
 |           +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
 |           +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-01.5-[cc]--+-00.0  Red Hat, Inc. Virtio network device
 |           |            \-00.1  Tencent Technology (Shenzhen) Company Limited Device 1600
 |           +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.1-[cd]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
 |           |            \-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
 |           \-07.2-[ce]--+-00.0  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
 |                        \-00.1  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
 +-[0000:b2]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
 |           +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
 |           +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
 |           +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-01.1-[b3-bb]----00.0-[b4-bb]--+-00.0-[b5]--
 |           |                               +-01.0-[b6-b8]----00.0-[b7-b8]----00.0-[b8]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
 |           |                               +-02.0-[b9]----00.0  Device 025e:0b60
 |           |                               +-03.0-[ba]--
 |           |                               \-1a.0-[bb]----00.0  Device 1eb6:6011
 |           +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.1-[bc-c1]----00.0-[bd-c1]--+-00.0-[be-c0]----00.0-[bf-c0]----00.0-[c0]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
 |           |                               \-01.0-[c1]--+-00.0  Mellanox Technologies MT2910 Family [ConnectX-7]
 |           |                                            \-00.1  Mellanox Technologies MT2910 Family [ConnectX-7]
 |           +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           \-07.1-[c2]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
 |                        \-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
 +-[0000:99]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
 |           +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
 |           +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
 |           +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-01.1-[9a-a2]----00.0-[9b-a2]--+-00.0-[9c-9e]----00.0-[9d-9e]----00.0-[9e]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
 |           |                               +-01.0-[9f]--
 |           |                               +-02.0-[a0]--
 |           |                               +-03.0-[a1]--
 |           |                               \-1a.0-[a2]----00.0  Device 1eb6:6011
 |           +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           \-07.1-[a3]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
 |                        \-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
 +-[0000:80]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
 |           +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
 |           +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
 |           +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-01.1-[81-86]----00.0-[82-86]--+-00.0-[83]--+-00.0  Mellanox Technologies MT2910 Family [ConnectX-7]
 |           |                               |            \-00.1  Mellanox Technologies MT2910 Family [ConnectX-7]
 |           |                               \-01.0-[84-86]----00.0-[85-86]----00.0-[86]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
 |           +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.1-[87]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
 |           |            +-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
 |           |            \-00.5  Advanced Micro Devices, Inc. [AMD] Device 14ca
 |           \-07.2-[88]--+-00.0  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
 |                        \-00.1  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
 +-[0000:34]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
 |           +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
 |           +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
 |           +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-01.3-[35-49]----00.0-[36-49]--+-02.0-[37-39]----00.0-[38-39]----1a.0-[39]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device a110
 |           |                               +-03.0-[3a-3c]----00.0-[3b-3c]----1a.0-[3c]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device a110
 |           |                               +-04.0-[3d-3f]----00.0-[3e-3f]----1a.0-[3f]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device a110
 |           |                               +-05.0-[40-42]----00.0-[41-42]----1a.0-[42]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device a110
 |           |                               +-06.0-[43-45]----00.0-[44-45]----1a.0-[45]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device a110
 |           |                               +-07.0-[46-48]----00.0-[47-48]----1a.0-[48]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device a110
 |           |                               \-1a.0-[49]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device a110
 |           +-01.5-[4a]----00.0  Red Hat, Inc. Virtio network device
 |           +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.2-[4b-4c]----00.0-[4c]----00.0  ASPEED Technology, Inc. ASPEED Graphics Family
 |           +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.1-[4d]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
 |           |            +-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
 |           |            \-00.4  Advanced Micro Devices, Inc. [AMD] Device 14c9
 |           \-07.2-[4e]--+-00.0  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
 |                        \-00.1  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
 +-[0000:26]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
 |           +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
 |           +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
 |           +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-01.1-[27-2c]----00.0-[28-2c]--+-00.0-[29-2b]----00.0-[2a-2b]----00.0-[2b]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
 |           |                               \-01.0-[2c]--
 |           +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           \-07.1-[2d]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
 |                        \-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
 +-[0000:0e]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
 |           +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
 |           +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
 |           +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-01.1-[0f-14]----00.0-[10-14]--+-00.0-[11]--
 |           |                               \-01.0-[12-14]----00.0-[13-14]----00.0-[14]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
 |           +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-03.1-[15-1d]----00.0-[16-1d]--+-00.0-[17]--+-00.0  Mellanox Technologies MT2910 Family [ConnectX-7]
 |           |                               |            \-00.1  Mellanox Technologies MT2910 Family [ConnectX-7]
 |           |                               +-01.0-[18-1a]----00.0-[19-1a]----00.0-[1a]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
 |           |                               +-02.0-[1b]--
 |           |                               +-03.0-[1c]----00.0  Shenzhen DAPU Microelectronics Co., Ltd NVMe SSD Controller DPU600
 |           |                               \-1a.0-[1d]----00.0  Device 1eb6:6011
 |           +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
 |           \-07.1-[1e]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
 |                        \-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
 \-[0000:00]-+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14a4
             +-00.2  Advanced Micro Devices, Inc. [AMD] Device 149e
             +-00.3  Advanced Micro Devices, Inc. [AMD] Device 14a6
             +-01.0  Advanced Micro Devices, Inc. [AMD] Device 149f
             +-01.1-[01-09]----00.0-[02-09]--+-00.0-[03-05]----00.0-[04-05]----00.0-[05]----00.0  Chengdu C-3000 IC Design Co., Ltd. Device 6430
             |                               +-01.0-[06]--+-00.0  Mellanox Technologies MT2910 Family [ConnectX-7]
             |                               |            \-00.1  Mellanox Technologies MT2910 Family [ConnectX-7]
             |                               +-02.0-[07]----00.0  Device 025e:0b60
             |                               +-03.0-[08]--
             |                               \-1a.0-[09]----00.0  Device 1eb6:6011
             +-02.0  Advanced Micro Devices, Inc. [AMD] Device 149f
             +-03.0  Advanced Micro Devices, Inc. [AMD] Device 149f
             +-04.0  Advanced Micro Devices, Inc. [AMD] Device 149f
             +-05.0  Advanced Micro Devices, Inc. [AMD] Device 149f
             +-07.0  Advanced Micro Devices, Inc. [AMD] Device 149f
             +-07.1-[0a]--+-00.0  Advanced Micro Devices, Inc. [AMD] Device 14ac
             |            +-00.1  Advanced Micro Devices, Inc. [AMD] Device 14dc
             |            +-00.4  Advanced Micro Devices, Inc. [AMD] Device 14c9
             |            \-00.5  Advanced Micro Devices, Inc. [AMD] Device 14ca
             +-07.2-[0b]--+-00.0  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
             |            \-00.1  Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode]
             +-14.0  Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller
             +-14.3  Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge
             +-18.0  Advanced Micro Devices, Inc. [AMD] Device 14ad
             +-18.1  Advanced Micro Devices, Inc. [AMD] Device 14ae
             +-18.2  Advanced Micro Devices, Inc. [AMD] Device 14af
             +-18.3  Advanced Micro Devices, Inc. [AMD] Device 14b0
             +-18.4  Advanced Micro Devices, Inc. [AMD] Device 14b1
             +-18.5  Advanced Micro Devices, Inc. [AMD] Device 14b2
             +-18.6  Advanced Micro Devices, Inc. [AMD] Device 14b3
             +-18.7  Advanced Micro Devices, Inc. [AMD] Device 14b4
             +-19.0  Advanced Micro Devices, Inc. [AMD] Device 14ad
             +-19.1  Advanced Micro Devices, Inc. [AMD] Device 14ae
             +-19.2  Advanced Micro Devices, Inc. [AMD] Device 14af
             +-19.3  Advanced Micro Devices, Inc. [AMD] Device 14b0
             +-19.4  Advanced Micro Devices, Inc. [AMD] Device 14b1
             +-19.5  Advanced Micro Devices, Inc. [AMD] Device 14b2
             +-19.6  Advanced Micro Devices, Inc. [AMD] Device 14b3
             \-19.7  Advanced Micro Devices, Inc. [AMD] Device 14b4
root@S1:/public/home/public_user/pd-test/net# hy-smi --showtopo
================================================================================================
HCU[0]          : (Topology) Numa Node 0
HCU[0]          : (Topology) Numa Affinity 0
HCU[1]          : (Topology) Numa Node 0
HCU[1]          : (Topology) Numa Affinity 0
HCU[2]          : (Topology) Numa Node 0
HCU[2]          : (Topology) Numa Affinity 0
HCU[3]          : (Topology) Numa Node 0
HCU[3]          : (Topology) Numa Affinity 0
HCU[4]          : (Topology) Numa Node 1
HCU[4]          : (Topology) Numa Affinity 1
HCU[5]          : (Topology) Numa Node 1
HCU[5]          : (Topology) Numa Affinity 1
HCU[6]          : (Topology) Numa Node 1
HCU[6]          : (Topology) Numa Affinity 1
HCU[7]          : (Topology) Numa Node 1
HCU[7]          : (Topology) Numa Affinity 1
================================================================================================
```


##### mc_topo.config
```bash
0000:05:00.0 mlx5_0 hip:0
0000:2b:00.0 mlx5_1 hip:1
0000:14:00.0 mlx5_2 hip:2
0000:1a:00.0 mlx5_3 hip:3
0000:86:00.0 mlx5_4 hip:4
0000:b8:00.0 mlx5_5 hip:5
0000:c0:00.0 mlx5_6 hip:6
0000:9e:00.0 mlx5_7 hip:7
```

##### topo.config
```bash
0000:05:00.0 mlx5_0 0
0000:2b:00.0 mlx5_1 1
0000:14:00.0 mlx5_2 2
0000:1a:00.0 mlx5_3 3
0000:86:00.0 mlx5_4 4
0000:b8:00.0 mlx5_5 5
0000:c0:00.0 mlx5_6 6
0000:9e:00.0 mlx5_7 7
```

### 1P1D tp8测试

1P1D模式下，96K输入1K输出 gpu_mem=0.9 P节点 OOM

#### Prefill tp8
```bash {wrap}
#!/bin/bash

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_TOPO_FILE_FORCE=/public/home/public_user/pd-test/topo.config
export MC_TOPO_FILE_FORCE=/public/home/public_user/pd-test/mc_topo.config
export MC_IB_GID_INDEX=2
export MC_ENABLE_DEST_DEVICE_AFFINITY=1

model_path=/model/DeepSeek-R1-Channel-FP8
model=${model_path##*/}

HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
HOST=192.168.1.1
MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"
PORT=30000
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=12400
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/prefill_$model-tp$tp-dp$dp-ep$ep-${HOST_NAME}_${TIME_STR}.log"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7 "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $HOST  --port $PORT --trust-remote-code  \
                                --dist-init-addr ${DIST_INIT_ADDR} --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --disaggregation-mode prefill \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${LOG_FILE}


```

#### Decode tp8
```bash
#!/bin/bash

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_TOPO_FILE_FORCE=/public/home/public_user/pd-test/topo.config
export MC_TOPO_FILE_FORCE=/public/home/public_user/pd-test/mc_topo.config
export MC_IB_GID_INDEX=2
export MC_ENABLE_DEST_DEVICE_AFFINITY=1

HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
HOST=192.168.1.2
PORT="30003"

NNODES="1"
NODE_RANK="0"

MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

model_path=/model/DeepSeek-R1-Channel-FP8
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=12400
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_$model-tp$tp-dp$dp-ep$ep-${HOST_NAME}_${TIME_STR}.log"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7 "
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $HOST  --port $PORT --trust-remote-code  \
                                --dist-init-addr ${DIST_INIT_ADDR} --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --disaggregation-mode decode \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${LOG_FILE}

```

#### router.sh
```bash {wrap}
#!/bin/bash

PERFILL=http://192.168.1.1:30000
DECODE=http://192.168.1.2:30003
python3 -m sglang_router.launch_router --pd-disaggregation --prefill "${PERFILL}" --decode "${DECODE}" --policy round_robin --port 30002 > r.log 2>&1 &

```

#### benchmark
```bash {wrap}
python3 -m sglang.bench_serving --backend sglang-oai-chat    --port 30002 --host 0.0.0.0   --dataset-name random  --dataset-path  /public/home/public_user/ShareGPT_V3_unfiltered_cleaned_split.json --random-range-ratio 1   --random-input-len 2500         --random-output-len 1500         --num-prompts 1  --max-concurrency 1

```

#### 存在问题（已解决）

网络异常

<image token="KJRpb0f9co7FySxMt44cTnzOnfe" width="1884" height="908" align="center"/>

<image token="L4i7bektto307NxXe6ccTIaRnGl" width="1860" height="648" align="center"/>

处理方案
```bash {wrap}
export SGLANG_HOST_IP="${HOST}"
export MC_ENABLE_DEST_DEVICE_AFFINITY=1
export MC_IB_GID_INDEX=2 # 或者设置 3 ，默认是3 ，不设置也可以；
# GID信息通过命令查看：
show_gids | grep 192 | grep v2
```

#### Decode pp4tp2
```bash {wrap}
#!/bin/bash

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_TOPO_FILE_FORCE=/public/home/public_user/pd-test/topo.config
export MC_TOPO_FILE_FORCE=/public/home/public_user/pd-test/mc_topo.config
export MC_IB_GID_INDEX=2
export MC_ENABLE_DEST_DEVICE_AFFINITY=1

HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
HOST=192.168.1.2
PORT="30003"
export SGLANG_HOST_IP="${HOST}"
NNODES="1"
NODE_RANK="0"

MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

model_path=/model/DeepSeek-R1-Channel-FP8
model=${model_path##*/}
tp=2
pp=4
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=12400
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_$model-tp$tp-dp$dp-ep$ep-${HOST_NAME}_${TIME_STR}.log"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
# option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 " ## 仅 pp 1 时可用 
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7 "
option+=" --ep-size $ep "
option+=" --dp-size $dp --moe-dense-tp-size 1 --enable-dp-lm-head --enable-dp-attention"
option+=" --enable-dp-attention --moe-a2a-backend deepep "
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $HOST  --port $PORT --trust-remote-code  \
                                --dist-init-addr ${DIST_INIT_ADDR} --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --disaggregation-mode decode \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${LOG_FILE}


```

#### 报错
```bash {wrap}
FlashMLA/DCU MLA only supports a page_size of 64, change page_size to 64.
DeepEP MoE is enabled. The expert parallel size is adjusted to be the same as the tensor parallel size[2].
Pipeline parallelism is incompatible with overlap schedule.
Spec v2 is enabled for eagle/eagle3 speculative decoding and overlap schedule is turned on.
speculative_num_draft_tokens is adjusted to speculative_num_steps + 1 when speculative_eagle_topk == 1
KV cache is forced as chunk cache for decode server
/usr/local/lib/python3.10/dist-packages/sglang/srt/entrypoints/http_server.py:175: FastAPIDeprecationWarning: ORJSONResponse is deprecated, FastAPI now serializes data directly to JSON bytes via Pydantic when a return type or response model is set, which is faster and doesn't need a custom response class. Read more in the FastAPI docs: https://fastapi.tiangolo.com/advanced/custom-response/#orjson-or-response-model and https://fastapi.tiangolo.com/tutorial/response-model/
  from sglang.srt.utils.json_response import (
Traceback (most recent call last):
  File "/usr/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "/usr/local/lib/python3.10/dist-packages/sglang/launch_server.py", line 62, in <module>
    run_server(server_args)
  File "/usr/local/lib/python3.10/dist-packages/sglang/launch_server.py", line 47, in run_server
    launch_server(server_args)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/entrypoints/http_server.py", line 2169, in launch_server
    Engine._launch_subprocesses(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/entrypoints/engine.py", line 618, in _launch_subprocesses
    server_args.check_server_args()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/server_args.py", line 5987, in check_server_args
    and self.speculative_algorithm is None
AssertionError: Pipeline parallelism is not compatible with overlap schedule, speculative decoding, mixed chunked prefill.
INFO: Please install aiter if you want to infer with aiter_moe.


```

#### 解决

去掉 `--speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1` 参数重启

#### 报错2
```bash {wrap}
[2026-05-08 15:30:54 PP0 TP1 EP1] Load weight begin. avail mem=142.79 GB
INFO: Please install aiter if you want to infer with aiter_moe.

Loading safetensors checkpoint shards:   0% Completed | 0/163 [00:00<?, ?it/s][2026-05-08 15:30:54 PP3 TP0 EP0] Load weight begin. avail mem=142.79 GB
[2026-05-08 15:30:54 PP3 TP0 EP0] Only Deepseek V3/R1 on NV-platform with capability >= 80 or AMD-platform with capability >= gfx942(MI30x) can use shared experts fusion optimization. Shared experts fusion optimization is disabled.
Loading safetensors checkpoint shards:  71% Completed | 116/163 [00:02<00:02, 23.49it/s][2026-05-08 15:30:57 PP2 TP1 EP1] lm_head.weight not found in params_dict.
Loading safetensors checkpoint shards:  85% Completed | 139/163 [00:03<00:00, 35.48it/s][2026-05-08 15:30:58 PP0 TP0 EP0] lm_head.weight not found in params_dict.
[2026-05-08 15:30:58 PP1 TP0 EP0] lm_head.weight not found in params_dict.
Loading safetensors checkpoint shards: 100% Completed | 163/163 [00:03<00:00, 41.12it/s]
[2026-05-08 15:30:59 PP1 TP1 EP1] lm_head.weight not found in params_dict.
[2026-05-08 15:30:59 PP2 TP0 EP0] lm_head.weight not found in params_dict.
[2026-05-08 15:30:59 PP0 TP1 EP1] lm_head.weight not found in params_dict.
[2026-05-08 15:31:08 PP0 TP0 EP0] Scheduler hit an exception: Traceback (most recent call last):
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 3563, in run_scheduler_process
    scheduler = Scheduler(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 382, in __init__
    self.init_model_worker()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 626, in init_model_worker
    self.init_tp_model_worker()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 594, in init_tp_model_worker
    self.tp_worker = TpModelWorker(**worker_kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/tp_worker.py", line 261, in __init__
    self._init_model_runner()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/tp_worker.py", line 344, in _init_model_runner
    self._model_runner = ModelRunner(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 429, in __init__
    self.initialize(pre_model_load_memory)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 509, in initialize
    self.load_model()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 1221, in load_model
    self.model = self.loader.load_model(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_loader/loader.py", line 689, in load_model
    self.load_weights_and_postprocess(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_loader/loader.py", line 709, in load_weights_and_postprocess
    quant_method.process_weights_after_loading(module)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/quantization/w8a8_fp8.py", line 328, in process_weights_after_loading
    w1_packed = _pack_per_expert_deepep(w1)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/quantization/w8a8_fp8.py", line 316, in _pack_per_expert_deepep
    if is_moe_prefill() :
NameError: name 'is_moe_prefill' is not defined

[2026-05-08 15:31:08] Received sigquit from a child process. It usually means the child failed.

```

#### 报错3（未解决）

P D 同时去掉环境变量：

export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_FUSED_RMSNORM_ROPE=1
```bash {wrap}
[2026-05-08 15:56:39 PP3 TP0 EP0] Registering 0 cuda graph addresses
Capturing batches (bs=256 avail_mem=22.80 GB):   0%|          | 0/35 [00:02<?, ?it/s]
[2026-05-08 15:56:39 PP2 TP1 EP1] Registering 0 cuda graph addresses
[2026-05-08 15:56:39 PP2 TP0 EP0] Registering 0 cuda graph addresses
[2026-05-08 15:56:39 PP1 TP0 EP0] Scheduler hit an exception: Traceback (most recent call last):
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 645, in __init__
    self.capture()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 801, in capture
    _capture_one_stream()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 788, in _capture_one_stream
    ) = self.capture_one_batch_size(bs, forward, stream_idx)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 1016, in capture_one_batch_size
    run_once()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 1003, in run_once
    logits_output_or_pp_proxy_tensors = forward(
  File "/usr/local/lib/python3.10/dist-packages/torch/utils/_contextlib.py", line 120, in decorate_context
    return func(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3636, in forward
    hidden_states = self.model(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3450, in forward
    hidden_states, residual = layer(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3140, in forward
    hidden_states = self.mlp(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 932, in forward
    return self.forward_deepep(hidden_states, forward_batch, rms_weight=rms_weight, residual=residual,)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 1353, in forward_deepep
    final_hidden_states = self.experts(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/ep_moe/layer.py", line 374, in forward
    return self.forward_impl(hidden_states, topk_output)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/ep_moe/layer.py", line 389, in forward_impl
    dispatch_output = self.dispatcher.dispatch(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/batch_overlap/two_batch_overlap.py", line 1050, in dispatch
    return self._execute("dispatch", **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/batch_overlap/two_batch_overlap.py", line 1047, in _execute
    return getattr(self._inners[tbo_subbatch_index or 0], name)(**kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/token_dispatcher/deepep.py", line 891, in dispatch
    self.dispatch_a(hidden_states, topk_output)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/token_dispatcher/deepep.py", line 903, in dispatch_a
    inner_state = self._get_impl().dispatch_a(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/token_dispatcher/deepep.py", line 610, in dispatch_a
    buffer = self._get_buffer()
  File "/usr/local/lib/python3.10/dist-packages/torch/_dynamo/eval_frame.py", line 1044, in _fn
    return fn(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/token_dispatcher/deepep.py", line 825, in _get_buffer
    return DeepEPBuffer.get_deepep_buffer(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/token_dispatcher/deepep.py", line 230, in get_deepep_buffer
    cls._buffer = Buffer(
  File "/usr/local/lib/python3.10/dist-packages/deep_ep/buffer.py", line 153, in __init__
    self.runtime.sync(device_ids, ipc_handles, root_unique_id)
RuntimeError: Failed: CUDA error csrc/deep_ep.cu:279 'invalid argument'


```


### 1P2D:P tp8 D dp16ep16

#### Decode1
```bash {wrap}
#!/bin/bash

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_TOPO_FILE_FORCE=/public/home/public_user/pd-test/topo.config
export MC_TOPO_FILE_FORCE=/public/home/public_user/pd-test/mc_topo.config
export MC_IB_GID_INDEX=2
export MC_ENABLE_DEST_DEVICE_AFFINITY=1


HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
HOST=192.168.1.2
PORT="30003"
export SGLANG_HOST_IP="${HOST}"
MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

model_path=/model/DeepSeek-R1-Channel-FP8
model=${model_path##*/}
tp=16
pp=1
dp=16
ep=16
nodes=2
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=12400
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_$model-tp$tp-dp$dp-ep$ep-${HOST_NAME}_${TIME_STR}.log"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7 "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
option+=" --ep-size $ep "
option+=" --dp-size $dp --moe-dense-tp-size 1 --enable-dp-lm-head --enable-dp-attention"
option+=" --enable-dp-attention --moe-a2a-backend deepep "
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $HOST  --port $PORT --trust-remote-code  \
                                --dist-init-addr ${DIST_INIT_ADDR} --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --disaggregation-mode decode \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${LOG_FILE}


```

#### Decode2
```bash {wrap}
#!/bin/bash

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_TOPO_FILE_FORCE=/public/home/public_user/pd-test/topo.config
export MC_TOPO_FILE_FORCE=/public/home/public_user/pd-test/mc_topo.config
export MC_IB_GID_INDEX=2
export MC_ENABLE_DEST_DEVICE_AFFINITY=1


HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
HOST=192.168.1.3
PORT="30003"

MASTER_IP=192.168.1.2
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

model_path=/model/DeepSeek-R1-Channel-FP8
model=${model_path##*/}
tp=16
pp=1
dp=16
ep=16
nodes=2
rank=1
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=12400
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_$model-tp$tp-dp$dp-ep$ep-${HOST_NAME}_${TIME_STR}.log"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7 "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $HOST  --port $PORT --trust-remote-code  \
                                --dist-init-addr ${DIST_INIT_ADDR} --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --disaggregation-mode decode \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${LOG_FILE}


```

#### 报错
```bash {wrap}
Loading safetensors checkpoint shards: 100% Completed | 163/163 [00:16<00:00, 10.15it/s]
[2026-05-08 14:34:30 TP2] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:30 TP2] Load weight end. elapsed=32.63 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.33 GB, mem usage=41.72 GB.
[2026-05-08 14:34:30 TP0] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:30 TP0] Load weight end. elapsed=32.67 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.38 GB, mem usage=41.72 GB.
[2026-05-08 14:34:30 TP4] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:30 TP4] Load weight end. elapsed=32.71 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.37 GB, mem usage=41.72 GB.
[2026-05-08 14:34:30 TP3] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:30 TP3] Load weight end. elapsed=32.74 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.34 GB, mem usage=41.72 GB.
[2026-05-08 14:34:30 TP5] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:30 TP5] Load weight end. elapsed=32.75 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.33 GB, mem usage=41.72 GB.
[2026-05-08 14:34:30 TP6] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:30 TP6] Load weight end. elapsed=32.78 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.36 GB, mem usage=41.72 GB.
[2026-05-08 14:34:31 TP1] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:31 TP1] Load weight end. elapsed=33.03 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.33 GB, mem usage=41.72 GB.
[2026-05-08 14:34:31 TP7] Using FP8 KV cache but no scaling factors provided. Defaulting to scaling factors of 1.0. This may lead to less accurate results!
[2026-05-08 14:34:31 TP7] Load weight end. elapsed=33.14 s, type=DeepseekV3ForCausalLM, quant=compressed-tensors, avail mem=101.35 GB, mem usage=41.72 GB.
[2026-05-08 14:34:31 TP0] Using KV cache dtype: torch.float8_e4m3fn
[2026-05-08 14:34:32 TP0] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP0] Memory pool end. avail mem=14.15 GB
[2026-05-08 14:34:32 TP7] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP4] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP6] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP5] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP7] Memory pool end. avail mem=14.12 GB
[2026-05-08 14:34:32 TP4] Memory pool end. avail mem=14.14 GB
[2026-05-08 14:34:32 TP6] Memory pool end. avail mem=14.14 GB
[2026-05-08 14:34:32 TP5] Memory pool end. avail mem=14.10 GB
[2026-05-08 14:34:32 TP3] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP2] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP1] KV Cache is allocated. #tokens: 2659072, KV size: 87.01 GB
[2026-05-08 14:34:32 TP3] Memory pool end. avail mem=14.12 GB
[2026-05-08 14:34:32 TP2] Memory pool end. avail mem=14.11 GB
[2026-05-08 14:34:32 TP1] Memory pool end. avail mem=14.10 GB
[2026-05-08 14:34:34 TP5] Capture cuda graph begin. This can take up to several minutes. avail mem=13.96 GB
[2026-05-08 14:34:34 TP6] Capture cuda graph begin. This can take up to several minutes. avail mem=13.99 GB
[2026-05-08 14:34:34 TP7] Capture cuda graph begin. This can take up to several minutes. avail mem=13.97 GB
[2026-05-08 14:34:34 TP2] Capture cuda graph begin. This can take up to several minutes. avail mem=13.96 GB
[2026-05-08 14:34:34 TP4] Capture cuda graph begin. This can take up to several minutes. avail mem=13.99 GB
[2026-05-08 14:34:34 TP1] Capture cuda graph begin. This can take up to several minutes. avail mem=13.96 GB
[2026-05-08 14:34:34 TP0] Capture cuda graph begin. This can take up to several minutes. avail mem=14.00 GB
[2026-05-08 14:34:34 TP0] Capture cuda graph bs [1, 2, 3, 4, 5, 6, 7, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 40, 44, 48, 52, 56, 60, 64, 72, 80, 88, 96, 104, 112, 120, 128, 136, 144, 152, 160, 168, 176, 184, 192, 200, 208, 216, 224, 232, 240, 248, 256]
[2026-05-08 14:34:34 TP3] Capture cuda graph begin. This can take up to several minutes. avail mem=13.97 GB
Capturing batches (bs=256 avail_mem=13.61 GB):   0%|          | 0/51 [00:00<?, ?it/s][2026-05-08 14:34:42 TP3] Scheduler hit an exception: Traceback (most recent call last):
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 3563, in run_scheduler_process
    scheduler = Scheduler(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 382, in __init__
    self.init_model_worker()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 626, in init_model_worker
    self.init_tp_model_worker()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 594, in init_tp_model_worker
    self.tp_worker = TpModelWorker(**worker_kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/tp_worker.py", line 261, in __init__
    self._init_model_runner()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/tp_worker.py", line 344, in _init_model_runner
    self._model_runner = ModelRunner(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 429, in __init__
    self.initialize(pre_model_load_memory)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 648, in initialize
    self.init_device_graphs()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 2466, in init_device_graphs
    self.graph_runner = graph_runners[self.device](self)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 645, in __init__
    self.capture()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 801, in capture
    _capture_one_stream()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 788, in _capture_one_stream
    ) = self.capture_one_batch_size(bs, forward, stream_idx)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 1016, in capture_one_batch_size
    run_once()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/cuda_graph_runner.py", line 1003, in run_once
    logits_output_or_pp_proxy_tensors = forward(
  File "/usr/local/lib/python3.10/dist-packages/torch/utils/_contextlib.py", line 120, in decorate_context
    return func(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3636, in forward
    hidden_states = self.model(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3450, in forward
    hidden_states, residual = layer(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3140, in forward
    hidden_states = self.mlp(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 923, in forward
    return self.forward_normal(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 1077, in forward_normal
    final_hidden_states = self.experts(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/layer.py", line 1057, in forward
    return self.forward_impl(hidden_states, topk_output, shared_output, i_q, i_s, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/layer.py", line 1090, in forward_impl
    combine_input = self.run_moe_core(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/layer.py", line 1122, in run_moe_core
    return self.quant_method.apply(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/quantization/w8a8_fp8.py", line 397, in apply
    output = fused_moe_fp8_w8a8(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/fused_moe.py", line 1128, in fused_moe_fp8_w8a8
    block_size_m = cuda_config1["BLOCK_SIZE_M"]
KeyError: 'BLOCK_SIZE_M'

[2026-05-08 14:34:42] Received sigquit from a child process. It usually means the child failed.
INFO: Please install aiter if you want to infer with aiter_moe.


```

处理1

添加参数：
```bash {wrap}
export SGLANG_HOST_IP="${HOST}"

option+=" --ep-size $ep "
option+=" --dp-size $dp --moe-dense-tp-size 1 --enable-dp-lm-head --enable-dp-attention"
option+=" --enable-dp-attention --moe-a2a-backend deepep "
```

#### 调整后报错（临时解决-非最优方案）
```bash {wrap}
[Gloo] Rank 0 is connected to 0 peer ranks. Expected number of connected peer ranks is : 0
[Gloo] Rank 0 is connected to 0 peer ranks. Expected number of connected peer ranks is : 0
[2026-05-08 14:56:48 DP0 TP0 EP0] Init torch distributed ends. elapsed=2.76 s, mem usage=0.43 GB
[2026-05-08 14:56:48 DP7 TP7 EP7] Init torch distributed ends. elapsed=2.92 s, mem usage=0.46 GB
[2026-05-08 14:56:48 DP6 TP6 EP6] Init torch distributed ends. elapsed=3.09 s, mem usage=0.44 GB
[2026-05-08 14:56:48 DP5 TP5 EP5] Init torch distributed ends. elapsed=3.00 s, mem usage=0.48 GB
[2026-05-08 14:56:48 DP4 TP4 EP4] Init torch distributed ends. elapsed=3.23 s, mem usage=0.44 GB
[2026-05-08 14:56:48 DP3 TP3 EP3] Init torch distributed ends. elapsed=2.85 s, mem usage=0.46 GB
[2026-05-08 14:56:48 DP2 TP2 EP2] Init torch distributed ends. elapsed=2.80 s, mem usage=0.47 GB
[2026-05-08 14:56:48 DP1 TP1 EP1] Init torch distributed ends. elapsed=2.83 s, mem usage=0.48 GB
[2026-05-08 14:56:49 DP3 TP3 EP3] Load weight begin. avail mem=143.06 GB
[2026-05-08 14:56:49 DP0 TP0 EP0] Load weight begin. avail mem=143.10 GB
[2026-05-08 14:56:49 DP0 TP0 EP0] Only Deepseek V3/R1 on NV-platform with capability >= 80 or AMD-platform with capability >= gfx942(MI30x) can use shared experts fusion optimization. Shared experts fusion optimization is disabled.
[2026-05-08 14:56:49 DP2 TP2 EP2] Load weight begin. avail mem=143.05 GB
[2026-05-08 14:56:49 DP7 TP7 EP7] Load weight begin. avail mem=143.07 GB
[2026-05-08 14:56:49 DP1 TP1 EP1] Load weight begin. avail mem=143.05 GB
[2026-05-08 14:56:49 DP6 TP6 EP6] Load weight begin. avail mem=143.08 GB
[2026-05-08 14:56:49 DP5 TP5 EP5] Load weight begin. avail mem=143.05 GB
[2026-05-08 14:56:49 DP4 TP4 EP4] Load weight begin. avail mem=143.09 GB
INFO: Please install aiter if you want to infer with aiter_moe.

Loading safetensors checkpoint shards: 100% Completed | 163/163 [00:11<00:00, 14.24it/s]
[2026-05-08 14:57:03 DP2 TP2 EP2] Scheduler hit an exception: Traceback (most recent call last):
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 3563, in run_scheduler_process
    scheduler = Scheduler(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 382, in __init__
    self.init_model_worker()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 626, in init_model_worker
    self.init_tp_model_worker()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 594, in init_tp_model_worker
    self.tp_worker = TpModelWorker(**worker_kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/tp_worker.py", line 261, in __init__
    self._init_model_runner()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/tp_worker.py", line 344, in _init_model_runner
    self._model_runner = ModelRunner(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 429, in __init__
    self.initialize(pre_model_load_memory)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 509, in initialize
    self.load_model()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 1221, in load_model
    self.model = self.loader.load_model(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_loader/loader.py", line 689, in load_model
    self.load_weights_and_postprocess(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_loader/loader.py", line 709, in load_weights_and_postprocess
    quant_method.process_weights_after_loading(module)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/quantization/w8a8_fp8.py", line 328, in process_weights_after_loading
    w1_packed = _pack_per_expert_deepep(w1)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/quantization/w8a8_fp8.py", line 316, in _pack_per_expert_deepep
    if is_moe_prefill() :
NameError: name 'is_moe_prefill' is not defined

INFO: Please install aiter if you want to infer with aiter_moe.

```

可以通过启服务时去掉 `"--moe-dense-tp-size 1 --enable-dp-lm-head --enable-dp-attention"`

`option+=" --enable-dp-attention --moe-a2a-backend deepep "` 参数规避报错，但性能会变差；

### 2P1D: P tp8pp2 D tp8

P TP16
```bash {wrap}
#!/bin/bash

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_TOPO_FILE_FORCE=/public/home/public_user/pd-test/topo.config
export MC_TOPO_FILE_FORCE=/public/home/public_user/pd-test/mc_topo.config
export MC_IB_GID_INDEX=2
export MC_ENABLE_DEST_DEVICE_AFFINITY=1

model_path=/model/DeepSeek-R1-Channel-FP8
model=${model_path##*/}

HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
HOST=192.168.1.1
MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"
PORT=30000
export SGLANG_HOST_IP="${HOST}"
tp=16
pp=1
dp=1
ep=1
nodes=2
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=140000
gpu_mem=0.75
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/prefill_$model-tp$tp-dp$dp-ep$ep-${HOST_NAME}_${TIME_STR}.log"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7 "
#option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $HOST  --port $PORT --trust-remote-code  \
                                --dist-init-addr ${DIST_INIT_ADDR} --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --disaggregation-mode prefill \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${LOG_FILE}


```

D TP8
```bash {wrap}
#!/bin/bash

export USE_DCU_CUSTOM_ALLREDUCE=1
export SGL_CHUNKED_PREFIX_CACHE_THRESHOLD=0
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export GLIBC_TUNABLES=glibc.rtld.optional_static_tls=0x40000
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT=1200
export SGLANG_USE_LIGHTOP=1
export SGLANG_USE_OPT_CAT=1
export SGLANG_USE_FP8_W8A8_MOE=1

export SGLANG_USE_RMS_QUANT_PATH=1
export USE_FUSED_RMS_QUANT_PATH=1
export SGLANG_USE_FUSED_RMSNORM_ROPE=1
export SGLANG_TORCH_PROFILER_DIR=/workspace/prof
export SGLANG_SET_CPU_AFFINITY=1
export HIP_KERNEL_BATCH_CEILING=100
export GPU_MAX_HW_QUEUES=4
sysctl -w kernel.numa_balancing=0
export SGLANG_ENABLE_SPEC_V2=1
#triton改写算子
export SGLANG_KVALLOC_KERNEL=1
export SGLANG_CREATE_EXTEND_AFTER_DECODE_SPEC_INFO=1
export SGLANG_ASSIGN_EXTEND_CACHE_LOCS=1
export SGLANG_ASSIGN_REQ_TO_TOKEN_POOL=1
export SGLANG_GET_LAST_LOC=1
export SGLANG_CREATE_FLASHMLA_KV_INDICES_TRITON=1
export SGLANG_CREATE_CHUNKED_PREFIX_CACHE_KV_INDICES=1
#export SGLANG_USE_FUSED_MLA_CAT=1
#export HIP_GRAPH_ACCUMULATE_DISPATCH=0 #torchprof需要
export HIP_H2D_DISABLE_COPY_BUFFER=0 # 同步异步强制走WriteBuffer
export HIP_D2H_DISABLE_COPY_BUFFER=0 # 同步异步强制走ReadBuffer
export HIP_H2D_DIRECT_COPY_THRESHOLD=32768 # 小于此值走CPUCopy
export HIP_H2D_HSAAPI_COPY_THRESHOLD=32768 # 大于此值走HSACOPY（CopyBuffer）
export HIP_D2H_DIRECT_COPY_THRESHOLD=512 # 小于此值走CPUCopy
export HIP_D2H_HSAAPI_COPY_THRESHOLD=512 # 大于此值走HSACOPY（CopyBuffer）
#hiplaunch
export HSA_KERNARG_POOL_SIZE=8388608
export ROC_AQL_QUEUE_SIZE=131072
export NCCL_MAX_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export NCCL_MIN_NCHANNELS=16
export ALLREDUCE_STREAM_WITH_COMPUTE=1
export USE_SPE_MQP=1
export MC_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_ALLOWED_IBV_DEVICES=mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7
export ROCSHMEM_TOPO_FILE_FORCE=/public/home/public_user/pd-test/topo.config
export MC_TOPO_FILE_FORCE=/public/home/public_user/pd-test/mc_topo.config
export MC_IB_GID_INDEX=2
export MC_ENABLE_DEST_DEVICE_AFFINITY=1

HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
if [ -z "${HOST}" ]; then
    HOST=$(hostname -i 2>/dev/null | awk '{print $1}')
fi
HOST=192.168.1.3
PORT="30003"
export SGLANG_HOST_IP="${HOST}"
NNODES="1"
NODE_RANK="0"

MASTER_IP="${HOST}"
DIST_PORT="5000"
DIST_INIT_ADDR="${MASTER_IP}:${DIST_PORT}"

model_path=/model/DeepSeek-R1-Channel-FP8
model=${model_path##*/}
tp=8
pp=1
dp=1
ep=1
nodes=1
rank=0
#master_ip=
host_ip=$(hostname -I | awk '{print $1}')
master_ip=$host_ip
max_model_len=140000
gpu_mem=0.9
time=$(date "+%m%d-%H%M")
#mode="eager"
mode="cudagraph"
LOG_DIR="./logs"
mkdir -p "$LOG_DIR"

TIME_STR=$(date "+%Y%m%d-%H%M%S")
HOST_NAME=$(hostname)

LOG_FILE="${LOG_DIR}/decode_$model-tp$tp-dp$dp-ep$ep-${HOST_NAME}_${TIME_STR}.log"


if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi

option="--numa-node 0 0 0 0 1 1 1 1"
option+=" --disable-radix-cache "
option+=" --chunked-prefill-size -1"
option+=" --max-running-requests 256 "
option+=" --speculative-algorithm EAGLE --speculative-num-steps 2  --speculative-eagle-topk 1  --speculative-num-draft-tokens 1 "
option+=" --disaggregation-ib-device mlx5_0,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7 "
#SGLANG_HIP_PROF=1 hipprof --hip-trace --trace-off
python3 -m sglang.launch_server --model-path $model_path ${option} --context-length $max_model_len \
                                --quantization w8a8_fp8 --kv-cache-dtype fp8_e4m3 \
                                --host $HOST  --port $PORT --trust-remote-code  \
                                --dist-init-addr ${DIST_INIT_ADDR} --nnodes $nodes --node-rank $rank \
                                --dtype bfloat16  --tp-size $tp --pp-size $pp \
                                --disaggregation-mode decode \
                                --mem-fraction-static $gpu_mem --attention-backend dcu_mla \
                                2>&1 | tee  ${LOG_FILE}


```

#### 报错(已解决)
```bash {wrap}
[2026-05-08 20:32:53] INFO:     Started server process [28148]
[2026-05-08 20:32:53] INFO:     Waiting for application startup.
[2026-05-08 20:32:53] Using default chat sampling params from model generation config: {'temperature': 0.6, 'top_p': 0.95}
[2026-05-08 20:32:53] INFO:     Application startup complete.
[2026-05-08 20:32:53] INFO:     Uvicorn running on http://192.168.1.1:30000 (Press CTRL+C to quit)
[2026-05-08 20:32:54] INFO:     192.168.1.1:41496 - "GET /model_info HTTP/1.1" 200 OK
[2026-05-08 20:32:54] Start of pd disaggregation warmup ...
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
/usr/local/lib/python3.10/dist-packages/triton/backends/amd/compiler_hcu.py:501: UserWarning: Skip AMD buffer-ops lowering for this kernel because i64 addptr offsets were detected.
  warnings.warn(
[2026-05-08 20:33:20 TP3] Scheduler hit an exception: Traceback (most recent call last):
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 3579, in run_scheduler_process
    scheduler.run_event_loop()
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 1283, in run_event_loop
    dispatch_event_loop(self)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 3461, in dispatch_event_loop
    scheduler.event_loop_overlap_disagg_prefill()
  File "/usr/local/lib/python3.10/dist-packages/torch/utils/_contextlib.py", line 120, in decorate_context
    return func(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/disaggregation/prefill.py", line 402, in event_loop_overlap_disagg_prefill
    batch_result = self.run_batch(batch)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/scheduler.py", line 2626, in run_batch
    batch_result = self.model_worker.forward_batch_generation(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/managers/tp_worker.py", line 475, in forward_batch_generation
    out = self.model_runner.forward(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 2745, in forward
    output = self._forward_raw(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 2859, in _forward_raw
    ret, can_run_graph = self.forward_extend(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/model_executor/model_runner.py", line 2677, in forward_extend
    self.model.forward(
  File "/usr/local/lib/python3.10/dist-packages/torch/utils/_contextlib.py", line 120, in decorate_context
    return func(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3636, in forward
    hidden_states = self.model(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3450, in forward
    hidden_states, residual = layer(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 3140, in forward
    hidden_states = self.mlp(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 923, in forward
    return self.forward_normal(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/models/deepseek_v2.py", line 1077, in forward_normal
    final_hidden_states = self.experts(
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1775, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1786, in _call_impl
    return forward_call(*args, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/layer.py", line 1057, in forward
    return self.forward_impl(hidden_states, topk_output, shared_output, i_q, i_s, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/layer.py", line 1090, in forward_impl
    combine_input = self.run_moe_core(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/layer.py", line 1122, in run_moe_core
    return self.quant_method.apply(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/quantization/w8a8_fp8.py", line 397, in apply
    output = fused_moe_fp8_w8a8(
  File "/usr/local/lib/python3.10/dist-packages/sglang/srt/layers/moe/fused_moe_triton/fused_moe.py", line 1128, in fused_moe_fp8_w8a8
    block_size_m = cuda_config1["BLOCK_SIZE_M"]
KeyError: 'BLOCK_SIZE_M'

[2026-05-08 20:33:20] SIGQUIT received. signum=None, frame=None. It usually means one child failed.

```

### 批量测试脚本
```bash
model_path=/model/DeepSeek-R1-Channel-FP8

model='DeepSeek-R1-Channel-FP8'
time=$(date "+%m%d-%H%M")
mode="cudagraph"
rr=inf
logpath="./logs/${model}-${mode}-$time-$mode-rr$rr-mem09-$(hostname)"
port=30002
concurrency_multiplier=1
#host_ip=$(hostname -I | awk '{print $1}')
host_ip=0.0.0.0

#export HF_ENDPOINT="https://hf-mirror.com"

if [ ! -f ${logpath} ]; then
    mkdir ${logpath} -p
fi
all_log="${logpath}/all.csv"

#pairs=("4000 20" "3500 1500" "9000 1000")

echo "input,output,request_rate,max_concurrency,num_prompts,Peak_concurrent_requests,duration_s,rps,generate_throughput_tok_s,total_throughput_tok_s,Peak_output_token_throughput,mean_ttft_ms,p95_ttft_ms,p99_ttf     t_ms,mean_tpot_ms,p95_tpot_ms,p99_tpot_ms,mean_itl_ms,p95_itl_ms,p99_itl_ms" > $all_log

num_prompts=100


for rr  in 1.5 1.8; do
       for batch in 32 50 64 72 100; do

        prompt_tokens=2500
        completion_tokens=1500
        python3 -m sglang.bench_serving --backend sglang-oai-chat \
        --model ${model_path}   --request-rate $rr  \
        --port $port --host $host_ip \
        --dataset-name random --dataset-path /public/home/public_user/ShareGPT_V3_unfiltered_cleaned_split.json \
        --random-range-ratio 1  \
        --disable-ignore-eos \
        --random-input-len $prompt_tokens \
        --random-output-len $completion_tokens \
        --num-prompts $num_prompts  --max-concurrency $batch  2>&1 | tee ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log

        #Total token throughput
        Benchmark_duration=`grep -a "^Benchmark duration" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_input_tokens=`grep -a "^Total input tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Total_generated_tokens=`grep -a "^Total generated tokens" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Output_token_throughput=`grep -a "^Output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        Total_Token_throughput=`grep -a "^Total token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $5}'`
        request_rate=`grep -a "^Traffic request rate" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
#        Concurrency=`grep -a "^Concurrency" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $2}'`
        Concurrency=`grep -a "^Successful requests:" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $3}'`
        Peak_output_token_throughput=`grep -a "^Peak output token throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $6}'`
        Peak_concurrent_requests=`grep -a "^Peak concurrent requests" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_TTFT=`grep -a "^Mean TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Median_TTFT=`grep -a "^Median TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Median_TPOT=`grep -a "^Median TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Median_ITL=`grep -a "^Median ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_TPOT=`grep -a "^Mean TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        qps=`grep -a "^Request throughput" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P99_TTFT=`grep -a "^P99 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P99_TPOT=`grep -a "^P99 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P99_ITL=`grep -a "^P99 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P50_TTFT=`grep -a "^P50 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P50_TPOT=`grep -a "^P50 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P50_ITL=`grep -a "^P50 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        Mean_ITL=`grep -a "^Mean ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_TTFT=`grep -a "^P95 TTFT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_TPOT=`grep -a "^P95 TPOT" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`
        P95_ITL=`grep -a "^P95 ITL" ${logpath}/${model}-tp${tp}-${time}-${batch}-in${prompt_tokens}-out${completion_tokens}.log | awk -F ' ' '{print $4}'`


        echo "$prompt_tokens,$completion_tokens,$request_rate,$batch,$num_prompts,$Peak_concurrent_requests,$Benchmark_duration,$qps,$Output_token_throughput,$Total_Token_throughput,$Peak_output_token_throughput,$Mean_TTFT,$P95_TTFT,$P99_TTFT,$Mean_TPOT,$P95_TPOT,$P99_TPOT,$Mean_ITL,$P95_ITL,$P99_ITL">> $all_log
        cat $all_log


     done
done


```

## 关联资源

- [[飞书知识图谱/资源/JD PD分离测试要求--cu1jhned|JD PD分离测试要求]] · [在飞书中打开](https://bvjoh3z2qoz.feishu.cn/wiki/Quz2wT3WdiTlj6kitojcu1jhned)

## 关联知识

- [[飞书知识图谱/实体/Chunked Prefill|Chunked Prefill]]（concept）
- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/CUDA Graph|CUDA Graph]]（concept）
- [[飞书知识图谱/实体/DeepEP|DeepEP]]（concept）
- [[飞书知识图谱/实体/DeepSeek-R1-Channel-FP8|DeepSeek-R1-Channel-FP8]]（model）
- [[飞书知识图谱/实体/deepseek-r1-fp8|deepseek-r1-fp8]]（model）
- [[飞书知识图谱/实体/Docker|Docker]]（tool）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/ITL|ITL]]（metric）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/MoE|MoE]]（concept）
- [[飞书知识图谱/实体/Mooncake|Mooncake]]（framework）
- [[飞书知识图谱/实体/NCCL|NCCL]]（concept）
- [[飞书知识图谱/实体/OOM|OOM]]（issue）
- [[飞书知识图谱/实体/P50|P50]]（metric）
- [[飞书知识图谱/实体/P95|P95]]（metric）
- [[飞书知识图谱/实体/P99|P99]]（metric）
- [[飞书知识图谱/实体/PD分离|PD分离]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/Prometheus|Prometheus]]（tool）
- [[飞书知识图谱/实体/QPS|QPS]]（metric）
- [[飞书知识图谱/实体/RPS|RPS]]（metric）
- [[飞书知识图谱/实体/SGLang|SGLang]]（framework）
- [[飞书知识图谱/实体/Speculative Decoding|Speculative Decoding]]（concept）
- [[飞书知识图谱/实体/TBO|TBO]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/TPOT|TPOT]]（metric）
- [[飞书知识图谱/实体/TTFT|TTFT]]（metric）
- [[飞书知识图谱/实体/vLLM|vLLM]]（framework）

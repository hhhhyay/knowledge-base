---
title: 推理部署适配精度对齐
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: ZK5bwWQoMidtjTk7qlzcfMXnnue
feishu_type: DOCX
source_url: https://p283t9u4d9.feishu.cn/wiki/ZK5bwWQoMidtjTk7qlzcfMXnnue
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 推理部署适配精度对齐

[在飞书中打开](https://p283t9u4d9.feishu.cn/wiki/ZK5bwWQoMidtjTk7qlzcfMXnnue)

## 模型大小和路径

Neo++ Unify 9B

/mnt/afs/wupenghao/workspace/Neo_plus_train/RUN/d20260321_Neo_unify_9B_mot_newUnd_PT2_T2I_IT2I_res2k_lr1e-4_wd0_acc1_img128_seq16384_zs-1_wp8_sp1_gpu256/hf_step29000_ema

模型config

<view type="1">

  <file token="GAVtbRHv7oJpEbxXpimcLQ0DnJh" name="hf_step29000_ema.zip"/>

</view>

## 推理评测参考脚本
```markdown {wrap}
##概览
- 模型路径会按下面方式拼接：
`/mnt/afs/wangjiahao2/code/important/train_unify/Infer/inference_mot/<MODEL_NAME>/<STEP_NAME>hf`
（注意在脚本中修改对应路径）

- prompt 文件固定读取：
  `/mnt/afs/wupenghao/workspace/Neo_base/inference_mot/dpg_bench/prompts.jsonl`
（已贴在下方，注意修改脚本中的路径）

- 当前 `prompts.jsonl` 总数为 `1065`

- 输出图片目录固定为：
  `eval_output_<IMAGE_SIZE>/<MODEL_NAME>/<STEP_NAME>/dpgbench/`
- 评测结果文件固定为：
  `eval_output_<IMAGE_SIZE>/<MODEL_NAME>/<STEP_NAME>/dpgbench_results.txt`
  
- 图片文件名直接使用 prompt id，例如：
  `0.png`、`partiprompts0.png`、`COCOval2014000000010363.png`
  
- prompt id 不一定是纯数字，不能假设只会出现 `0/1/2/...`


##推理脚本说明
```bash
MODEL_NAME=neopp_mot \
STEP_NAME=pt2_it2i_32experts_norm_40000 \
IMAGE_SIZE=512 \
NUM_STEPS=50 \
CFG_SCALE=7.0 \
BATCH_SIZE=4 \
MICRO_BATCH_SIZE=4 \
COLLECT_EXPERT_USAGE=0 \
bash launch_dpgbench.sh
```

默认行为：

- 使用 `lmms_engine` 环境做生成，对应的requirement在下方。
- 使用 `dpgbench` bench做评测
- 默认 `CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7`
- 默认 `LAUNCH_MODE=device_map_multi`
- 默认每个 worker 吃 `2` 张卡，因此单机 `8` 卡会起 `4` 个 worker 分片生成
- `inference_dpgbench.py` 会把一个 prompt 生成的 `BATCH_SIZE` 张图拼成一张 grid 图保存。
- `compute_dpg_bench.py` 默认把每张输出图按 `2x2` 切成 4 个 crop 来打分。
- 因此标准评测配置下，`BATCH_SIZE` 必须和评测侧的 `pic-num` 对齐，当前默认就是 `4`。


```

### 相关脚本
<view type="2">

  <file token="IQY7bsJBLofyOpxaFD9cCXgwn3d" name="launch_dpgbench.sh"/>

</view>

<view type="2">

  <file token="DrQibxqy1o0BRXxjaXdcpYu8nZi" name="inference_dpgbench.py"/>

</view>

<view type="2">

  <file token="Lohrbx6X8oYCYhxxVwBcCR1Xn3g" name="compute_dpg_bench.py"/>

</view>

<view type="2">

  <file token="S5ndbjic2oImPvxIx2AcqO1cnqd" name="dpg_bench.csv"/>

</view>

<view type="1">

  <file token="ESvSbkioqoqGghxlND6cA5K5n7B" name="prompts.jsonl"/>

</view>

<view type="2">

  <file token="PbiVbfYmCoLACbxuAbtcTn5DnIe" name="requirements.txt"/>

</view>

## 对应结果
DPGBench:

<lark-table rows="2" cols="4" column-widths="199,100,122,100">

  <lark-tr>
    <lark-td rowspan="2">
      Neo++ Unify 9b
    </lark-td>
    <lark-td rowspan="2">
      PT2-144k
    </lark-td>
    <lark-td>
      512
    </lark-td>
    <lark-td>
      84.28
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      **1024**
    </lark-td>
    <lark-td>
      85.66
    </lark-td>
  </lark-tr>
</lark-table>

可视化例子：

<lark-table rows="4" cols="4" column-widths="149,336,110,325">

  <lark-tr>
    <lark-td>
      ID
    </lark-td>
    <lark-td>
      Prompt
    </lark-td>
    <lark-td>
      分辨率
    </lark-td>
    <lark-td>
      结果
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      0.png
    </lark-td>
    <lark-td>
      An expansive field, blanketed by the soft light of morning, cradles a collection of eight cabbages, their green heads round and plump. These vegetables are nestled among rows of rich soil, dotted with glistening droplets of dew that cling to their crinkled leaves. As wisps of mist begin to lift, the cabbages lie poised, ready for the day's impending harvest.
    </lark-td>
    <lark-td>
      1024*1024
    </lark-td>
    <lark-td>
      <image token="EmEIbde7GowDARxX87Mcs1SFned" width="2048" height="2048" align="center"/>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      diffusiondb0.png
    </lark-td>
    <lark-td>
      A visually striking digital portrayal of a futuristic woman, designed with an exquisite attention to detail that showcases her elegant pose, encased in an array of meticulously rendered leaves. Produced by the skilled artist Janice Sung, the image is a high-definition 4K creation that glows with stunning volumetric lighting effects. The woman's appearance is characterized by a level of hyper-realism that captures the intricate textures of her skin and the leaves, enhanced by the luminous, fantasy-inspired ambiance.
    </lark-td>
    <lark-td>
      1024*1024
    </lark-td>
    <lark-td>
      <image token="CU3DbYabHo1ohaxpOErcdxGyndg" width="2048" height="2048" align="center"/>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      drawtext0.png
    </lark-td>
    <lark-td>
      A visually striking isometric design spells out the word 'DRAW' using an array of artist pencils with softly rounded edges, demonstrating the principles of modular constructivism. Each pencil features a pastel color palette, blending harmoniously against a serene blue background. The entire composition benefits from soft, smooth lighting that accentuates the textures and forms, created with a physically based rendering technique that provides a realistic appearance, with the entire artwork centrally positioned within the frame, creating a trendy and aesthetically pleasing image.
    </lark-td>
    <lark-td>
      1024*1024
    </lark-td>
    <lark-td>
      <image token="XFu4bOvgcoLARvxcVgYcQDiGnrV" width="2048" height="2048" align="center"/>
    </lark-td>
  </lark-tr>
</lark-table>

###

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/DP|DP]]（concept）
- [[飞书知识图谱/实体/EP|EP]]（concept）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）

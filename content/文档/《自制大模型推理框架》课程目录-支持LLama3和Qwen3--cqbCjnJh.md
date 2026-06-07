---
title: 《自制大模型推理框架》课程目录-支持LLama3和Qwen3
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework:
  - PyTorch
issue_type: []
source: feishu
feishu_token: AGb0dpqwfohQ9oxx4QycqbCjnJh
feishu_type: DOCX
source_url: https://tvle9mq8jh.feishu.cn/docx/AGb0dpqwfohQ9oxx4QycqbCjnJh
last_synced_at: 2026-06-07T17:07:53.393Z
---

# 《自制大模型推理框架》课程目录-支持LLama3和Qwen3

[在飞书中打开](https://tvle9mq8jh.feishu.cn/docx/AGb0dpqwfohQ9oxx4QycqbCjnJh)

Hi，各位朋友们好！我是 KuiperInfe6r 的作者。KuiperInfer 作为一门开源课程，在 GitHub 上已斩获 2.8k star。

如今，在原课程的基础上，我们全新推出了《自制大模型推理框架》。这门课程不仅支持推理 Llama 系列模型，还支持 Cuda 加速和 Int8 量化课程，自推出以来便广受好评，同时课程也帮许许多多人找到了心仪的工作。

## <text bgcolor="light-yellow">**课程限时特惠-256**</text>

<image token="RvmCb1ERyoe6LhxelX6crMKEngf" width="905" height="1280" align="left"/>

## 如何报名课程

小程序自助下单后联系课程讲师

**小程序**

<image token="DjxmbbG4RoIvoWxPlvTcXE1hnae" width="472" height="462" align="left"/>

**课程讲师联系方式**

<image token="Pd9objHIPoLhAPxMaxQc6Vxrnjh" width="950" height="1296" align="left"/>

## 课程目录

课件请搭配视频一起学习，效果更好，每一节课程都录制了讲解视频。

<lark-table rows="24" cols="5" column-widths="100,198,231,100,149">

  <lark-tr>
    <lark-td>
      **课时** {align="center"}
    </lark-td>
    <lark-td>
      **课时名称**
    </lark-td>
    <lark-td>
      **学到的推理框架知识点** {align="center"}
    </lark-td>
    <lark-td>
      **能额外学到的知识** {align="center"}
    </lark-td>
    <lark-td>
      **备注** {align="center"}
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      0
    </lark-td>
    <lark-td>
      大模型优化综述
      **课件：**
      [[飞书知识图谱/资源/大模型推理框架优化技术概述-最终版--cREWXnid|大模型推理框架优化技术概述-最终版]]
    </lark-td>
    <lark-td>
      本次课程从<text bgcolor="light-yellow">算子优化、系统调度、解码、KV Cache和多机多卡并行出发</text>，分点总结大模型优化的方方面面。
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td rowspan="20">
      <text bgcolor="light-yellow">**本门课所有课时都配有视频和课件。**</text>
      <text bgcolor="light-yellow">**只要256元，包答疑。**</text>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      1 {align="center"}
    </lark-td>
    <lark-td>
      课程项目环境安装
      **课件：**
      [[飞书知识图谱/资源/第1课-环境安装--cVfrXnOb|第1课-环境安装]]
    </lark-td>
    <lark-td>
      必读
    </lark-td>
    <lark-td>
      <grid cols="2">
        <column width="50">
          
        </column>
        <column width="50">
          
        </column>
      </grid>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      2 {align="center"}
    </lark-td>
    <lark-td>
      内存的管理和设备类
      **课件：**[[飞书知识图谱/资源/第2次课程-内存的管理和设备类--cctydnvb|第2次课程-内存的管理和设备类]]
    </lark-td>
    <lark-td>
      1. 设备类管理硬件相关的操作，包括初始化硬件，管理硬件属性等，包括显存申请方法等。
			1. Buffer类封装内存管理，进行自动化内存生命周期管理。
			1. 统一CPU主存和GPU显存接口（申请、释放、拷贝）。
    </lark-td>
    <lark-td>
      1. C++类和对象相关知识，了解多态和封装是什么。
			1. C++中的析构函数、智能指针以及RAII等知识点。
			1. 类内方法的定义和重载。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      3  {align="center"}
    </lark-td>
    <lark-td>
      算子类的设计
      **课件：**[[飞书知识图谱/资源/第3次课程-算子类的设计--clfUXnvg|第3次课程-算子类的设计]]
    </lark-td>
    <lark-td>
      1. 如何在大型模型中设计算子层
			2. 如何在算子层中存储权重、输入及输出
			3. 如何设计算子调用计算过程的标准化接口
    </lark-td>
    <lark-td>
      1. C++中的get和set方法；
			1. 类内变量的存储机制；
			1. 用枚举类加强语义；
			1. 巩固同名函数重载的理解。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      4 {align="center"}
    </lark-td>
    <lark-td>
      张量的设计与实现
      **课件：**[[飞书知识图谱/资源/第4次课程-张量的设计与实现--ct3A3nNd|第4次课程-张量的设计与实现]]
    </lark-td>
    <lark-td>
      1. 怎么设计一个接口良好的多维矩阵类，接口包括读取，访问，写入等；
			1. 第2节中的内存管理类如何进行联动，使得张量底层的显存/内存资源能被自动管理；
			1. 如何设计工具方法(function)来方便对张量实例中数据的访问。
    </lark-td>
    <lark-td>
      1. 继续巩固智能指针相关的知识；
			1. 用reinterpreter_cast替代c风格的强制转换；
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      5 {align="center"}
    </lark-td>
    <lark-td>
      RMSNorm算子的Cuda实现
      **课件：**[[飞书知识图谱/资源/第5次课程-RMSNorm算子的CUDA实现--cw2HXnxb|第5次课程-RMSNorm算子的CUDA实现]]
    </lark-td>
    <lark-td>
      1. Cuda基础，Thread和Block的定义；
			1. RMSNorm算子的公式讲解；
			1. RMSNorm算子的CUDA和CPU实现；
			1. CUDA中的块内规约实现原理(BlockReduce)。
    </lark-td>
    <lark-td>
      1. 函数指针的定义；
			1. 怎么让一个函数指针指向两个不同的函数；
			1. 除了用Cuda之外，还可以学会怎么用C++实现算子。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      6 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      Nsight调优Cuda算子
      **课件：**[[飞书知识图谱/资源/第6次课程-Nsight compute的使用和算子的优化--c6TUcnJh|第6次课程-Nsight compute的使用和算子的优化]]
    </lark-td>
    <lark-td>
      1. 让更多的Cuda线程来参与归约计算的过程；
			1. 纠正第5次课程cuda代码中的一些纰漏；
			1. 利用Nsight软件的分析结果指导Cuda代码的编写和调优。
    </lark-td>
    <lark-td>
      1. Nvidia Nsight的基本使用；
			1. Nsight分析报告上的参数分析；
			1. 根据分析报告找出核函数实现可能的瓶颈。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      7 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      LLama模型的量化
      **课件：**[[飞书知识图谱/资源/第7次课程-模型的量化--c9CuSnbf|第7次课程-模型的量化]]
    </lark-td>
    <lark-td>
      1. 怎么导出量化后的LLama模型；
			1. 加载LLama量化模型和非量化模型有什么异同；
			1. 怎么利用量化系数反推得到量化前的浮点权重。
    </lark-td>
    <lark-td>
      量化相关的知识学习：
      1. 量化是什么；
			1. 量化系数有什么用；
			1. 怎么利用量化公式去求出量化系数。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      8 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      Cuda的向量化存取
      **课件：**[[飞书知识图谱/资源/第8次课程-Cuda的向量化存取--ckx2Yneh|第8次课程-Cuda的向量化存取]]
    </lark-td>
    <lark-td>
      1. 回顾Warp范围内的Reduce和线程块范围(Block)Reduce；
			1. 利用向量化存取技术继续优化Cuda算子
    </lark-td>
    <lark-td>
      巩固以下的知识点：
      1. Nvidia Nsight的基本使用；
			1. Nsight分析报告上的参数分析；
			1. 根据分析报告找出核函数实现可能的瓶颈。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      9 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      显存的管理
      **课件：**[[飞书知识图谱/资源/第9次课程-自制大模型推理框架-显存的管理--c3LDIn4f|第9次课程-自制大模型推理框架-显存的管理]]
    </lark-td>
    <lark-td>
      1. 显存的管理形式，对大块显存和小块显存的申请分别进行优化；
			1. 从cpu到cuda显存的互相转换，如何将cpu上的数据同步到gpu上。
    </lark-td>
    <lark-td>
      1. 巩固第二次课程中的C++类和对象相关知识，多态和封装等知识；
			1. 如果使用STL库中的map结构来管理显存。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      10 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      模型权重的载入
      **课件：**[[飞书知识图谱/资源/第10课-自制大模型推理框架-mmap映射模型文件--crKJMndd|第10课-自制大模型推理框架-mmap映射模型文件]]
    </lark-td>
    <lark-td>
      1. 回顾模型文件中的配置信息和权重排布；
			1. 怎么用Mmap快速打开超大的大模型权重文件并能定位到其中的权重起始位置。
    </lark-td>
    <lark-td>
      1. 熟悉系统接口Mmap，了解文件读取时内核所起到的作用；
			1. 掌握进程内存和文件之间的映射原理。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      11 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      带权重算子的初始化
      **课件：**[[飞书知识图谱/资源/第11课-自制大模型推理框架-算子层的创建和权重的载入--crR1Jnig|第11课-自制大模型推理框架-算子层的创建和权重的载入]]
    </lark-td>
    <lark-td>
      1. 模型权重和带权重算子类的联动；
			1. 如何将映射到进程中的大模型权重数据赋值给算子，以供下一步计算使用。
    </lark-td>
    <lark-td>
      1. 掌握STL标准库中vector类的使用；
			1. 掌握类的构造函数和赋值函数(set 和 get方法）。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      12 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      算子计算后端的选择
      **课件：**[[飞书知识图谱/资源/第12课-自制大模型推理框架-算子后端的选择--cW3YTnwd|第12课-自制大模型推理框架-算子后端的选择]]
    </lark-td>
    <lark-td>
      我们实现了两套算子(分别是cpu和gpu上的)。
      本次课程我们将讲授如何根据设备类型选择不同的算子计算后端
    </lark-td>
    <lark-td>
      继续巩固第5次课程中关于
      1. 函数指针的定义；
			1. 怎么让一个函数指针指向两个不同的函数；
      这两点的知识。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      13 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      GEMV算子的CPU实现
      **课件：**[[飞书知识图谱/资源/自制大模型推理框架-第13课-Sgemv算子的实现-cpu--ceokBnTf|自制大模型推理框架-第13课-Sgemv算子的实现-cpu]]
    </lark-td>
    <lark-td>
      1. Pytorch中的nn.Linear层原理；
			1. 在CPU设备上动手实现一个高效的GEMV运算，并在运算结果上与Pytorch对齐。
    </lark-td>
    <lark-td>
      1. 主流数学库Armadillo的使用；
			1. Armdillo中的常见数据结构，向量、矩阵等；
			1. Armadilo数学库中的乘、加、矩阵乘等运算。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      14 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      GEMV算子的GPU实现
      **课件：**[[飞书知识图谱/资源/自制大模型推理框架-第13课-Sgemv算子的实现-cuda--c7MgPnog|自制大模型推理框架-第13课-Sgemv算子的实现-cuda]]
      量化实现课件：
    </lark-td>
    <lark-td>
      1. 在GPU设备上用CUDA动手实现一个高效的GEMV运算，并在运算结果上与Pytorch对齐。
			1. 高性能Cuda编程知识的再一次串联，规约、向量化、循环展开等。
    </lark-td>
    <lark-td>
      本次课程主要是学习高性能cuda相关的知识。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      15 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      KVCache的原理与实现
      **课件：**[[飞书知识图谱/资源/自制大模型推理框架-第15次课程-KVCache的原理和实现--cZrZMnPe|自制大模型推理框架-第15次课程-KVCache的原理和实现]]
    </lark-td>
    <lark-td>
      1. 从算法的角度讲授自注意力是什么；
			1. q，k，v这三个矩阵的计算方式，以及kv cache的存储方式；
			1. 自注意力的计算方式，包括如何节省算力，避免重复计算。
    </lark-td>
    <lark-td>
      1. KVCache机制是什么；
			1. KVCache为什么能节省显存，怎么节省的；
			1. KVCache和自注意力机制计算之间的关系，两者怎么融合。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      16 {align="center"}
    </lark-td>
    <lark-td>
      KV Cache的实现和自注意力计算中的QKV
      **课件：**[[飞书知识图谱/资源/KV Cache的实现和自注意力计算中的QKV--cKRY1nQw|KV Cache的实现和自注意力计算中的QKV]]
    </lark-td>
    <lark-td>
      1. 巩固上节课关于KVCache的内容；
			1. 学习怎么获取到KV Cache中的Query，Key和Value矩阵，以及自注意力模块中多个算子的计算流程。
    </lark-td>
    <lark-td>
      1. 怎么对小算子的计算过程进行组合得到一个更大的算子；
			1. 从本节课开始我们将学习，LLama模型的整体结构，帮助同学们补充大模型知识。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      17 {align="center"}
    </lark-td>
    <lark-td>
      MLP算子的实现
      **课件：**[[飞书知识图谱/资源/Llama模型中的MLP层实现--cwxxMnnf|Llama模型中的MLP层实现]]
    </lark-td>
    <lark-td>
      1. 前馈神经网络层的实现，了解其中的线性映射和非线性激活的流程；
			1. 怎么用Cuda实现swiglu算子，如何将多个Cuda计算过程进行组合，实现MLP模块。
    </lark-td>
    <lark-td>
      1. 从算法层面继续分析LLama模型，这节课是对MLP模块的分析；
			1. 回顾算子注册和调用的流程。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      18 {align="center"}
    </lark-td>
    <lark-td>
      多头自注意力机制的实现
      **课件：**[[飞书知识图谱/资源/第18次课程-用Cuda实现LLama模型中多头注意力算子--cNNBtn1f|第18次课程-用Cuda实现LLama模型中多头注意力算子]]
    </lark-td>
    <lark-td>
      1. 多头自注意力的计算方式解析；
			1. 在计算注意力时的维度输入和输出维度，以及各自的含义；
			1. 用Cuda实现自注意力值的求解过程。
    </lark-td>
    <lark-td>
      从算法层面分析什么是单头自注意力机制，什么是多头自注意力机制，以及它们各自的计算方法。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      19 {align="center"}
    </lark-td>
    <lark-td>
      LLama模型中其余算子的Cuda实现
      **课件：**[[飞书知识图谱/资源/第19次课程-llama模型中的其余算子实现--cZF5pnje|第19次课程-llama模型中的其余算子实现]]
    </lark-td>
    <lark-td>
      1. 多头自注意力算子中softmax 的Cuda算子实现；
			1. nn.Embedding算子的Cuda实现；
			1. 其余LLama模型所需Cuda算子的实现。
    </lark-td>
    <lark-td>
      1. 我们将继续从算法的角度深入剖析LLama大型模型的各个组件；
			1. 在本节课结束时，我们将确保所有所需的算子都已准备就绪。因此，接下来我们将探讨推理过程中的数据流动情况。
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      20 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      LLama 3.2模型的支持
      **课件：**[[飞书知识图谱/资源/自制大模型推理框架-第20次课程-LLama3.2模型的支持--c3bVInde|自制大模型推理框架-第20次课程-LLama3.2模型的支持]]
    </lark-td>
    <lark-td>
      1. LLama3与LLama2模型之间的主要区别在于分词器的改进以及窗口大小的调整等方面。
			1. BPE（Byte Pair Encoding）算法的核心原理及其具体实现方式。
			1. 如何通过修改配置文件来切换至LLama3的开发环境。
    </lark-td>
    <lark-td>
      在对比LLama2/3的差异时，重点分析了不同参数量的模型适用于哪些特定场景，并解释了为何选择LLama3.2作为我们的教学版本。
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      21 {align="center"}
       {align="center"}
    </lark-td>
    <lark-td>
      课程项目Demo，用课程大模型推理框架跑LLama3和Qwen2.5
      **课件：**[[飞书知识图谱/资源/第21次课程-用课程大模型推理框架跑LLama3和Qwen2.5--cSrM8nF2|第21次课程-用课程大模型推理框架跑LLama3和Qwen2.5]]
    </lark-td>
    <lark-td>
      1. 从输入字符串到最终生成结果，需要历经几个过程，其中包含输入数据的流动、变换等情况。
			1. 在大模型推理过程中，prompt 阶段和 generate 阶段分别是什么？在这两个阶段需要进行哪些特殊处理？
			1. 如何进行采样以根据概率得到最终的词元输出？
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      22 {align="center"}
    </lark-td>
    <lark-td>
      Qwen3模型的支持
      **课件：**[[飞书知识图谱/资源/第22课-《cuda自制大模型推理框架》课程中支持qwen3--cHSisn9W|第22课-《cuda自制大模型推理框架》课程中支持qwen3]]
    </lark-td>
    <lark-td>
      1. 支持Qwen3模型
			1. Qwen3模型在模型结构上的差异
			1. 怎么支持思维链(think)
    </lark-td>
    <lark-td>
    </lark-td>
    <lark-td>
    </lark-td>
  </lark-tr>
</lark-table>

## 关联资源

- [[飞书知识图谱/资源/大模型推理框架优化技术概述-最终版--cREWXnid|大模型推理框架优化技术概述-最终版]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第1课-环境安装--cVfrXnOb|第1课-环境安装]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第2次课程-内存的管理和设备类--cctydnvb|第2次课程-内存的管理和设备类]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第3次课程-算子类的设计--clfUXnvg|第3次课程-算子类的设计]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第4次课程-张量的设计与实现--ct3A3nNd|第4次课程-张量的设计与实现]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第5次课程-RMSNorm算子的CUDA实现--cw2HXnxb|第5次课程-RMSNorm算子的CUDA实现]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第6次课程-Nsight compute的使用和算子的优化--c6TUcnJh|第6次课程-Nsight compute的使用和算子的优化]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第7次课程-模型的量化--c9CuSnbf|第7次课程-模型的量化]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第8次课程-Cuda的向量化存取--ckx2Yneh|第8次课程-Cuda的向量化存取]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第9次课程-自制大模型推理框架-显存的管理--c3LDIn4f|第9次课程-自制大模型推理框架-显存的管理]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第10课-自制大模型推理框架-mmap映射模型文件--crKJMndd|第10课-自制大模型推理框架-mmap映射模型文件]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第11课-自制大模型推理框架-算子层的创建和权重的载入--crR1Jnig|第11课-自制大模型推理框架-算子层的创建和权重的载入]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第12课-自制大模型推理框架-算子后端的选择--cW3YTnwd|第12课-自制大模型推理框架-算子后端的选择]] · 飞书源链接不可用
- [[飞书知识图谱/资源/自制大模型推理框架-第13课-Sgemv算子的实现-cpu--ceokBnTf|自制大模型推理框架-第13课-Sgemv算子的实现-cpu]] · 飞书源链接不可用
- [[飞书知识图谱/资源/自制大模型推理框架-第13课-Sgemv算子的实现-cuda--c7MgPnog|自制大模型推理框架-第13课-Sgemv算子的实现-cuda]] · 飞书源链接不可用
- [[飞书知识图谱/资源/自制大模型推理框架-第15次课程-KVCache的原理和实现--cZrZMnPe|自制大模型推理框架-第15次课程-KVCache的原理和实现]] · 飞书源链接不可用
- [[飞书知识图谱/资源/KV Cache的实现和自注意力计算中的QKV--cKRY1nQw|KV Cache的实现和自注意力计算中的QKV]] · 飞书源链接不可用
- [[飞书知识图谱/资源/Llama模型中的MLP层实现--cwxxMnnf|Llama模型中的MLP层实现]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第18次课程-用Cuda实现LLama模型中多头注意力算子--cNNBtn1f|第18次课程-用Cuda实现LLama模型中多头注意力算子]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第19次课程-llama模型中的其余算子实现--cZF5pnje|第19次课程-llama模型中的其余算子实现]] · 飞书源链接不可用
- [[飞书知识图谱/资源/自制大模型推理框架-第20次课程-LLama3.2模型的支持--c3bVInde|自制大模型推理框架-第20次课程-LLama3.2模型的支持]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第21次课程-用课程大模型推理框架跑LLama3和Qwen2.5--cSrM8nF2|第21次课程-用课程大模型推理框架跑LLama3和Qwen2.5]] · 飞书源链接不可用
- [[飞书知识图谱/资源/第22课-《cuda自制大模型推理框架》课程中支持qwen3--cHSisn9W|第22课-《cuda自制大模型推理框架》课程中支持qwen3]] · 飞书源链接不可用

## 关联知识

- [[飞书知识图谱/实体/CP|CP]]（concept）
- [[飞书知识图谱/实体/KV Cache|KV Cache]]（concept）
- [[飞书知识图谱/实体/PyTorch|PyTorch]]（framework）

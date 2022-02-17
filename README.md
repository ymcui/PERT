[**中文**](https://github.com/ymcui/PERT/README.md) | [**English**](https://github.com/ymcui/PERT/blob/main/README_EN.md)

<p align="center">
    <br>
    <img src="./pics/banner.png" width="500"/>
    <br>
</p>
<p align="center">
    <a href="https://github.com/ymcui/PERT/blob/master/LICENSE">
        <img alt="GitHub" src="https://img.shields.io/github/license/ymcui/PERT.svg?color=blue&style=flat-square">
    </a>
</p>
在自然语言处理领域中，预训练语言模型（Pre-trained Language Models，PLMs）已成为非常重要的基础技术。在近两年，哈工大讯飞联合实验室发布了多种中文预训练模型资源以及相关配套工具。作为相关工作的延续，在本项目中，我们提出了一种 **基于乱序语言模型的预训练模型（PERT）**，在不引入掩码标记[MASK]的情况下自监督地学习文本语义信息。*PERT在部分中英文NLU任务上获得性能提升，但也在部分任务上效果较差，请酌情使用。*目前提供了中文和英文的PERT模型，包含两种模型大小（base、large）。

论文：**PERT: Pre-Training BERT with Permuted Language Model（待后续上传）**   
作者：Yiming Cui, Ziqing Yang, Ting Liu, Zhigang Chen

其他资源：

[中文MacBERT](https://github.com/ymcui/MacBERT) | [中文ELECTRA](https://github.com/ymcui/Chinese-ELECTRA) | [中文XLNet](https://github.com/ymcui/Chinese-XLNet) | [中文BERT](https://github.com/ymcui/Chinese-BERT-wwm) |  [知识蒸馏工具TextBrewer](https://github.com/airaria/TextBrewer) | [模型裁剪工具TextPruner](https://github.com/airaria/TextPruner)

查看更多哈工大讯飞联合实验室（HFL）发布的资源：https://github.com/ymcui/HFL-Anthology

## 新闻
**2022/2/17   感谢对本项目的关注，预计下周发出模型，技术报告待完善后发出。**

## 内容导引
| 章节                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| [简介](#简介)                         | PERT预训练模型的基本原理                                       |
| [模型下载](#模型下载)         | PERT预训练模型的下载地址                               |
| [快速加载](#快速加载)                 | 如何使用[🤗Transformers](https://github.com/huggingface/transformers)快速加载模型 |
| [基线系统效果](#基线系统效果) | 在部分NLU任务上的基线系统效果                                |
| [FAQ](#FAQ)                           | 常见问题答疑                                                 |
| [引用](#引用)                         | 本项目的技术报告                                          |


## 简介
面向自然语言理解（NLU）的预训练模型的学习大致分为两类：使用和不使用带掩码标记[MASK]的输入文本。

算法启发：一定程度的乱序文本不影响理解。

大体思想：PERT对原始输入文本进行一定的词序调换，从而形成乱序文本（因此不会引入额外的[MASK]标记）。PERT的学习目标是预测原token所在的位置，具体见下例。


| 说明            | 输入文本                                                     | 输出目标                                                     |
| :-------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 原始文本        | 研究表明这一句话的顺序并不影响阅读。                         | -                                                            |
| WordPiece分词后 | 研 究 表 明 这 一 句 话 的 顺 序 并 不 影 响 阅 读 。        | -                                                            |
| BERT            | 研 究 表 明 这 一 句 **[MASK]** 的 顺 **[MASK]** 并 不 **[MASK]** 响 阅 读 。 | 位置7 → 话<br/>位置10 → 序<br/>位置13 → 影                   |
| PERT            | 研 究 **明** **表** 这 一 句 话 的 顺 序 并 不 **响** **影** 阅 读 。 | 位置2（明）→位置3（表）<br/>位置3（表）→位置2（明）<br/>位置13（响）→位置14（影）<br/>位置14（影）→位置13（响） |


更详细的算法细节待后续补充说明。

## 模型下载

#### 原版下载地址

这里主要提供TensorFlow 1.15版本的模型权重。如需PyTorch或者TensorFlow2版本的模型，请看下一小节。

* **`PERT-large`**：24-layer, 1024-hidden, 16-heads, 330M parameters
* **`PERT-base`** 12-layer, 768-hidden, 12-heads, 110M parameters

| 模型简称                 | 语种 |          语料           | Google下载 | 百度盘下载 |
| :----------------------- | :--: | :---------------------: | :--------: | :--------: |
| **`Chinese-PERT-large`** | 中文 |  EXT数据<sup>[1]</sup>  |     -      |     -      |
| **`Chinese-PERT-base`**  | 中文 |  EXT数据<sup>[1]</sup>  |     -      |     -      |
| **`English-PERT-large`** | 英文 | WikiBooks<sup>[2]</sup> |     -      |     -      |
| **`English-PERT-base`**  | 英文 | WikiBooks<sup>[2]</sup> |     -      |     -      |

> [1] EXT数据包括：中文维基百科，其他百科、新闻、问答等数据，总词数达5.4B，约占用20G磁盘空间，与MacBERT相同。
>
> [2] Wikipedia + BookCorpus

以TensorFlow版`Chinese-PERT-base`为例，下载完毕后对zip文件进行解压得到：

```
chinese_pert_base_L-12_H-768_A-12.zip
    |- pert_model.ckpt      # 模型权重
    |- pert_model.meta      # 模型meta信息
    |- pert_model.index     # 模型index信息
    |- pert_config.json     # 模型参数
    |- vocab.txt            # 词表（与谷歌原版一致）
```

其中`bert_config.json`和`vocab.txt`与谷歌原版`BERT-base, Chinese`完全一致。

#### PyTorch以及TensorFlow 2版本

通过🤗transformers模型库可以下载TensorFlow (v2)和PyTorch版本模型。

下载方法：点击任意需要下载的模型 → 选择"Files and versions"选项卡 → 下载对应的模型文件。


## 快速加载
由于PERT主体部分仍然是BERT结构，用户可以使用[transformers库](https://github.com/huggingface/transformers)轻松调用PERT模型。

**注意：本目录中的所有模型均使用BertTokenizer以及BertModel加载。**

```python
from transformers import BertTokenizer, BertModel

tokenizer = BertTokenizer.from_pretrained("MODEL_NAME")
model = BertModel.from_pretrained("MODEL_NAME")
```
其中`MODEL_NAME`对应列表如下：

| 模型名             | MODEL_NAME             |
| ------------------ | ---------------------- |
| Chinese-PERT-large | hfl/chinese-pert-large |
| Chinese-PERT-base  | hfl/chinese-pert-base  |
| English-PERT-large | hfl/english-pert-large |
| English-PERT-base  | hfl/english-pert-base  |

## 基线系统效果
TBA


## FAQ
TBA


## 引用
TBA


## 关注我们
欢迎关注哈工大讯飞联合实验室官方微信公众号，了解最新的技术动态。

![qrcode.png](https://github.com/ymcui/cmrc2019/raw/master/qrcode.jpg)


## 问题反馈
如有问题，请在GitHub Issue中提交。

- 在提交问题之前，请先查看FAQ能否解决问题，同时建议查阅以往的issue是否能解决你的问题。
- 重复以及与本项目无关的issue会被[stable-bot](stale · GitHub Marketplace)处理，敬请谅解。
- 我们会尽可能的解答你的问题，但无法保证你的问题一定会被解答。
- 礼貌地提出问题，构建和谐的讨论社区。
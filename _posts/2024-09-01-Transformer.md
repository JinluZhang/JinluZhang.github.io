---
layout: post
title: 'Transformer学习笔记'
date: 2024-09-01
categories: 技术
author: Beaug
tags: Transformer 大模型
---




休假在家训模型玩儿，顺便把实践过程整理成文档。

本文训练了一个英德互译的模型，模型架构忠实复现Transformer论文（[Attention Is All You Need](https://arxiv.org/pdf/1706.03762)
）,代码来自 https://github.com/harvardnlp/annotated-transformer。

文章用三万组英-德语料训练了一个翻译模型，并以“Three dogs are playing in the water.” 这句英文如何翻译成德文 “Drei Hunde spielen im Wasser.”为例子，拆解了token化、encoder、decoder的完整流程。


# 模型整体结构





# 训练语料
使用的语料是torchtext.datasets.Multi30k，划分好的数据集大小为：

| 数据集类型   | 数据集大小 |
|:-----------|:----------|
| train   |   29000 |
| valid   |   1014 |
| test   |   1000 |

以valid数据集为例，有val.de和val.en两文件，内容示例如下：
```
Eine Gruppe von Männern lädt Baumwolle auf einen Lastwagen
Ein Mann schläft in einem grünen Raum auf einem Sofa.
Ein Junge mit Kopfhörern sitzt auf den Schultern einer Frau.
```
```
A group of men are loading cotton onto a truck
A man sleeping in a green room on a couch.
A boy wearing headphones sits on a woman's shoulders.
```

使用代码示例：

```python
def build_vocabulary(spacy_de, spacy_en):
    def tokenize_de(text):
        return tokenize(text, spacy_de)

    def tokenize_en(text):
        return tokenize(text, spacy_en)

    print("Building German Vocabulary ...")
    train, val, test = datasets.Multi30k(language_pair=("de", "en"))
    vocab_src = build_vocab_from_iterator(
        yield_tokens(train + val + test, tokenize_de, index=0),
        min_freq=2,
        specials=["<s>", "</s>", "<blank>", "<unk>"],
    )

    print("Building English Vocabulary ...")
    train, val, test = datasets.Multi30k(language_pair=("de", "en"))
    vocab_tgt = build_vocab_from_iterator(
        yield_tokens(train + val + test, tokenize_en, index=1),
        min_freq=2,
        specials=["<s>", "</s>", "<blank>", "<unk>"],
    )

    vocab_src.set_default_index(vocab_src["<unk>"])
    vocab_tgt.set_default_index(vocab_tgt["<unk>"])

    return vocab_src, vocab_tgt

def yield_tokens(data_iter, tokenizer, index):
    for from_to_tuple in data_iter:
        yield tokenizer(from_to_tuple[index])


```


torchtext.datasets支持的数据集还有：
* Language modeling: WikiText2, WikiText103, PennTreebank, EnWik9
* Machine translation: IWSLT2016, IWSLT2017, Multi30k
* Sequence tagging (e.g. POS/NER): UDPOS, CoNLL2000Chunking
* Question answering: SQuAD1, SQuAD2
* Text classification: SST2, AG_NEWS, SogouNews, DBpedia, YelpReviewPolarity, YelpReviewFull, YahooAnswers, AmazonReviewPolarity, AmazonReviewFull, IMDB
* Model pre-training: CC-100


 <font color=green>（application定义一致，几十万对应我们SOA中的服务个数）</font> 



# token化

tokenize


The cluster management system we internally call Borg admits, schedules, starts, restarts, and monitors the full range of applications that Google runs. This paper explains how.

我们内部称为Borg的集群管理系统负责接收、调度、启动、重启和监控Google所有的应用。本文介绍它是如何实现的。

Borg provides three main benefits: it (1) hides the details of resource management and failure handling so its users can focus on application development instead; (2) operates with very high reliability and availability, and supports applications that do the same; and (3) lets us run workloads across tens of thousands of machines effectively. Borg is not the first system to address these issues, but it’s one of the few operating at this scale, with this degree of resiliency and completeness. This paper is organized around these topics, concluding with a set of qualitative observations we have made from operating Borg in production for more than a decade.

Borg提供了三个主要的好处：（1）隐藏资源管理和故障处理细节，使用户可以专注于应用开发；（2）高可靠和高可用的运维，并支持应用程序也能够如此；（3）让我们可以在几万台机器上高效地运行负载。Borg不是第一个涉及这些问题的系统，但它是少有的运行在如此大规模，具有弹性且完整的系统之一。本文围绕这些主题来编写，总结了十多年来我们在生产环境运行Borg的一些定性观察。

![Figure_1_architecture_of_Borg.png](http://jinluzhang.github.io/assets/posts_img/2019-07-BORG-img/Figure_1_architecture_of_Borg.png)

​        图1. Borg的架构。图中只画出了数千个工作节点的很小一部分

# 2. 用户视角 **The user perspective**

Borg’s users are Google developers and system administrators (site reliability engineers or SREs) that run Google’s applications and services. Users submit their work to Borg in the form of *jobs*, each of which consists of one or more *tasks* that all run the same program (binary). Each job runs in one Borg *cell*, a set of machines that are managed as a unit. The remainder of this section describes the main features exposed in the user view of Borg.

Borg的用户是Google的RD和SRE。用户以作业*Jobs*的方式将他们的工作提交给Borg。作业job由一个或多个任务*Tasks*组成，每个task执行相同的二进制程序 <font color=green>（task间互为replica）</font>。每个作业job都运行在一个Borg单元*Cell*里。Cell是一组机器的管理单元。下面的小节将介绍用户视角看到的Borg系统的主要特性。




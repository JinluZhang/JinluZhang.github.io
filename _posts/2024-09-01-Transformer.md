---
layout: post
title: 'Transformer学习笔记'
date: 2024-09-01
categories: 技术
author: Beaug
tags: Transformer 大模型 annotated-transformer
---




休假在家训模型玩儿，顺便把实践过程整理成文档。

本文训练了一个英德互译的模型，模型架构忠实复现Transformer论文（[Attention Is All You Need](https://arxiv.org/pdf/1706.03762)
）,代码来自 https://github.com/harvardnlp/annotated-transformer。

文章用三万组英-德语料训练了一个翻译模型，并以“Three dogs are playing in the water.” 这句英文如何翻译成德文 “Drei Hunde spielen im Wasser.”为例子，拆解了token化、encoder、decoder的完整流程。


# 模型整体结构





# 训练语料
使用的语料是torchtext.datasets.Multi30k，这是一个有三万多组【德语-英语】语料对的数据集，可用于训练机器翻译模型。

划分好的数据集大小为：

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

使用代码示例，调用datasets.Multi30k(language_pair=("de", "en"))函数后，返回ShardingFilterIterDataPipe对象:

```python
train, val, test = datasets.Multi30k(language_pair=("de", "en"))

```

torchtext.datasets支持的数据集还有：
* Language modeling: WikiText2, WikiText103, PennTreebank, EnWik9
* Machine translation: IWSLT2016, IWSLT2017, Multi30k
* Sequence tagging (e.g. POS/NER): UDPOS, CoNLL2000Chunking
* Question answering: SQuAD1, SQuAD2
* Text classification: SST2, AG_NEWS, SogouNews, DBpedia, YelpReviewPolarity, YelpReviewFull, YahooAnswers, AmazonReviewPolarity, AmazonReviewFull, IMDB
* Model pre-training: CC-100


# token化

tokenizer使用的是spacy库提供的en_core_web_sm、de_core_news_sm，分词实现比较简单，基本只用空格隔开了单词，如果处理海量训练语料的话词表会爆炸。除分词外，还支持分词、词性标注、命名实体识别和依存句法分析等功能。

生产环境可用的token库可参考llama使用的SentencePiece，和HuggingFace封装的版本https://github.com/huggingface/tokenizers

分词、词性标注、命名实体识别和依存句法分析等。

```python
def load_tokenizers():

    try:
        spacy_de = spacy.load("de_core_news_sm")
        print(type(spacy_de))
    except IOError:
        os.system("python -m spacy download de_core_news_sm")
        spacy_de = spacy.load("de_core_news_sm")

    try:
        spacy_en = spacy.load("en_core_web_sm")
    except IOError:
        os.system("python -m spacy download en_core_web_sm")
        spacy_en = spacy.load("en_core_web_sm")

    return spacy_de, spacy_en

def tokenize(text, tokenizer):
    return [tok.text for tok in tokenizer.tokenizer(text)]

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

使用spacy_en进行token化后的结果示例如下，"Three dogs are playing in the water."这句话会被分割成8个token。

```
>>> doc=spacy_en( "Three dogs are playing in the water." )
>>> for token in doc:
...     print(token.text, token.pos_, token.dep_)
...
Three NUM nummod
dogs NOUN nsubj
are AUX aux
playing VERB ROOT
in ADP prep
the DET det
water NOUN pobj
. PUNCT punct
```
vocab是token化后形成的词表，token化在下节展开，此处先看下







Borg的用户是Google的RD和SRE。用户以作业*Jobs*的方式将他们的工作提交给Borg。作业job由一个或多个任务*Tasks*组成，每个task执行相同的二进制程序 <font color=green>（task间互为replica）</font>。每个作业job都运行在一个Borg单元*Cell*里。Cell是一组机器的管理单元。下面的小节将介绍用户视角看到的Borg系统的主要特性。




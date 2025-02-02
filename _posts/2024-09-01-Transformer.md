---
title: 'Transformer代码实现详解'
date: 2024-09-01
categories: 技术
author: Beaug
tags: Transformer 大模型 annotated-transformer
---



休假在家训模型玩儿，顺便把实践过程整理成文档。

本文训练了一个英德互译的模型，模型架构忠实复现Transformer论文（[Attention Is All You Need](https://arxiv.org/pdf/1706.03762)
）,代码来自 https://github.com/harvardnlp/annotated-transformer。

文章用三万组英-德语料训练了一个翻译模型，并基于具体案例拆解了token化、encoder、decoder的完整流程。

看完文章将了解：
1.一句德文输入，在各个中间环节的具体内容，及含义
2.在这个完整例子中，Transformer到底有多少个参数




# 1.数据准备

## 1.1.训练语料

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

## 1.2.训练数据处理

为了说清楚构造流程，先将batch_size设置成4，max_padding设置成18，使得数据小一些，方便看怎么构造的mask和target。

第一个训练Batch的原始数据如下，encoder的输入是德文串，希望最终decoder的输出是对应的英文串。
```python
[
    ('Ein Mann und eine Frau sprechen vor einem Kaffeehaus miteinander.',
        'A man and a woman tak outside of a coffee house.'),
    ('Eine Frau in einem gepunkteten T-Shirt wartet einen Moment ab, in dem sie die Straße gefahrlos überqueren kann.',
        'A woman in a polka dotted shirt waits for a safe moment to cross the street.'),
    ('Ein kleines grünes Auto, auf dessen Tür die Nummer\xa063 steht, fährt auf einer Rennstrecke.',
        'A small green car with the number 63 on the door races on a track.'),
    ('Ein Mann blick mit seinem Sohn durch ein Teleskop.',
        'A man looking through a telescope with his son.')
]
```

经过token化及padding后，输入数据会被处理成如下形式：
```python
0 = {Tensor: (18,)} tensor([   0,    5,   12,    8,   18,   17,  617,   28,    6, 6437,  299,    4,    1,    2,    2,    2,    2,    2])
1 = {Tensor: (18,)} tensor([   0,   14,   17,    7,    6, 1673,  113,  245,   20, 1635,  291,    9,    7,   26,  111,   19,   36,    3])
2 = {Tensor: (18,)} tensor([   0,    5,   68,  758,  213,    9,   11,  747,  562,   19,  471,  675, 5770,   30,    9,   64,   11,   13])
3 = {Tensor: (18,)} tensor([   0,    5,   12, 7538,   10,   79,  633,   59,   15, 1077,    4,    1,    2,    2,    2,    2,    2,    2])
```
- 首先，会分词后逐一查词表，比如Ein在词表列表中是第6个token，则token化后的内容为“5”，即它在词表中的index值。第一句德文`'Ein Mann und eine Frau sprechen vor einem Kaffeehaus miteinander.'` 查找词表后，会得到11个token id，内容为`5,   12,    8,   18,   17,  617,   28,    6, 6437,  299,    4`
- 注意到每行第一个token都是“0”，这是“<s>”的token id，代表一句话的开始。一句话结束后，会在末尾添加“</s>”的token id “1”。例子中，第一句德文和第四句德文比较短，所以添加上了“1”；第二句和第三句德文比较长，做了截断处理。
- 当token化后整个句子长度不足18，会用pad_id补全到18，即第一句话末尾的5个“2”。此处`pad_id=vocab_src.get_stoi()["<blank>"]`

类似的处理方式，最终希望decoder输出的英文串，会被处理成如下形式：
```python
0 = {Tensor: (18,)} tensor([  0,   6,  12,  11,   4,   16,    3,  58,   13,   4, 574,  358,    5,   1,    2,   2,   2,   2])
1 = {Tensor: (18,)} tensor([  0,   6,  16,   7,   4, 1254, 2434,  23,  708,  55,   4, 3887, 1463,  18,  484,   8,  40,   5])
2 = {Tensor: (18,)} tensor([  0,   6,  77,  53, 142,   14,    8, 459, 4885,   9,   8,  533, 1751,   9,    4, 307,   5,   1])
3 = {Tensor: (18,)} tensor([  0,   6,  12,  56,  63,    4,  877,  14,   27, 738,   5,    1,    2,   2,    2,   2,   2,   2])
```

接下来的一个关键操作，是生成decoder的输入，和对应的预测目标target。

仍以第一个德-英数据对为例子，decoder是自回归的，当decoder的输入是“A man and a”时，希望decoder的输出是“woman”；下一轮，当decoder的输入是“A man and a woman”时，希望decoder的输出是“tak”。

所以对这一个德-英数据对，事实上会生成17条训练语料。内容如下：
- decoder输入：`[  0]`，对应的预测目标target：`6`
- decoder输入：`[  0,   6]`，对应的预测目标target：`12`
- decoder输入：`[  0,   6,  12]`，对应的预测目标target：`11`
- decoder输入：`[  0,   6,  12,  11]`，对应的预测目标target：`4`
- decoder输入：`[  0,   6,  12,  11,   4]`，对应的预测目标target：`16`
以此类推。

具体代码实现，是为英文串生成一个mask矩阵，把不该被看到的内容掩盖掉。关键代码如下：
```python
class Batch:
    """Object for holding a batch of data with mask during training."""

    def __init__(self, src, tgt=None, pad=2):  # 2 = <blank>
        self.src = src
        self.src_mask = (src != pad).unsqueeze(-2)
        if tgt is not None:
            self.tgt = tgt[:, :-1]
            self.tgt_y = tgt[:, 1:]
            self.tgt_mask = self.make_std_mask(self.tgt, pad)
            self.ntokens = (self.tgt_y != pad).data.sum()

    @staticmethod
    def make_std_mask(tgt, pad):
        "Create a mask to hide padding and future words."
        tgt_mask = (tgt != pad).unsqueeze(-2)
        tgt_mask = tgt_mask & subsequent_mask(tgt.size(-1)).type_as(
            tgt_mask.data
        )
        return tgt_mask
```
## 1.3.token化

### tokenizer
tokenizer使用的是spacy库提供的en_core_web_sm、de_core_news_sm，分词实现比较简单，基本只用空格隔开了单词，如果处理海量训练语料的话词表会爆炸。除分词外，还支持分词、词性标注、命名实体识别和依存句法分析等功能。

生产环境可用的token库可参考llama使用的SentencePiece，和HuggingFace封装的版本https://github.com/huggingface/tokenizers

分词、词性标注、命名实体识别和依存句法分析等。

使用spacy_en进行token化后的结果示例如下，"Three dogs are playing in the water." 这句话会被分割成8个token：`'Three'、'dogs'、'are'、'playing'、'in'、'the'、'water'、'.'`。

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

### 词表
将所有语料进行token化后，会生成词表。词表就是将所有token去重后生成的列表。
如果我只有两条英文训练语料：“Three dogs are playing in the water.”、“My dog is playing in the water.”，那么，生成的词表大小将是11，内容为：
`'Three'、'dogs'、'are'、'playing'、'in'、'the'、'water'、'.'、'My'、'dog'、'is'`。

在本文的翻译场景中，德语词表大小为8316、英语词表大小为6384。

词表大小会直接影响模型的参数量。Encoder输入是德语，【TODO 参数量计算】


### 代码示例

```python
spacy_de = spacy.load("de_core_news_sm")

def tokenize(text, tokenizer):
    return [tok.text for tok in tokenizer.tokenizer(text)]

def build_vocabulary(spacy_de, spacy_en):
    def tokenize_de(text):
        return tokenize(text, spacy_de)

    print("Building German Vocabulary ...")
    train, val, test = datasets.Multi30k(language_pair=("de", "en"))
    vocab_src = build_vocab_from_iterator(
        yield_tokens(train + val + test, tokenize_de, index=0),
        min_freq=2,
        specials=["<s>", "</s>", "<blank>", "<unk>"],
    )
    vocab_src.set_default_index(vocab_src["<unk>"])

    return vocab_src, vocab_tgt

def yield_tokens(data_iter, tokenizer, index):
    for from_to_tuple in data_iter:
        yield tokenizer(from_to_tuple[index])
```




# 损失函数

在本文的翻译场景中，德语词表大小为8316、英语词表大小为6384。德译英，将问题看成是一个6384类的多分类问题，loss是输出与真实分布之间的KL散度。

为了防止模型过拟合、改善泛化能力，采用了Label Smoothing算法。

代码及注释：

```python
class LabelSmoothing(nn.Module):
    "Implement label smoothing."

    def __init__(self, size, padding_idx, smoothing=0.0):
        super(LabelSmoothing, self).__init__()
        self.criterion = nn.KLDivLoss(reduction="sum")
        self.padding_idx = padding_idx
        self.confidence = 1.0 - smoothing
        self.smoothing = smoothing
        self.size = size
        self.true_dist = None

    def forward(self, x, target):
        assert x.size(1) == self.size
        true_dist = x.data.clone()
        # 初始化时，将其填充为每个类别一个很小的概率值，这个值是 self.smoothing / (self.size - 2)，表示将 smoothing 均匀分布给除正确类别和填充标记外的所有类别。
        true_dist.fill_(self.smoothing / (self.size - 2))
        # 这里使用 scatter_ 将正确类别的概率设置为 self.confidence，即 1.0 - smoothing
        true_dist.scatter_(1, target.data.unsqueeze(1), self.confidence)
        # 填充标记的位置不进行损失计算
        true_dist[:, self.padding_idx] = 0
        mask = torch.nonzero(target.data == self.padding_idx)
        if mask.dim() > 0:
            true_dist.index_fill_(0, mask.squeeze(), 0.0)
        self.true_dist = true_dist
        return self.criterion(x, true_dist.clone().detach())

```




# 2.模型详解

## 2.1.概览

本章以`Einige Männer blicken auf einen Computerbildschirm in einem Büro .`这句德文如何翻译成英文为例子，拆解每一个模型模块。


N=6, d_model=512, d_ff=2048, h=8, dropout=0.1

参考Transformer论文大图，模型由如下四大块儿构成：

第一部分是德文输入的embedding和PositionalEncoding，每一个token都将映射成一个512维的向量。
```python
(src_embed): Sequential(
    (0): Embeddings(
      (lut): Embedding(8316, 512)
    )
    (1): PositionalEncoding(
      (dropout): Dropout(p=0.1, inplace=False)
    )
  )
```

`Einige Männer blicken auf einen Computerbildschirm in einem Büro .`这句德文首先经过token化，会生成一个长度为128的向量。128是max_padding值，每一个encoder的input都会补全到128长度。
```python
tensor([[   0,  425,   31,  338,   11,   20, 1613,    7,    6, 1011,    4,    1,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,    2,
            2,    2,    2,    2,    2,    2,    2,    2]])
```

经过embedding和PositionalEncoding，输出的结果是一个(1,128,512)大小的Tensor：1是batch_size，在做inference的时候没有并行推理，设置成1；128是token数；512是每一个token被映射成的向量长度。示例如下：

```python
tensor([[[-0.4575,  1.2455, -0.2974,  ...,  1.6284,  1.0158,  1.3557],
         [ 1.3021,  1.1043,  1.0065,  ...,  1.9354,  0.0502,  0.0639],
         [ 0.4431, -1.4890,  1.2940,  ...,  0.3886, -0.3936,  2.3299],
         ...,
         [-0.5675,  0.0000,  0.7862,  ...,  1.7392,  0.4239,  1.4839],
         [ 0.4837,  1.2939,  0.6693,  ...,  1.7392,  0.4240,  1.4839],
         [ 1.1977,  0.5032, -0.2390,  ...,  1.7392,  0.4241,  0.0000]]],
       grad_fn=<MulBackward0>)
```

而后这个(1,128,512)大小的Tensor，会送入Encoder，生成(1,128,512)的输出。此处，Encoder的输入和输出长度是一样的。

Encoder的模型结构如下：

```python
(encoder): Encoder(
    (layers): ModuleList(
      (0-5): 6 x EncoderLayer(
        (self_attn): MultiHeadedAttention(
          (linears): ModuleList(
            (0-3): 4 x Linear(in_features=512, out_features=512, bias=True)
          )
          (dropout): Dropout(p=0.1, inplace=False)
        )
        (feed_forward): PositionwiseFeedForward(
          (w_1): Linear(in_features=512, out_features=2048, bias=True)
          (w_2): Linear(in_features=2048, out_features=512, bias=True)
          (dropout): Dropout(p=0.1, inplace=False)
        )
        (sublayer): ModuleList(
          (0-1): 2 x SublayerConnection(
            (norm): LayerNorm()
            (dropout): Dropout(p=0.1, inplace=False)
          )
        )
      )
    )
    (norm): LayerNorm()
  )
```

输出结果如下：

```python
tensor([[[ 1.9084,  1.7498, -0.2022,  ..., -0.3563, -0.6095, -0.0592],
         [ 1.6080,  2.0479,  0.4022,  ..., -0.2230, -0.4846, -0.5235],
         [ 0.1693, -0.3136,  1.2662,  ..., -1.1920,  1.8090,  0.4581],
         ...,
         [-1.1504,  0.4094, -1.2005,  ..., -1.9467,  0.5757, -0.6707],
         [-0.0957,  0.6487,  0.1256,  ..., -0.6127,  0.5276, -0.3006],
         [ 0.4663,  0.1231, -0.5799,  ..., -1.2637,  1.1503, -0.6398]]],
        )
```

而后是自回归的decoder过程。只要不断调用，decoder可以生成无线长的输出，但实际应用中会限制一个最大输出长度，此处设置为了64。

第一轮输入一个内容为“start_symbol”的(1,1)大小的Tensor，然后经过Embedding、PositionalEncoding过程后，送入Decoder，最终生成一个(1,1,512)的输出，代表生成的第一个token，然后经过Generator，最终生成token“403”,对应英文单词Some。

自回归63次，最终生成如下token串:
```python
tensor([[  0, 423,  36, 213,  20,   4, 430, 663,   7,  28, 785, 785, 191,   5,
           1,   1,   5,   1,   5,   1,   1,   5,   1,   1,   5,   1,   1,   5,
           1,   5,   1,   5,   1,   1,   5,   1,   5,   1,   5,   1,   5,   1,
           5,   1,   1,   1,   1,   1,   5,   1,   1,   1,   1,   1,   1,   5,
           1,   1,   1,   1,   1,   1,   1,   1,   1,   1,   1,   1,   1,   1,
           1,   1]])
```
对应的英文内容是：“Some men look at a computer screen in an office office room .”


```python
  (tgt_embed): Sequential(
    (0): Embeddings(
      (lut): Embedding(6384, 512)
    )
    (1): PositionalEncoding(
      (dropout): Dropout(p=0.1, inplace=False)
    )
  )
  
  (decoder): Decoder(
    (layers): ModuleList(
      (0-5): 6 x DecoderLayer(
        (self_attn): MultiHeadedAttention(
          (linears): ModuleList(
            (0-3): 4 x Linear(in_features=512, out_features=512, bias=True)
          )
          (dropout): Dropout(p=0.1, inplace=False)
        )
        (src_attn): MultiHeadedAttention(
          (linears): ModuleList(
            (0-3): 4 x Linear(in_features=512, out_features=512, bias=True)
          )
          (dropout): Dropout(p=0.1, inplace=False)
        )
        (feed_forward): PositionwiseFeedForward(
          (w_1): Linear(in_features=512, out_features=2048, bias=True)
          (w_2): Linear(in_features=2048, out_features=512, bias=True)
          (dropout): Dropout(p=0.1, inplace=False)
        )
        (sublayer): ModuleList(
          (0-2): 3 x SublayerConnection(
            (norm): LayerNorm()
            (dropout): Dropout(p=0.1, inplace=False)
          )
        )
      )
    )
    (norm): LayerNorm()
  )
  
  (generator): Generator(
    (proj): Linear(in_features=512, out_features=6384, bias=True)
  )
```

模型讲解比较好的文章可参考：https://jalammar.github.io/illustrated-transformer/





## 2.1.Embedding

```python
class Embeddings(nn.Module):
    def __init__(self, d_model, vocab):
        super(Embeddings, self).__init__()
        self.lut = nn.Embedding(vocab, d_model)
        self.d_model = d_model

    def forward(self, x):
        embedding_val = self.lut(x) * math.sqrt(self.d_model)
        print(f"【Embeddings】{x} -> {embedding_val}")
        return embedding_val
```

## 2.2.Attention

参考我的另一篇llama2代码解读文档。



```python
class MultiHeadedAttention(nn.Module):
    def __init__(self, h, d_model, dropout=0.1):
        "Take in model size and number of heads."
        super(MultiHeadedAttention, self).__init__()
        assert d_model % h == 0
        # We assume d_v always equals d_k
        self.d_k = d_model // h
        self.h = h
        self.linears = clones(nn.Linear(d_model, d_model), 4)
        self.attn = None
        self.dropout = nn.Dropout(p=dropout)

    def forward(self, query, key, value, mask=None):
        "Implements Figure 2"
        if mask is not None:
            # Same mask applied to all h heads.
            mask = mask.unsqueeze(1)
        nbatches = query.size(0)

        # 1) Do all the linear projections in batch from d_model => h x d_k
        query, key, value = [
            lin(x).view(nbatches, -1, self.h, self.d_k).transpose(1, 2)
            for lin, x in zip(self.linears, (query, key, value))
        ]

        # 2) Apply attention on all the projected vectors in batch.
        x, self.attn = attention(
            query, key, value, mask=mask, dropout=self.dropout
        )

        # 3) "Concat" using a view and apply a final linear.
        x = (
            x.transpose(1, 2)
            .contiguous()
            .view(nbatches, -1, self.h * self.d_k)
        )
        del query
        del key
        del value
        return self.linears[-1](x)
```

## 2.3.PositionalEncoding

TODO




# 3.训练过程

```python
config = {
        "batch_size": 32,
        "distributed": False,
        "num_epochs": 8,
        "accum_iter": 10,
        "base_lr": 1.0,
        "max_padding": 72,
        "warmup": 3000,
        "file_prefix": "multi30k_model_",
    }
```


![Figure_1_Loss_and_LearningRate.png](http://jinluzhang.github.io/assets/posts_img/2024-09-01-Transformer/Figure_1_Loss_and_LearningRate.png)





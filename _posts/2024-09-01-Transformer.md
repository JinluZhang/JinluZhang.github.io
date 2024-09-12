---
layout: post
title: 'Transformer代码实现详解'
date: 2024-09-01
categories: 技术
author: Beaug
tags: Transformer 大模型 annotated-transformer
---




休假在家训模型玩儿，顺便把实践过程整理成文档。

本文训练了一个英德互译的模型，模型架构忠实复现Transformer论文（[Attention Is All You Need](https://arxiv.org/pdf/1706.03762)
）,代码来自 https://github.com/harvardnlp/annotated-transformer。

文章用三万组英-德语料训练了一个翻译模型，并以“Three dogs are playing in the water.” 这句英文如何翻译成德文 “Drei Hunde spielen im Wasser.”为例子，拆解了token化、encoder、decoder的完整流程。

看完文章将了解：
1.一句德文输入，在各个中间环节的具体内容，及含义
2.在这个完整例子中，Transformer到底有多少个参数




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
def load_tokenizers():

    try:
        spacy_de = spacy.load("de_core_news_sm")
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


# 模型整体结构

N=6, d_model=512, d_ff=2048, h=8, dropout=0.1


```python
EncoderDecoder(
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
  (src_embed): Sequential(
    (0): Embeddings(
      (lut): Embedding(8316, 512)
    )
    (1): PositionalEncoding(
      (dropout): Dropout(p=0.1, inplace=False)
    )
  )
  (tgt_embed): Sequential(
    (0): Embeddings(
      (lut): Embedding(6384, 512)
    )
    (1): PositionalEncoding(
      (dropout): Dropout(p=0.1, inplace=False)
    )
  )
  (generator): Generator(
    (proj): Linear(in_features=512, out_features=6384, bias=True)
  )
)
```



# 构造训练输入数据

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
- 注意到每行第一个token都是“0”，这是"<s>"的token id，代表一句话的开始。一句话结束后，会在末尾添加"</s>"的token id “1”。例子中，第一句德文和第四句德文比较短，所以添加上了“1”；第二句和第三句德文比较长，做了截断处理。
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

# 推理过程



# 训练过程

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






任务*Tasks*组成，每个task执行相同的二进制程序 <font color=green>（task间互为replica）</font>


[GPU0] Epoch 0 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   7.66 | Tokens / Sec:   293.6 | Learning Rate: 5.4e-07
Epoch Step:     41 | Accumulation Step:   5 | Loss:   7.42 | Tokens / Sec:   329.3 | Learning Rate: 1.1e-05
Epoch Step:     81 | Accumulation Step:   9 | Loss:   7.00 | Tokens / Sec:   338.4 | Learning Rate: 2.2e-05
Epoch Step:    121 | Accumulation Step:  13 | Loss:   6.81 | Tokens / Sec:   337.4 | Learning Rate: 3.3e-05
Epoch Step:    161 | Accumulation Step:  17 | Loss:   6.55 | Tokens / Sec:   335.2 | Learning Rate: 4.4e-05
Epoch Step:    201 | Accumulation Step:  21 | Loss:   6.41 | Tokens / Sec:   344.5 | Learning Rate: 5.4e-05
Epoch Step:    241 | Accumulation Step:  25 | Loss:   6.20 | Tokens / Sec:   335.7 | Learning Rate: 6.5e-05
Epoch Step:    281 | Accumulation Step:  29 | Loss:   6.01 | Tokens / Sec:   345.3 | Learning Rate: 7.6e-05
Epoch Step:    321 | Accumulation Step:  33 | Loss:   5.75 | Tokens / Sec:   341.9 | Learning Rate: 8.7e-05
Epoch Step:    361 | Accumulation Step:  37 | Loss:   5.56 | Tokens / Sec:   341.0 | Learning Rate: 9.7e-05
Epoch Step:    401 | Accumulation Step:  41 | Loss:   5.51 | Tokens / Sec:   340.2 | Learning Rate: 1.1e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   5.08 | Tokens / Sec:   339.9 | Learning Rate: 1.2e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   4.88 | Tokens / Sec:   335.7 | Learning Rate: 1.3e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   4.58 | Tokens / Sec:   344.8 | Learning Rate: 1.4e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   4.69 | Tokens / Sec:   337.1 | Learning Rate: 1.5e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   4.51 | Tokens / Sec:   340.7 | Learning Rate: 1.6e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   4.39 | Tokens / Sec:   338.4 | Learning Rate: 1.7e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   4.43 | Tokens / Sec:   344.0 | Learning Rate: 1.8e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   4.17 | Tokens / Sec:   338.3 | Learning Rate: 1.9e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   4.33 | Tokens / Sec:   332.3 | Learning Rate: 2.0e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   4.02 | Tokens / Sec:   338.8 | Learning Rate: 2.2e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   3.86 | Tokens / Sec:   334.6 | Learning Rate: 2.3e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   4.02 | Tokens / Sec:   339.1 | Learning Rate: 2.4e-04
(tensor(3.8996), <__main__.TrainState object at 0x294076e50>)
[GPU0] Epoch 1 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   3.93 | Tokens / Sec:   330.4 | Learning Rate: 2.4e-04
Epoch Step:     41 | Accumulation Step:   5 | Loss:   3.88 | Tokens / Sec:   341.6 | Learning Rate: 2.6e-04
Epoch Step:     81 | Accumulation Step:   9 | Loss:   3.81 | Tokens / Sec:   331.8 | Learning Rate: 2.7e-04
Epoch Step:    121 | Accumulation Step:  13 | Loss:   3.83 | Tokens / Sec:   337.3 | Learning Rate: 2.8e-04
Epoch Step:    161 | Accumulation Step:  17 | Loss:   3.52 | Tokens / Sec:   334.5 | Learning Rate: 2.9e-04
Epoch Step:    201 | Accumulation Step:  21 | Loss:   3.60 | Tokens / Sec:   333.5 | Learning Rate: 3.0e-04
Epoch Step:    241 | Accumulation Step:  25 | Loss:   3.44 | Tokens / Sec:   334.5 | Learning Rate: 3.1e-04
Epoch Step:    281 | Accumulation Step:  29 | Loss:   3.39 | Tokens / Sec:   341.3 | Learning Rate: 3.2e-04
Epoch Step:    321 | Accumulation Step:  33 | Loss:   3.34 | Tokens / Sec:   338.7 | Learning Rate: 3.3e-04
Epoch Step:    361 | Accumulation Step:  37 | Loss:   3.12 | Tokens / Sec:   333.4 | Learning Rate: 3.4e-04
Epoch Step:    401 | Accumulation Step:  41 | Loss:   3.16 | Tokens / Sec:   336.1 | Learning Rate: 3.5e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   3.47 | Tokens / Sec:   341.0 | Learning Rate: 3.6e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   3.27 | Tokens / Sec:   339.9 | Learning Rate: 3.7e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   3.14 | Tokens / Sec:   338.1 | Learning Rate: 3.8e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   3.18 | Tokens / Sec:   342.8 | Learning Rate: 4.0e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   3.25 | Tokens / Sec:   336.5 | Learning Rate: 4.1e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   3.11 | Tokens / Sec:   336.9 | Learning Rate: 4.2e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   3.04 | Tokens / Sec:   339.2 | Learning Rate: 4.3e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   3.03 | Tokens / Sec:   344.6 | Learning Rate: 4.4e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   3.24 | Tokens / Sec:   337.2 | Learning Rate: 4.5e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   2.84 | Tokens / Sec:   335.2 | Learning Rate: 4.6e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   2.86 | Tokens / Sec:   337.2 | Learning Rate: 4.7e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   2.88 | Tokens / Sec:   340.1 | Learning Rate: 4.8e-04
(tensor(2.8472), <__main__.TrainState object at 0x294076e50>)
[GPU0] Epoch 2 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   2.82 | Tokens / Sec:   332.6 | Learning Rate: 4.9e-04
Epoch Step:     41 | Accumulation Step:   5 | Loss:   2.96 | Tokens / Sec:   334.7 | Learning Rate: 5.0e-04
Epoch Step:     81 | Accumulation Step:   9 | Loss:   2.85 | Tokens / Sec:   334.3 | Learning Rate: 5.1e-04
Epoch Step:    121 | Accumulation Step:  13 | Loss:   2.89 | Tokens / Sec:   337.5 | Learning Rate: 5.2e-04
Epoch Step:    161 | Accumulation Step:  17 | Loss:   2.73 | Tokens / Sec:   334.6 | Learning Rate: 5.3e-04
Epoch Step:    201 | Accumulation Step:  21 | Loss:   2.80 | Tokens / Sec:   340.4 | Learning Rate: 5.4e-04
Epoch Step:    241 | Accumulation Step:  25 | Loss:   2.75 | Tokens / Sec:   339.2 | Learning Rate: 5.5e-04
Epoch Step:    281 | Accumulation Step:  29 | Loss:   2.64 | Tokens / Sec:   339.1 | Learning Rate: 5.6e-04
Epoch Step:    321 | Accumulation Step:  33 | Loss:   2.85 | Tokens / Sec:   339.0 | Learning Rate: 5.7e-04
Epoch Step:    361 | Accumulation Step:  37 | Loss:   2.67 | Tokens / Sec:   338.6 | Learning Rate: 5.9e-04
Epoch Step:    401 | Accumulation Step:  41 | Loss:   2.79 | Tokens / Sec:   336.3 | Learning Rate: 6.0e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   2.57 | Tokens / Sec:   333.8 | Learning Rate: 6.1e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   2.32 | Tokens / Sec:   336.4 | Learning Rate: 6.2e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   2.47 | Tokens / Sec:   338.1 | Learning Rate: 6.3e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   2.57 | Tokens / Sec:   335.6 | Learning Rate: 6.4e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   2.18 | Tokens / Sec:   342.0 | Learning Rate: 6.5e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   2.42 | Tokens / Sec:   337.4 | Learning Rate: 6.6e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   2.50 | Tokens / Sec:   339.4 | Learning Rate: 6.7e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   2.44 | Tokens / Sec:   334.4 | Learning Rate: 6.8e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   2.28 | Tokens / Sec:   339.3 | Learning Rate: 6.9e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   2.32 | Tokens / Sec:   338.5 | Learning Rate: 7.0e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   2.32 | Tokens / Sec:   335.9 | Learning Rate: 7.1e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   2.45 | Tokens / Sec:   333.9 | Learning Rate: 7.3e-04
(tensor(2.1266), <__main__.TrainState object at 0x294076e50>)
[GPU0] Epoch 3 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   2.27 | Tokens / Sec:   345.7 | Learning Rate: 7.3e-04
Epoch Step:     41 | Accumulation Step:   5 | Loss:   2.32 | Tokens / Sec:   340.9 | Learning Rate: 7.4e-04
Epoch Step:     81 | Accumulation Step:   9 | Loss:   1.82 | Tokens / Sec:   339.3 | Learning Rate: 7.5e-04
Epoch Step:    121 | Accumulation Step:  13 | Loss:   2.05 | Tokens / Sec:   335.7 | Learning Rate: 7.6e-04
Epoch Step:    161 | Accumulation Step:  17 | Loss:   2.22 | Tokens / Sec:   334.4 | Learning Rate: 7.8e-04
Epoch Step:    201 | Accumulation Step:  21 | Loss:   2.07 | Tokens / Sec:   334.5 | Learning Rate: 7.9e-04
Epoch Step:    241 | Accumulation Step:  25 | Loss:   2.05 | Tokens / Sec:   332.6 | Learning Rate: 8.0e-04
Epoch Step:    281 | Accumulation Step:  29 | Loss:   1.90 | Tokens / Sec:   335.9 | Learning Rate: 8.1e-04
Epoch Step:    321 | Accumulation Step:  33 | Loss:   1.94 | Tokens / Sec:   339.6 | Learning Rate: 8.0e-04
Epoch Step:    361 | Accumulation Step:  37 | Loss:   1.60 | Tokens / Sec:   336.1 | Learning Rate: 8.0e-04
Epoch Step:    401 | Accumulation Step:  41 | Loss:   1.73 | Tokens / Sec:   336.4 | Learning Rate: 7.9e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   1.85 | Tokens / Sec:   334.8 | Learning Rate: 7.9e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   2.08 | Tokens / Sec:   337.5 | Learning Rate: 7.8e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   1.75 | Tokens / Sec:   338.8 | Learning Rate: 7.8e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   1.77 | Tokens / Sec:   333.1 | Learning Rate: 7.7e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   1.87 | Tokens / Sec:   336.7 | Learning Rate: 7.7e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   1.80 | Tokens / Sec:   340.9 | Learning Rate: 7.6e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   2.05 | Tokens / Sec:   336.4 | Learning Rate: 7.6e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   1.98 | Tokens / Sec:   340.5 | Learning Rate: 7.5e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   1.47 | Tokens / Sec:   334.1 | Learning Rate: 7.5e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   1.78 | Tokens / Sec:   328.8 | Learning Rate: 7.4e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   1.88 | Tokens / Sec:   336.8 | Learning Rate: 7.4e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   2.13 | Tokens / Sec:   335.1 | Learning Rate: 7.4e-04
(tensor(1.7449), <__main__.TrainState object at 0x294076e50>)
[GPU0] Epoch 4 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   1.46 | Tokens / Sec:   331.6 | Learning Rate: 7.3e-04
Epoch Step:     41 | Accumulation Step:   5 | Loss:   1.42 | Tokens / Sec:   337.1 | Learning Rate: 7.3e-04
Epoch Step:     81 | Accumulation Step:   9 | Loss:   1.48 | Tokens / Sec:   332.2 | Learning Rate: 7.3e-04
Epoch Step:    121 | Accumulation Step:  13 | Loss:   1.39 | Tokens / Sec:   339.5 | Learning Rate: 7.2e-04
Epoch Step:    161 | Accumulation Step:  17 | Loss:   1.59 | Tokens / Sec:   336.3 | Learning Rate: 7.2e-04
Epoch Step:    201 | Accumulation Step:  21 | Loss:   1.40 | Tokens / Sec:   323.9 | Learning Rate: 7.1e-04
Epoch Step:    241 | Accumulation Step:  25 | Loss:   1.65 | Tokens / Sec:   343.5 | Learning Rate: 7.1e-04
Epoch Step:    281 | Accumulation Step:  29 | Loss:   1.79 | Tokens / Sec:   338.2 | Learning Rate: 7.1e-04
Epoch Step:    321 | Accumulation Step:  33 | Loss:   1.42 | Tokens / Sec:   340.1 | Learning Rate: 7.0e-04
Epoch Step:    361 | Accumulation Step:  37 | Loss:   1.61 | Tokens / Sec:   337.5 | Learning Rate: 7.0e-04
Epoch Step:    401 | Accumulation Step:  41 | Loss:   1.77 | Tokens / Sec:   333.8 | Learning Rate: 7.0e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   1.60 | Tokens / Sec:   338.9 | Learning Rate: 6.9e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   1.67 | Tokens / Sec:   342.8 | Learning Rate: 6.9e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   1.35 | Tokens / Sec:   329.3 | Learning Rate: 6.9e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   1.72 | Tokens / Sec:   340.3 | Learning Rate: 6.8e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   1.31 | Tokens / Sec:   342.7 | Learning Rate: 6.8e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   1.64 | Tokens / Sec:   339.6 | Learning Rate: 6.8e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   1.80 | Tokens / Sec:   335.9 | Learning Rate: 6.7e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   1.50 | Tokens / Sec:   325.5 | Learning Rate: 6.7e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   1.33 | Tokens / Sec:   334.8 | Learning Rate: 6.7e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   1.18 | Tokens / Sec:   332.7 | Learning Rate: 6.6e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   1.49 | Tokens / Sec:   332.3 | Learning Rate: 6.6e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   1.61 | Tokens / Sec:   339.1 | Learning Rate: 6.6e-04
(tensor(1.5847), <__main__.TrainState object at 0x294076e50>)
[GPU0] Epoch 5 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   1.32 | Tokens / Sec:   337.4 | Learning Rate: 6.6e-04
Epoch Step:     41 | Accumulation Step:   5 | Loss:   1.11 | Tokens / Sec:   334.7 | Learning Rate: 6.5e-04
Epoch Step:     81 | Accumulation Step:   9 | Loss:   1.42 | Tokens / Sec:   341.4 | Learning Rate: 6.5e-04
Epoch Step:    121 | Accumulation Step:  13 | Loss:   1.33 | Tokens / Sec:   345.2 | Learning Rate: 6.5e-04
Epoch Step:    161 | Accumulation Step:  17 | Loss:   1.28 | Tokens / Sec:   336.8 | Learning Rate: 6.4e-04
Epoch Step:    201 | Accumulation Step:  21 | Loss:   1.16 | Tokens / Sec:   339.7 | Learning Rate: 6.4e-04
Epoch Step:    241 | Accumulation Step:  25 | Loss:   1.64 | Tokens / Sec:   338.9 | Learning Rate: 6.4e-04
Epoch Step:    281 | Accumulation Step:  29 | Loss:   1.18 | Tokens / Sec:   333.5 | Learning Rate: 6.4e-04
Epoch Step:    321 | Accumulation Step:  33 | Loss:   1.24 | Tokens / Sec:   334.7 | Learning Rate: 6.3e-04
Epoch Step:    361 | Accumulation Step:  37 | Loss:   1.19 | Tokens / Sec:   338.6 | Learning Rate: 6.3e-04
Epoch Step:    401 | Accumulation Step:  41 | Loss:   1.43 | Tokens / Sec:   338.0 | Learning Rate: 6.3e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   1.42 | Tokens / Sec:   341.3 | Learning Rate: 6.3e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   1.41 | Tokens / Sec:   338.6 | Learning Rate: 6.2e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   1.07 | Tokens / Sec:   335.5 | Learning Rate: 6.2e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   1.34 | Tokens / Sec:   343.5 | Learning Rate: 6.2e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   1.45 | Tokens / Sec:   340.6 | Learning Rate: 6.2e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   1.23 | Tokens / Sec:   335.1 | Learning Rate: 6.1e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   1.24 | Tokens / Sec:   330.3 | Learning Rate: 6.1e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   1.31 | Tokens / Sec:   325.8 | Learning Rate: 6.1e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   1.25 | Tokens / Sec:   335.3 | Learning Rate: 6.1e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   1.15 | Tokens / Sec:   337.4 | Learning Rate: 6.0e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   1.28 | Tokens / Sec:   332.6 | Learning Rate: 6.0e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   1.14 | Tokens / Sec:   329.5 | Learning Rate: 6.0e-04
(tensor(1.4928), <__main__.TrainState object at 0x294076e50>)
[GPU0] Epoch 6 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   1.05 | Tokens / Sec:   325.8 | Learning Rate: 6.0e-04
Epoch Step:     41 | Accumulation Step:   5 | Loss:   1.03 | Tokens / Sec:   334.7 | Learning Rate: 6.0e-04
Epoch Step:     81 | Accumulation Step:   9 | Loss:   1.11 | Tokens / Sec:   338.6 | Learning Rate: 5.9e-04
Epoch Step:    121 | Accumulation Step:  13 | Loss:   1.20 | Tokens / Sec:   342.2 | Learning Rate: 5.9e-04
Epoch Step:    161 | Accumulation Step:  17 | Loss:   1.13 | Tokens / Sec:   342.6 | Learning Rate: 5.9e-04
Epoch Step:    201 | Accumulation Step:  21 | Loss:   1.14 | Tokens / Sec:   334.0 | Learning Rate: 5.9e-04
Epoch Step:    241 | Accumulation Step:  25 | Loss:   1.08 | Tokens / Sec:   331.7 | Learning Rate: 5.9e-04
Epoch Step:    281 | Accumulation Step:  29 | Loss:   0.86 | Tokens / Sec:   337.4 | Learning Rate: 5.8e-04
Epoch Step:    321 | Accumulation Step:  33 | Loss:   1.21 | Tokens / Sec:   337.8 | Learning Rate: 5.8e-04
Epoch Step:    361 | Accumulation Step:  37 | Loss:   1.25 | Tokens / Sec:   339.3 | Learning Rate: 5.8e-04
Epoch Step:    401 | Accumulation Step:  41 | Loss:   1.42 | Tokens / Sec:   341.1 | Learning Rate: 5.8e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   1.14 | Tokens / Sec:   336.4 | Learning Rate: 5.8e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   1.00 | Tokens / Sec:   336.1 | Learning Rate: 5.7e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   1.13 | Tokens / Sec:   329.8 | Learning Rate: 5.7e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   1.36 | Tokens / Sec:   331.8 | Learning Rate: 5.7e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   1.14 | Tokens / Sec:   334.8 | Learning Rate: 5.7e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   1.06 | Tokens / Sec:   331.1 | Learning Rate: 5.7e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   1.38 | Tokens / Sec:   334.8 | Learning Rate: 5.6e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   1.24 | Tokens / Sec:   329.9 | Learning Rate: 5.6e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   1.10 | Tokens / Sec:   335.6 | Learning Rate: 5.6e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   1.14 | Tokens / Sec:   331.1 | Learning Rate: 5.6e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   1.38 | Tokens / Sec:   335.5 | Learning Rate: 5.6e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   1.34 | Tokens / Sec:   335.4 | Learning Rate: 5.6e-04
(tensor(1.4629), <__main__.TrainState object at 0x294076e50>)
[GPU0] Epoch 7 Training ====
Epoch Step:      1 | Accumulation Step:   1 | Loss:   0.92 | Tokens / Sec:   336.8 | Learning Rate: 5.5e-04
Epoch Step:     41 | Accumulation Step:   5 | Loss:   1.06 | Tokens / Sec:   338.7 | Learning Rate: 5.5e-04
Epoch Step:     81 | Accumulation Step:   9 | Loss:   1.04 | Tokens / Sec:   338.7 | Learning Rate: 5.5e-04
Epoch Step:    121 | Accumulation Step:  13 | Loss:   1.10 | Tokens / Sec:   342.0 | Learning Rate: 5.5e-04
Epoch Step:    161 | Accumulation Step:  17 | Loss:   0.99 | Tokens / Sec:   347.5 | Learning Rate: 5.5e-04
Epoch Step:    201 | Accumulation Step:  21 | Loss:   0.95 | Tokens / Sec:   336.9 | Learning Rate: 5.5e-04
Epoch Step:    241 | Accumulation Step:  25 | Loss:   1.13 | Tokens / Sec:   335.9 | Learning Rate: 5.4e-04
Epoch Step:    281 | Accumulation Step:  29 | Loss:   1.17 | Tokens / Sec:   336.2 | Learning Rate: 5.4e-04
Epoch Step:    321 | Accumulation Step:  33 | Loss:   0.98 | Tokens / Sec:   335.5 | Learning Rate: 5.4e-04
Epoch Step:    361 | Accumulation Step:  37 | Loss:   1.01 | Tokens / Sec:   338.6 | Learning Rate: 5.4e-04
Epoch Step:    401 | Accumulation Step:  41 | Loss:   1.26 | Tokens / Sec:   341.0 | Learning Rate: 5.4e-04
Epoch Step:    441 | Accumulation Step:  45 | Loss:   0.83 | Tokens / Sec:   336.5 | Learning Rate: 5.4e-04
Epoch Step:    481 | Accumulation Step:  49 | Loss:   0.91 | Tokens / Sec:   335.2 | Learning Rate: 5.3e-04
Epoch Step:    521 | Accumulation Step:  53 | Loss:   0.94 | Tokens / Sec:   339.0 | Learning Rate: 5.3e-04
Epoch Step:    561 | Accumulation Step:  57 | Loss:   0.97 | Tokens / Sec:   340.1 | Learning Rate: 5.3e-04
Epoch Step:    601 | Accumulation Step:  61 | Loss:   1.08 | Tokens / Sec:   335.8 | Learning Rate: 5.3e-04
Epoch Step:    641 | Accumulation Step:  65 | Loss:   1.12 | Tokens / Sec:   331.5 | Learning Rate: 5.3e-04
Epoch Step:    681 | Accumulation Step:  69 | Loss:   1.03 | Tokens / Sec:   336.4 | Learning Rate: 5.3e-04
Epoch Step:    721 | Accumulation Step:  73 | Loss:   1.12 | Tokens / Sec:   343.9 | Learning Rate: 5.3e-04
Epoch Step:    761 | Accumulation Step:  77 | Loss:   1.02 | Tokens / Sec:   329.0 | Learning Rate: 5.2e-04
Epoch Step:    801 | Accumulation Step:  81 | Loss:   0.93 | Tokens / Sec:   332.3 | Learning Rate: 5.2e-04
Epoch Step:    841 | Accumulation Step:  85 | Loss:   1.08 | Tokens / Sec:   337.8 | Learning Rate: 5.2e-04
Epoch Step:    881 | Accumulation Step:  89 | Loss:   1.07 | Tokens / Sec:   328.9 | Learning Rate: 5.2e-04
(tensor(1.4349), <__main__.TrainState object at 0x294076e50>)


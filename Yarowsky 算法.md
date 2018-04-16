Yarowsky 算法是一个非监督的学习算法，用于词义消歧 (WSD, word sense disambiguation)，其中使用了 "one sense per collocation" 和 "one sense per discourse" 的人类语言性质。

## 过程

算法从一堆未标记 (untagged) 的语料开始，语料中标明所给一词多义 (polysemous) 词语例子，并且以行存储所有相关句。例如，Yarowsky 在他 1995 年论文中使用了 "plant" 一词来描述算法。如果假设一个词存在两种不同的意思，下一步则是对每种意思去确认一些代表性的搭配词组种子 (seed collocations)，并给出词义标签，随后对所有训练例子分配包含 see collocations 的合适的标签。例如，词语 "life" 和 "manufacturing" 被作为初始搭配词组种子 (seed collocations) 被选中分别代表 senses A 和 senses B。剩余的例子 (Yarowsky 认为是 85%-98%) 仍旧是未标记的。

算法应当初始选择代表性的搭配词组种子 (seed collocations) ，这些种子可以准确有效区分 sense A 和 sense B。这一操作可以通过选择字典词条的词义来完成。

这些词组如果与目标词相邻，则倾向于有更强的影响 (effect)，这些影响 (effect) 随着距离逐渐减弱。

通过 Yarowsky 在 1993 年给出的标准 (criteria)，那些出现在最为可信的目标词组搭配关系的种子词 (seed words) 将被选中。

在 predicate-argument relationship 中，词语的影响相比于以和目标词相同的距离随意组合要更强。有内容词 (content words) 的搭配词组的影响要比有功能词 (function words) 的更强。



以上知，通过语料库，搭配词组可以与目标词有不同搭配关系。这可以提供一种词语区分评级 (word different rankings) 或者更甚给出区分分类 (different classifications)。

另外 (Alternatively) ，这一工作可以通过对每一分类明确(identifying) 一个单独定义搭配来完成，并且只对那些包含其中一个定义词 (defining words) 的上下文 (contexts) 来使用种子 (seeds) 。



一个公开可用的数据库 [WordNet](https://www.wikiwand.com/en/WordNet) 可被用于作为一个定义措辞 (terms) 的自动源 (automatic source) 。



除此外，那些出现在目标词附近的有很大频率的词会被分别选为搭配代表种子 (seed collocations representative) 。这一方法并非完全自动化，一个人类判断 (human judge) 应当决定哪一个词将被选为各目标词的意义，输出将成为意义的可信指标 (reliable indicators of the senses) 。




# 笔记：SeqGAN Sequence Generative Adversarial Nets with Policy Gradient

## 摘要

GAN 作为一种新的训练生成模型的方法，使用一个判别模型（D）去引导生成模型（G）的训练。这样一方法在生成实数数据（real-valued data）有很好的表现。



但是，它在生成离散标记序列（sequences of discrete tokens）方面却存在限制。主要原因在于，生成模型的离散输出在通过判别模型到生成模型的梯度更新时存在困难。除此外，判别模型仅能评估一个完整序列，对于部分生成序列，平衡序列目前分数和一旦生成了完整序列的未来分数是非平凡的（non-trivial）。



文章中提出了一个被称为 SeqGAN 的序列生成框架去解决上述问题。将数据生成器建模为强化学习的随机策略（stochastic policy in reinforcement learning）。强化学习的反馈信号从 GAN 的在完整序列上判断的判别器中出来，并使用蒙特卡洛搜索来传递回到中间状态机步骤。



在对合成数据与现实人物的广泛实验中，这一方法在强基线（strong baselines）上有明显的提升。



## introduction 中提及工作

* RNNs with LSTM cells (Hochreiter and Schmidhuber 1997)  have excellent performance ranging from natural language generation to handwriting generation (Wen et al. 2015; Graves 2013)

* the most common approach to training an RNN: maximize the log predictive likelihood of each true token in the training sequence given the previous observed tokens (Salakhutdinov 2009)

* (Bengio et al. 2015) found the maximum likelihood approaches suffer from **exposure bias** in the inference stage: the model generates a sequence iteratively and predicts next token conditioned on its previously predicted ones that my be never observed in the training data.

* to address the exposure bias, (Bengio et al. 2015) proposed a training strategy called **scheduled sampling (SS)** , where the generative model is partially fed with its own synthetic data as prefix (observed tokens) rather than the true data when deciding the next token in the training stage.

* BUT (Huszár 2015) showed that SS is an <u>inconsistent traning strategy</u> and <u>fails to address the problem fundamentally</u>.

* Another possible solution of the training/inference discrepancy problem: build the loss function on the entire generated sequence instead of each transition.

* In the application of machine translation, a task specific sequence score/loss, bilingual evaluation understudy, aka BLEU (Papineni et al. 2002) can be adopted to guide the sequence.

* BUT in many other practical applications like poem generation (Zhang and Lapata 2014) and chatbot (Hingston 2009) , a task specific loss may not be directly available to score a generated sequence accurately.

* GAN (Goodfellow and others 2014) is a promising framework for alleviation the above problem. This approach has been successful and been mostly applied in <u>computer vision tasks of generating samples of natural images (Denton et al. 2015)</u> .

* ① GAN is designed for generating <u>real-valued, continuous data</u>, but has **difficulties in directly generating sequences of discrete tokens like texts** (Huszár 2015) .If the generated data is based on discrete tokens, the “slight change” guidance from the discriminative net makes little sense because there is probably no corresponding token for such slight change in the limited dictionary space (Goodfellow 2016).

* ② GAN can <u>only give the score/loss for an entire sequence</u> when it has been generated.

* (Bachman and Precup 2015; Bahdanau et al. 2016) consider the sequence generation procedure as a sequential decision making process. 

  注：文章跟随此方法 fix ①

* this paper use Monte Carlo (MC) search to approximate the state-action value. train the policy (generative model) via policy gradient (Sutton et al. 1999), which naturally avoids the differentiation difficulty for discrete data in a conventional GAN.

  注：此方法 fix ②




## related word 中提及工作

* (Salakhutdinov 2009; Bengio et al. 2013). (Hinton, Osindero, and Teh 2006) first proposed to use the contrastive divergence algorithm to efficiently training deep belief nets (**DBN**). 

* (Bengio et al. 2013) proposed denoising autoencoder (**DAE**) that learns the data distribution in a supervised learning fashion.

  注：DBN and DAE  both learn a low dimensional representation (encoding) for each data instance and generate it from a decoding network.

* variational autoencoder (**VAE**) that combines deep learning with statistical inference intended to represent a data instance in a latent hidden space (Kingma and Welling 2014), while stillutilizing (deep) neural networks for non-linear mapping.

* Recurrent neural networks can be trained to produce sequences of tokens in many applications such as machine translation (Sutskever, Vinyals, and Le 2014; Bahdanau, Cho, and Bengio 2014).

  这一部分中与 introduction 部分有许多重复的部分，不一一列出

最后引出工作，SeqGAN extends GANs with the RL-based generator to solve the sequence generation problem. SeqGAN 使用的框架是 GANs + RL-based generator，修改了生成器的部分。



## SeqGAN 模型介绍

序列生成问题描述如下：

​	给定一个真实世界的结构化序列的数据集，训练一个以 $\theta$ 为参数的的生成模型 $G_{\theta}$ 去产生序列 $Y_{1:T}=(y_1, \cdots,y_t,\cdots,y_T),y_t\in \cal Y$ ，这个 $\cal Y$ 表示候选标记的词汇（the vocabulary of candidate tokens）。

​	文章基于强化学习来解释这一问题。在时间步长 $t$ 中，状态 $s$ 表示目前所产生标记 $(y_1,\cdots,y_{t-1})$ ，动作 $a$ 表示下一个要选的标记 $y_t$ 。因此策略模型 $G_{\theta}(y_t|Y_{1:t-1})$ 是随机的，而在选择动作后，状态的转换是确定的。

​	例如：

​	对于下一状态 $s'=Y_{1:t}$ 有 $\delta^a_{s,s'}=1$ 。如果现在状态 $s=Y_{1:t-1}$ 并且已选择行动 $a=y_t$ ，那么对于再下一个状态 $s''$ 便有 $\delta^a_{s,s'}=0$ 。

​	除此外，同样训练一个以 $\phi$ 为参数的判别模型 $D_{\phi}$ 去为改进判别器提供引导。 $D_{\phi}(Y_1:T)$ 标志着一个序列是否像真实数据的概率。
---
title:      区分性训练（Discriminative Training）及其在语音识别（ASR）上的运用
author:     liuyan
catalog:    true
tags:
  - 语音识别
  - deep learning
date:       2018-12-16 16:25:12
urlname:
categories: 语音识别
---

### 介绍

- 语音识别声学模型DNN训练通常用交叉熵（Cross-Entropy，CE）作为损失函数进行训练，但是在基于帧识别的语音识别中我们一般使用WER来评价语音识别的准确率，我们更关心的是序列的准确性，这就导致损失函数和训练目标不一致。
- 序列区分性训练（Discriminative Training，DT）在识别序列上定义误差，更接近我们语音识别的最终目标。
- 常见的DT目标函数有最大互信息（ maximum mutual information, MMI），增强型最大互信息（Boosted MMI, BMMI），最小音素错误（minimum phone error, MPE）和最小贝叶斯风险（minimum bayes risk, MBR） 

<!-- more -->

使用CE准则的时候，帧的正确率提高了但是最终解码的WER可能没有变化甚至变坏了，这也是我们想让训练准则直接和最后的评估函数（WER）相关的原因。 下面我们主要以MMI为主要介绍对象。

### GMM与DT

在语音GMM-HMM框架下，我们使用最大似然准则（Maximum likelihood estimation ）进行训练：
$$
F _ { \mathrm { ML } } = \sum _ { u = 1 } ^ { U } \log P \left( \mathbf { X } _ { u } |  W _ { u } ; \theta \right)
$$
其中$ W _ { u } $是标注序列，$ X _ { u } $是语音信号，$ \theta $是声学模型参数。

MMI准则（Maximum mutual information estimation ）为：
$$
\begin{aligned} F _ { \mathrm { MMI } } & = \sum _ { u = 1 } ^ { U } \log P \left( W _ { u } | \mathbf { x } _ { u } ; \theta \right) \\ = \sum _ { u = 1 } ^ { U } \log \frac { P \left( \mathbf { X } _ { u } | W _ { w } ; \theta \right) P \left( W _ { u } \right) } { \sum _ { w ^ { \prime } } P \left( \mathbf { X } _ { u } | w ^ { \prime } ; \theta \right) P \left( w ^ { \prime } \right) } \end{aligned}
$$
其中$ P ( W _ { u } ) $是固定的语言模型。

分子上的$ P ( \mathbf { X } _ { u } | W _ { u }  ; \theta )  $，正是ML的目标函数；而分母则是所有文本（包括训练文本和它的所有竞争者）产生训练语音的概率（按语言模型加权）和。

两者之间的区别在于条件概率不同。ML中只要训练文本产生训练语音的概率大就行，而MMI要求的是训练语音对应训练文本的概率大，就是要训练文本产生语音信号的概率与其它文本产生语音信号的概率之差大。 

### DNN与DT

#### MMI损失函数

在DNN神经网络中，DT准则可以替换CE准则作为损失函数。MMI准则为：
$$
\begin{aligned} { \mathcal { L } _ { M M I } (\theta ; \mathbb { S }) } & = \sum _ { m = 1 } ^ { M } \mathcal { L } _ { \mathrm { MMI } } \left( \theta ; \mathbf { o } ^ { m } , \mathbf { w } ^ { m } \right) \\ = \sum _ { m = 1 } ^ { M } \log P \left( \mathbf { w } ^ { m } | \mathbf { o } ^ { m } ; \theta \right) \\ = \sum _ { m = 1 } ^ { M } \log \frac { p \left( \mathbf { o } ^ { m } | \mathbf { s } ^ { m } ; \theta \right) ^ { \kappa } P \left( \mathbf { w } ^ { m } \right) } { \sum _ { \mathbf { w } } p \left( \mathbf { o } ^ { m } | \mathbf { s } ^ { w } ; \theta \right) ^ { \kappa } P ( \mathbf { w } ) } \end{aligned}
$$
其中$ \mathbf { o } ^ { m } = \mathbf { o } _ { 1 } ^ { m } , \cdots , \mathbf { o } _ { t } ^ { m } , \cdots , \mathbf { o } _ { T _ { m } } ^ { m } $和$ \mathbf { w } ^ { m } = \mathbf { w } _ { 1 } ^ { m } , \cdots , \mathbf { w } _ { t } ^ { m } , \cdots , \mathbf { w } _ { N _ { m } } ^ { m } $分别为第m个音频样本的观测序列和正确的单词标注序列，$ T _ { m } $为第m个音频样本的帧数，$ N _ { m } $是标注序列的单词总数，$ \theta $是模型参数，$ \mathbf { s } ^ { m } = \mathbf { s } _ { 1 } ^ { m } , \cdots , \mathbf { s } _ { t } ^ { m } , \cdots , \mathbf { s } _ { T _ { m } } ^ { m } $是$ w _ { m } $的状态序列，$ \kappa $是声学缩放系数。

MMI准则公式中，分子Numerator表示的是正确单词序列的可能性，分母Denominator是所有可能单词序列的可能性之和。

MMI准则最大化单词序列分布和观察序列分布之间的互信息,，减小句子错误率。最大化分子， 最小化分母。

DT训练之前需要使用CE准则生成alignments和lattices，DT的初始化模型为使用CE准则训练出的最好模型。

理论上说,，DT训练分母应该取遍所有可能的单词序列。不过在实际中，这个求和运算是限制在解码得到的lattice上做的，这样可以减少运算量。

DNN训练算法一般是用来最小化一个目标方程, 所以我们可以对MMI准则取反进行最小化，而不是最大化互信息。

#### MMI求导

$$
\frac { \partial \mathcal { L } _ { MMI}  (\theta ; \mathbb { S }) } { \partial \theta } = \sum _ { m = 1 } ^ { M } \sum _ { t = 1 } ^ { T } \frac { \partial \mathcal { L } (\theta ; \mathbb { S } ) } { \partial z _ { mt } } \frac { \partial z _ { mt } } { \partial \theta }
$$
其中$ z _ { mt } $表示softmax层的输入，$ \frac { \partial z _ { mt } } { \partial \theta } $的计算方式和CE准则是一样的没有区别，我们称之为外导数。真正和DT相关的是$ \frac { \partial \mathcal { L } (\theta ; \mathbb { S } ) } { \partial z _ { mt } } $，我们称之为内导数。

$$
\frac { \partial \mathscr { L } _ { M M I } ( \theta ; \mathbb { S } ) } { \partial z _ { m t } ( i ) } = \kappa \left( \gamma _ { m t } ^ { n u m } ( i ) - \gamma _ { m t } ^ { d e n } ( i ) \right)
$$
其中$ z _ { m t } (i ) $是$ z _ {m t } $的第i个元素，最终求导结果如上过程省略。$ \gamma _ { mt } ^ { n u m } ( r ) $和$ \gamma _ { mt } ^ { d e n } ( r ) $分别表示t时刻分子lattice和分母lattice在状态r上的后验概率，可以通过在lattice上使用[前向-后向算法](https://liuyanfeier.github.io/2019/01/07/viterbi%E4%BB%A5%E5%8F%8Aforward-backword%E7%AE%97%E6%B3%95/)得到：

$$
\gamma _ { t } ( r ) = P \left( s _ { t } = r | ( \theta ; \mathbb { S } ) \right) = \frac { \alpha _ { t } ( r ) \beta _ { t } ( r ) } { \sum _ { r } ^ { N } \alpha _ { t } ( r ) }
$$
其中$ \alpha _ { t } ( r ) $和$ \beta _ { t } ( r ) $分别是前向后向算法中计算出来的前向因子和后向因子，$ \sum _ { r } ^ { N } \alpha _ { t } ( r ) $是前向或者后向算法中从起始节点到结束节点的概率和。

前向后向算法都是在lattice上进行的，下图是一个word级别的lattice：

![](1.png)

#### kaldi中的DT实现

流程图：
![](2.png)

该流程图是kaldi nnet3中的DT计算过程，其中的lattice rescore是DNN前向计算出$ P ( s | o ) $，除以先验概率$ P ( s ) $，得到似然概率$ P ( o | s ) $，替换lattice边上对应的$ P ( o | s ) $。
其中`vector<BaseFloat> answers`是前面计算出来的DNN的输出对应参考pdf的概率。
```c++
// nnet3/discriminative-training.cc
// 对lattice进行声学校正, 将负（缩放）声学对数似然置于lattice的弧中
size_t DiscriminativeComputation::LatticeAcousticRescore(
    const std::vector<BaseFloat> &answers,
    size_t index, Lattice *lat) {
  int32 num_states = lat->NumStates();

  for (StateId s = 0; s < num_states; s++) {
    for (fst::MutableArcIterator<Lattice> aiter(lat, s);
         !aiter.Done(); aiter.Next()) {
      Arc arc = aiter.Value();
      if (arc.ilabel != 0) { // input-side has transition-ids, output-side empty
        arc.weight.SetValue2(-answers[index]);
        // graph cost: lm + transition + pronunciation
        // acoustic cost: -P(o|s)
        index++;
        aiter.SetValue(arc);
      }
    }
    LatticeWeight final = lat->Final(s);
    if (final != LatticeWeight::Zero()) {
      final.SetValue2(0.0); // 确保在最终概率中没有声学项
      lat->SetFinal(s, final);
    }
  }

  // 用于rescore lattice的对数似然的索引个数
  return index;
}
```

在分母lattice上进行前向后向的计算函数为LatticeForwardBackward；在分子lattice的前向后向计算函数为AlignmentToPosterior。
```c++
// lat/lattice-functions.cc
// 在lattice上执行前向后向算法并计算弧的后验概率
BaseFloat LatticeForwardBackward(const Lattice &lat, Posterior *post,
                                 double *acoustic_like_sum) {
  // 注意Posterior定义如下:  
  // Indexed [frame], then a list of (transition-id, posterior-probability) pairs.
  // typedef std::vector<std::vector<std::pair<int32, BaseFloat> > > Posterior;
  using namespace fst;
  typedef Lattice::Arc Arc;
  typedef Arc::Weight Weight;
  typedef Arc::StateId StateId;

  if (acoustic_like_sum) *acoustic_like_sum = 0.0;

  // 确保lattices在拓扑上排序
  if (lat.Properties(fst::kTopSorted, true) == 0)
    KALDI_ERR << "Input lattice must be topologically sorted.";
  KALDI_ASSERT(lat.Start() == 0);

  int32 num_states = lat.NumStates();
  vector<int32> state_times;
  // 拓扑迭代den_lats中的每个state，并按顺序对每个state进行计数，结果
  // 保存在vector state_times中，最后一个state对应的计数时间应该为帧的数量值
  int32 max_time = LatticeStateTimes(lat, &state_times);
  std::vector<double> alpha(num_states, kLogZeroDouble);
  std::vector<double> &beta(alpha); 
  // 重用相同的内存，beta是alpha的引用
  double tot_forward_prob = kLogZeroDouble;

  post->clear();
  post->resize(max_time);

  alpha[0] = 0.0;
  // Propagate alphas forward.
  for (StateId s = 0; s < num_states; s++) {
    double this_alpha = alpha[s];
    // alpha[]里面存储着从init state走到该state的所有路径中cost value加和最大值
    for (ArcIterator<Lattice> aiter(lat, s); !aiter.Done(); aiter.Next()) {
      const Arc &arc = aiter.Value();
      // arc_like = arc.weight.value1 + arc.weight.value2
      double arc_like = -ConvertToCost(arc.weight);    
      // LogAdd返回两者之间较大的一个... + something else
      alpha[arc.nextstate] = LogAdd(alpha[arc.nextstate], this_alpha + arc_like);
    }
    // get状态s的final weight; if == Weight::Zero() => non-final
    // final state上面有单独的语言和声学分 
    Weight f = lat.Final(s);
    if (f != Weight::Zero()) {     
      double final_like = this_alpha - (f.Value1() + f.Value2());
      tot_forward_prob = LogAdd(tot_forward_prob, final_like);
      KALDI_ASSERT(state_times[s] == max_time &&
                   "Lattice is inconsistent (final-prob not at max_time)");
    }
  }
  for (StateId s = num_states-1; s >= 0; s--) {
    Weight f = lat.Final(s);
    // 如果s不是final state, this_beta = 0
    double this_beta = -(f.Value1() + f.Value2());
    // beta[]里面存储的是从该state走到final state的所有路径中cost value加和最大值（加上final state的权值）
    for (ArcIterator<Lattice> aiter(lat, s); !aiter.Done(); aiter.Next()) {
      const Arc &arc = aiter.Value();
      double arc_like = -ConvertToCost(arc.weight),
          arc_beta = beta[arc.nextstate] + arc_like;
      this_beta = LogAdd(this_beta, arc_beta);
      int32 transition_id = arc.ilabel;

      // 该if判断是一个优化，以避免不需要的exp()函数
      if (transition_id != 0 || acoustic_like_sum != NULL) {
        double posterior = Exp(alpha[s] + arc_beta - tot_forward_prob);

        if (transition_id != 0) // 该弧上有tid，不是epsilon
          // (*post)[state_times[s]]是以时间帧为编号的vector
          (*post)[state_times[s]].push_back(std::make_pair(transition_id,
                                                           static_cast<kaldi::BaseFloat>(posterior)));
        if (acoustic_like_sum != NULL)
          *acoustic_like_sum -= posterior * arc.weight.Value2();
      }
    }
    if (acoustic_like_sum != NULL && f != Weight::Zero()) {
      double final_logprob = - ConvertToCost(f),
          posterior = Exp(alpha[s] + final_logprob - tot_forward_prob);
      *acoustic_like_sum -= posterior * f.Value2();     // value2声学分数 
    }
    beta[s] = this_beta;
  }
  double tot_backward_prob = beta[0];
  if (!ApproxEqual(tot_forward_prob, tot_backward_prob, 1e-8)) {
    KALDI_WARN << "Total forward probability over lattice = " << tot_forward_prob
              << ", while total backward probability = " << tot_backward_prob;
  }
  // 按照第一个元素排序，把tid(pdfid)相同的后验combine起来(posterior值加起来)
  for (int32 t = 0; t < max_time; t++)
    MergePairVectorSumming(&((*post)[t]));
  return tot_backward_prob;
}
```

#### trick

- frame rejection
当分子alignment的状态没有在分母lattice中出现的时候，会导致梯度过大，舍弃该帧的梯度。这种情况对于silence帧尤其常见，因为silence经常出现在分子的lattice，但是很容易被分母的lattice忽略。
如果新的对齐和lattice在每轮训练后被重新生成，那么运算结果将得到进一步改进。

```c++
// 如果两个post[i]的第一个元素(tid)没有交集，返回true
    if (PosteriorEntriesAreDisjoint(post1[i], post2[i])) {
      num_disjoint++;
      if (drop_frames)
        (*post)[i].clear();
    }
```

- 帧平滑
$$
J _ { \mathrm { FS } - \mathrm { SEQ } } ( \theta ; \mathrm { S } ) = ( 1 - H ) J _ { \mathrm { CE } } ( \theta ; \mathrm { S } ) + H J _ { \mathrm { SEQ } } ( \theta ; \mathrm { S } )
$$
当训练dt准则函数持续改进时，只用DNN计算出的帧准确率却显著变差。帧/序列的比从 1:4 (H = 4/s )到 1:10 (H = 10/11 )常常是有效的。

- 更小的lr，大数据集上smbr效果最好

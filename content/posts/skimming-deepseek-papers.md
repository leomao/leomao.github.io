---
title: "Deepseek 論文略讀心得"
date: 2025-01-28T23:25:55+08:00
slug: "skimming-deepseek-papers"
tags: ["papers"]
math: true
---

這陣子有點被 Deepseek 洗版，於是就稍微瞄了一下 Deepseek 這系列的幾篇論文：
* [DeepSeekMath]
* [DeepSeek-V3 Technical Report]
* [DeepSeek-R1]

因為沒有看得非常仔細，就先隨性寫一點自己比較目前有看到且有興趣的部分。剩下的之後興致來了再說。

## About papers

這幾篇論文有把很多細節寫出來我還滿喜歡的。有看到 unsuccessful attempts
滿開心的，希望以後的論文都可以多寫一點。

[DeepSeek-V3 Technical Report] 有提到不少關於如何讓整個 pipeline
跑得快的細節，很有趣。

整體來說論文寫得滿詳細的而且又有 open source，好像已經有 reproduce
成功的案例了？以工程的角度來說這幾篇真的滿有價值的。

## MoE (Mixture of Experts) + Auxiliary-Loss-Free Load Balancing

它這邊用的 MoE 是 layer 層級的，然後有規定 $N_s$ 個 experts 是一定要用，剩下
$N_r$ 個裡面會選 top $k$ 個來用。然後 load balancing 是直接對每個 expert
額外記錄一個 bias 在取 top $k$ 的時候加上去來調節。這些 bias
項有自己的更新方式並且只是用來控制 routing 的，輸出的值還是用原本的 scores 做
weighted sum。

## MTP (Multi-Token Prediction)

簡單來說就是除了原本的 next token cross entropy loss 以外，他在 model
後面串了幾個單層的 tranformer block (每個前面有疊 RMSNorm 跟一個 linear
projection) 來 prediction $t+1, \dots, t+k$ tokens。這個額外的 loss
主要是在訓練的時候使用，會讓 model 本體在最後 output head 前的 embedding
變得更容易 (讓後面的小 models) 預測接下來的 tokens。

## GRPO (Group Relative Policy Optimization)

很多 RL 的演算法都是假設跟環境互動很貴，而且可能不能在某個特定 state
多次嘗試。但如果沒有這些限制，而且反過來說是訓練 model 比較貴的話呢？

GRPO 看起來就是把 value model 拔掉然後直接多 sample 一些結果出來做 PPO。其中
advantage 就直接拿這組 sample 的 reward 來估。實際上論文中不只平移 baseline
(mean) 還除標準差 (standard deviation)。

我其實不確定除標準差之後是否還是原本 gradient 的 unbiased estimator，但反正
PPO 做 clipping 就已經很隨性了，更不用說這邊整個 reward 也都是 reward model
生出來的。而 reward model 也會在訓練過程中一直變化。總之我姑且就當作是某種調整
graident 幅度的方式，訓練能穩定可能才是最重要的。

話說回來，之前還有看到一篇 [REBEL]，不知道 Deepseek 這個架構用 REBEL 的 loss
來訓練會怎樣。

[DeepSeekMath]: https://arxiv.org/abs/2402.03300
[DeepSeek-V3 Technical Report]: https://arxiv.org/abs/2412.19437
[DeepSeek-R1]: https://arxiv.org/abs/2501.12948
[REBEL]: https://arxiv.org/abs/2404.16767


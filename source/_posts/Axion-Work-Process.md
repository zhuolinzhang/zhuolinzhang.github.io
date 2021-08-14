---
title: Axion Work Process
date: 2021-06-22 17:30:35
tags: ALP
---

## 2021-08

### 14

今天用`MadGraph5`尝试做了一下MC模拟。因为最近对于EFT的学习以及和万老师再次讨论有了思路。

在`SMEFTatNLO`模型中，我们通过

```
import model SMEFTatNLO-full
generate g g > z z QCD=2 QED=2 NP=2 NP^2=2 [virt=QCD]
```

生成MC样本。最后可以得到$gg \rightarrow h \rightarrow zz$ 过程和$gg\rightarrow zz$的box图的干涉。这也是万老师希望做的。但是今天又跟万老师讨论了一下发现她有需要去做一个有ALP的样本。那么我们就不能用`SMEFTatNLO`来完成这项工作。

在`Feynrules`网站上，我找到了一个UFO，即`ALP_linear_UFO`。这个模型倒是有我们想要的$gg\rightarrow a \rightarrow zz$过程，但是它不能处理NLO过程。所以最后这项工作恐怕还需要自己写一个`Feynrules`的UFO来处理。

今天遗留的问题：

- 既然昨天的讨论中说，ALP的有效顶点与Higgs的有效顶点是一样的，那么为什么我们不能直接改Higgs的质量把它当作ALP？
- 在`ALP_linea_UFO`中，ALP的PDG ID是9000005，这该怎么理解？
- 为什么`SMEFTatNLO-full`包含了$ggH$过程？之前用写入了cpq3i、cpW、cpWB的restrict card去产生$ggH$过程却失败了，这是为什么？相比`SMEFTatNLO-full`，缺少了什么Wilson coefficients？或者是Yukawa coupling constant？

可能需要做的事情：

- 学习如何写`Feynrules`UFO；
- 考虑用JHUGen或者POWHEG去产生这些过程。

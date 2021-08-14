---
title: The Experiment Log of The Study of HIG-18-016
date: 2021-05-04 23:59:22
tags: CMS
---

# 2021

## May

### 04

学习`python`中的类，主要阅读《python学习手册》一书的内容。第六部分还有两章，但是看不下去了。看程序书实在是枯燥。

这两周（至5.13）需要完成的任务有：

1. 检查用不同的代码生成的`TTree`作图却产生迥异结果的原因
2. 重构画图代码
3. 写一个计算cutflow的小程序

当前思路：

1. 首先检查老代码的cut条件与新代码的cut条件是否一致。随后可以重新提交老代码。之后利用`uproot`，将信号样本的`TTree`导出成文本文件。随后，利用`difflib`检查两个文本文件的区别在哪里。如果新代码的结果是老代码的子集，那么检查新代码的flat `TTree`文件。但我感觉这样有可能依然无法找到问题所在。
2. 将脚本完全写成OOP的。将一些方法，例如scale，放在class histogram中。最好能抛弃exec的动态调用，再class histogram中加入一些属性，实现直方图分类。另外，做ratio图部分的代码也需要重构。尽量避免复制粘贴一段代码的事情，减少维护的工作量。此外，可以考虑将信号的stack图改成将信号放大1000倍（举个栗子）的形式。
3. 将结果输入成`numpy`矩阵。根据文件名？分类。Then, scale the number and sum it. We should also calculate the ratio of pass and all. In the end, we can get the final result.

任务很重，后面几天到8号没有集中的学习的时间，需要抓紧。

### 05

阅读《python学习手册》（第四版）32章-33章，33章未看完。

### 10

修改以前写的cut好的`ZHNtuple`程序，与新的不做cut的程序的变量命名和cut条件保持一致。并提交给CRAB。

一个可能的问题的解决方法：如果`scram b`没有激活`edmplugin`，运行cmsRun提示`Unable to find plugin '' in category 'CMS EDM Framework Module'.`

彻底清除cache，删掉警告的存在重复命名的目录。

```bash
scram b distclean && scram b vclean && scram b clean
```

### 11

找到了存在4日任务1问题的原因。老的workflow，即保存cut后的`TTree`的analyzer脚本是正确的。在我之前的workflow中，我找到的muon和jet对都是我希望所找的。而在当前的workflow中，我为了图省事，全部选取了leading particles作为筛选对象。但实际上，leading particle不一定满足我需要的cut条件。而这种情况下，显然不能只选取leading particles去做cut。只能说如果全部cut完，还有多个粒子对满足条件，此时再选取leading particles作为我需要的粒子。

但是只是找到了原因。怎么改还是毫无头绪。应该把我的workflow反过来，先cut再把`TTree`变成flat。但我不知道`RDataFrame`怎么去做非flat `TTree`的cut。我之前想着加All但是目前发现这样好像行不通。muon貌似正常，jet总是不对。莫名其妙。

重构代码进度缓慢。目前大概把新的hist类写完了。离完成还差得远。先解决上一个问题再说吧。

### 12

虽然确定了问题，但是找不到解决方案。`RDataFrame`似乎无法解决问题。其中的`Filter`方法只能对每一个事例做判选。没有办法实现我想要的功能。即：loop每一个事例中的vector，留下我想要的元素，剔除我不要的元素。我写了一个lambda函数，

```cpp
d.Filter([](std::vector<float> &mu1Pt, std::vector<float> &mu2Pt){for (auto i1 : mu1Pt) {for (auto i2 : mu2Pt) {if (i1 > 25 && i2 > 15) return true; else return false;}}}, {"mu1Pt", "mu2Pt"}, "Muon pt");
```

但不幸的是它只能loop每一个vector的第0个元素。所以恐怕`RDataFrame`无法解决这个问题。

目前的思路是利用`TTreeReader`，重写muon1，muon2，jet1，jet2四个vector，随后对这4个vector进行判选，同时也需要避免重复。

但我认为与其这么麻烦还不如回到老的workflow上去。

### 14

总算是将任务1全部完成了。新代码完全使用`TTreeReader`，损失了一些速度，但是物理是正确的。而且加入了cutflow的功能。相比原来的代码功能上保持了一致。

想想最开始写cutflow的时候是加上不同的cut条件画出不同的不变质量谱。复制粘贴了好几个for循环。又丑又笨。当你在复制粘贴程序的时候就应该意识到写一个函数来完成这些操作。而不是复制粘贴。这次用了相对巧妙一些的办法。每通过一个cut条件，计数器+1，最后选取这一个entry里得分最高的作为这个entry的分数。随后根据不同的分数计算cutflow。目前看这么做可行。

目前还有一点隐忧就是cut好的analyzer出来的TTree的事例数与这种方法出来的事例数仍然有100左右的差距。需要通过`uproot`导出然后利用`pandas`比较差异。看看到底怎么回事。

接下来就可以全力以赴重构画图脚本了。当然cutflow的事情还不算完，还需要写一个脚本把这些cutflow归类处理。希望可以利用`pandas`完成这项工作。

### 17

I have fixed bugs in the new skim macro. But There is still one problem in the new macro. An event passed all selections in the new workflow, but this event can't pass all selections in the old workflow. I don't understand the reason. Let me add the cutflow function in the old macro, then, the reason may come out.

### 30

很久没有更新了。因为太忙有时候就忘了更新，久而久之就不记得了。

这一段时间完成了从旧的workflow向新的workflow的切换。彻底修好了skim脚本的bug。彻底重构了画图脚本。这也是我第一次写的面向对象的程序。目前，根据CMS的要求，下一周要把数据从ReReco切换到UL。

此外，跟老板讨论了申请博士的问题。前景依然不明朗。

此外还要接一个唯象的工作，分析AZZ。A指axion。不知道我已经快要还光的理论知识还能不能hold住这个题目。但是别无他法。毕竟实验的工作前景不明朗。

## June

### 22

在12号的大组会上给了报告。报告本身挺翻车的。但是事后的建议让我认识到了组里的高效。我认为有时间的话还是应该将分析从MINIAOD迁移到NANOAOD上。NANOAOD的工具比MINIAOD要简单非常多。

目前需要做的事情是把$\sigma \times BR$关于$p_T$和$m_{dijet}$的分布给做出来。这个图在之前的CMSDAS的long exercise上做过。但当时只是作图，没有把代码看清楚。我不是很清楚通过combine计算出来的signal strength是怎么转化为limit的。如果这个问题搞明白了后面就简单了。

### 28

MC的一些总结参见：[A Note on Analysis Based on MC Generators - Zhuolin's blog (zhuolinzhang.github.io)](https://zhuolinzhang.github.io/2021/06/29/A-Note-on-Analysis-Based-on-MC-Generators/)

目前我需要看理论的$p p \rightarrow z h$的微分截面关于$p_T^{dijet}$的分布。怎么看呢？要么手算，要么用MC generator生成。我选择后者。

这个样本

```
dataset=/ZH_HToBB_ZToLL_M125_13TeV_powheg_pythia8/RunIIAutumn18MiniAOD-102X_upgrade2018_realistic_v15-v2/MINIAODSIM
```

却使用powheg产生的。很不幸，我没用过powheg。这几天为此颇费脑筋。

首先我在本地用MG5产生了一个

```
generate p p > z > z h, z > mu+ mu-, h > b b~
```

过程的样本。可是这个过程的截面是$0.01607\ \mathrm{pb}$左右。与该样本的截面$0.07924\ \mathrm{pb}$相去甚远。所以我想恐怕是物理过程有区别。

在McM里，Select View勾选Fragment，可以看到生成样本使用的`_cff.py`文件。勾选Generator parameters可以看到截面。从cff文件可以看到产生样本使用的card以及pythia8的设置。打开card，发现

```
vdecaymode   11   ! 1: e, 2: mu, 3: tau, 4:nu_e, 5: nu_mu, 6: nu_tau, 0: hadronic, 10: inclusive, 11: inclusive leptons, 12: inclusive neutrinos
hdecaymode  -1   ! for PYTHIA or HERWIG: do not decay the Higgs boson
```

果然过程不一样。从XSDB里可以看到这个截面是NLO的，那恐怕powheg产生的样本是NLO样本。于是在MG5中

```
define l+ = e+ mu+ ta+ # default l+ excluded tau
define l- = e- mu- ta-
generate p p > z > h l+ l- [QCD]
```

这样产生的过程是NLO的。可以注意到这里Higgs没有衰变。随后在Pythia8的card里设置

```
DM_1 = 25:m0 = 125.0
DM_2 = 25:onMode = off
DM_3 = 25:onIfMatch = 5 -5 # Higgs decay to b b~
```

最后得到的截面为$0.07618 \pm 9.5\times 10^{-5}\ \mathrm{pb}$。结果已经很接近了。但目前还需要确认$\tau$在不在powheg的$Z$的衰变产物内。如果没有$\tau$子恐怕又要头疼一阵了。

一些可能有用的链接：

- [PowhegBOXPrecompiled < CMS < TWiki (cern.ch)](https://twiki.cern.ch/twiki/bin/viewauth/CMS/PowhegBOXPrecompiled) powheg的tutorial。目前我试图在HTCondor上提交讲gridpack转换成LHE的作业，一直不成功。暂时没有找到原因。目前只好在lxplus上直接运行，但是速度特别特别慢。
- [HowToGenXSecAnalyzer < CMS < TWiki (cern.ch)](https://twiki.cern.ch/twiki/bin/viewauth/CMS/HowToGenXSecAnalyzer)可以计算任意CMS官方MC样本的截面。
- [cms-sw/genproductions: Generator fragments for MC production (github.com)](https://github.com/cms-sw/genproductions) CMSSW关于generator的仓库。有各种各样的MC generator cards(`bin/`)，Pythia8的参数(`genfragments/`)和第二个链接的算截面的工具。

## August

### 14

13日在组会上作了报告，但是反响平平。这一个多月完成了$\sigma(ZH) \times BR(Z \rightarrow l^+ l^-)$的upper limit关于$p_T$的分布图。目前存在三个问题

- 如何在lxplus上生成$ZH$的EFT样本？目前这种先写restrict_card，再产生样本的workflow，是无法在lxplus上工作的。
- MG5 reweighting 的原理是什么？怎么用reweighting的方法产生MC样本？怎样找到$ZH$ EFT的benchmark point？
- 为什么将信号强度放缩到截面分布的放缩因子是pfjets的截面分布？

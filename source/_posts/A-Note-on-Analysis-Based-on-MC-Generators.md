---
title: A Note on Analysis Based on MC Generators
date: 2021-06-29 02:42:54
tags: CMS
---

这几天主要在玩`MadGraph5@NLO`，正好轴子的工作也需要好好玩MadGraph，一举两得。这一周实验上的工作是需要看一看理论上的$p p \rightarrow z \rightarrow z h$的微分截面关于$p_T^{dijet}$的分布。 在此过程中遇到了很多问题。目前我已经熟练地掌握了用MadGraph产生MC样本以及对其分析的技巧。尽管我本科就玩过，当时没有玩明白。madgraph的语法非常简单，通常我们用madgraph产生样本之后，用Pythia8做parton shower/hadronization，随后用Delphes对Pythia8产生的文件做探测器模拟。每一个环节都可以在card里修改参数，得到我想要的结果。

Madgraph在parton level会产生`.lhe`文件。这个文件是一个XML语法的文本文件。读法参见[arXiv:hep-ph/0609017](https://arxiv.org/abs/hep-ph/0609017)和[arXiv:hep-ph/0109068v1]( https://arxiv.org/abs/hep-ph/0109068v1)。LHE文件详细地给出了每个事例的物理过程。Pythia8会输出一个`.hepmc`文件。这个文件目前我还不知道是怎么读的。但是今晚找到了HepMC的官网，读取问题有望得到解决。MadAnalysis可以直接从`.lhe`和`.hepmc`中做我们需要的直方图。如果我们需要的直方图在自动生成范围外，可以在MadAnalysis的card里输入我们需要的直方图。Delphes最后输出的是一个ROOT文件。这倒省事了。下面我来总结一下我目前遇到的坑。


## Madgraph

### 语法

Madgraph的语法非常简单。从[MadgraphWiki](https://cp3.irmp.ucl.ac.be/projects/madgraph/wiki/InputEx)摘录如下：

> - Use a ">" to separate the initial state particle(s) from the final state ones.
>
> - Initial state can be one (decay) or two (scattering) particles.
>
> - Use " x x > z > y y y " form to require particle z to appear in a s-channel intermediate state (not recommended due to gauge invariance).
>
> - Use " x x > y y y $ z" form to forbid particle z from appearing in a s-channel intermediate state (not recommended due to gauge invariance).
>
> - Use " x x > y y y / z" form to exclude particle z to appear as internal particle in any diagram.
>
> - Use comma to specify decay chains. For instance "x x > Z W, Z >y y, W > k k" means "x x > Z W" where Z then decays to yy and W to kk.
> - Coupling orders are automatically detected to generate the leading order processes. To explicitly specify coupling orders, add them after the process on the same line:
>   x x > y y y / z QCD=0
>   x x > Z W QED=2, Z > y y, W > k k
>   Note that if a coupling order is omitted, it defaults to infinity (or, for restricted couplings such as HIG in the HEFT model, to the restricted value).
> - Use the multi-label "p" to indicate a proton or an anti-proton.
>   The symbol "p~" does not exist. In fact the type of initial state (pp or ppbar or parton-parton fixed energy) is specified later, during the run, in the run_card.dat.
>   The multi-lable "j" suggests a final-state jet.
> - Note that the syntax for [MadGraph](https://cp3.irmp.ucl.ac.be/projects/madgraph/wiki/MadGraph) 5 differs slightly from previous versions: a space (" ") is required between particle names, and the decay chain syntax has been upgraded to a more powerful syntax which allows better control of the different processes.
>

这里说的是`generate`的语法。在`generate`之后执行`output name`就会在`bin`目录下产生一个`name`子目录。里面的`index.html`上有MC运行的结果和过程的费曼图信息。随后执行`launch`就进入了执行界面。在执行时可以输入数字对应的开关，切换和开关各个选项。缺少的模块可以使用`install`安装。例如

```
install pythia8
```

就会自动下载安装Pythia8。选项部分结束后，可以更改涉及到的所有card。完成上述所有过程，MC就开始跑了。

### 坑

MadGraph5 v3.1.x似乎对语法的要求更严格了，很多以前可以run通的语法在这个版本会报错。这时候需要修改设置。

```
set acknowledged_v3.1_syntax true --global
```

这样MadGraph v2.x的语法也可以使用了。

在WSL里使用是还有很多额外的坑。因为WSL自带的包比较少，所以会出现画不出费曼图的问题。安装gs就可以解决这个问题。在Ubuntu中，可以通过apt直接安装gs。此外，MadAnalysis需要pdflatex才能生成PDF的分析报告，这也需要注意。

### 分析

MadAnalysis可以直接分析`.lhe`文件。此外，我们还可以使用`ExRootAnalysis`将`.lhe`变成`.root`。安装方法同上，傻瓜式安装。这样就可以产生我们这些实验民工喜欢的ROOT文件了。我们既可以在MadGraph里打开开关让它在运行的同时直接把`.lhe`变成`.root`，也可以进入ExRootAnalysis所在的文件夹，执行

```bash
./ExRootLHEFConverter <input .lhe file> <output .root file>
```

就能将`.lhe`文件转化成`.root`文件。

查看这个ROOT文件，需要在root里输入

```cpp
gSystem->Load("libExRootAnalysis.so")
```

才能正确查看。当然，写一个`rootlogon.C`就不用每次打开文件都要这么敲一遍了。

## Pythia8

我们通常使用Pythia8做parton shower和hadronization。Pythia8会产生一个`.hepmc`文件。非常大。看起来我这里使用的Pythia 8.245会产生HepMC2文件。HepMC2与HepMC3是不一样的。此前我一直不清楚怎么分析这个文件，但是今晚无意间发现了这个网站[HepMC3 event record library: Main page (cern.ch)](http://hepmc.web.cern.ch/hepmc/index.html)。看起来这个问题有望得到解决。

而且conda-forge里有HepMC3和HepMC2的包。参见[Hepmc2 :: Anaconda.org](https://anaconda.org/conda-forge/hepmc2)。也许这下连安装都不用了。这两天我来试一试。

## Delphes3

CMS使用Geant4做探测器模拟。但是MadGraph没有集成Geant4，集成了Delphes。安装方法同上。可以看到Delphes的cards里就有CMS的card。当然，因为CMS官方不使用Delphes，这个card的内容可能还需要检验。

Delphes最后会产生一个`.root`文件。这下终于不用转换了。与ExRootAnalysis类似，在root中加载动态链接库才能正确读取Delphes产生的ROOT文件。

```cpp
gSystem->Load("libDelphes.so")
```

目前macOS Big Sur不能正确地安装Delphes，编译到某个步骤会报错，不幸的是我的x86的Mac Mini也中招了。我在官网看到3个有关的tickets了。需要Delphes官方修复。

Delphes产生的ROOT文件并不是一个flat TTree。我用`RDataFrame`也没读出来。不是说`RDataFrame`一定不行，我的`RDataFrame`学的不好。`RDataFrame`对cpp的要求相对来说比较高，我的cpp水平不太好，学不下去。但是用`TTreeReaderArray`可以读。

官方推荐使用`ExRootTreeReader`做分析。没错，就是上面的`ExRootAnalysis`。目前没有时间学这个。先用`TTreeReaderArray`做吧。




---
title: The Experiment Log of The Study of HIG-18-016
date: 2021-05-04 23:59:22
tags: CMS
---

# 2021

## May

### 04

From now on, I will write the experiment log everyday. I will try to write this in English. However, for my poor English, it is hard to write every log in English. I will try my best.

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


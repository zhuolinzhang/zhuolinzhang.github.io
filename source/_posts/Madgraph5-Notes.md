---
title: Madgraph5 Notes
date: 2021-06-22 17:30:56
tags:
---

# SMEFT model

# Parton Shower

## 调用pythia8出现的错误

#### Splitting .lhe event file for PY8 parallelization... 后迅速出现错误

可能是pythia8的并行计算出现了问题，在madgraph中输入

```
set nb_core 1
```

后关闭并行计算，重新`launch`即可避免此错误。

#### `PYTHIA Abort from Pythia::Pythia: unmatched version numbers : in code 8.235 but in XML 8.230`

对于我来说是因为通过`conda`安装了`ROOT`，并且在`conda`的`ROOT`环境中运行了madgraph。而我的`ROOT`环境中的`pythia8`版本与madgraph安装的pythia8版本不一致。在`conda`环境外重新安装运行madgraph即可避免此问题。

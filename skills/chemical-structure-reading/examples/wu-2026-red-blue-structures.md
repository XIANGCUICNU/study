# Red-to-Blue Photon Upconversion：分子结构提取

## 文献与范围

Yun Wu 等，*Red-to-Blue Photon Upconversion Based on BODIPY Derivative Photosensitizers*，*Chemistry – A European Journal*，2026，[DOI: 10.1002/chem.71254](https://doi.org/10.1002/chem.71254)。

依据用户提供的正文 PDF，主要读取第 2 页 **Scheme 1b**。该图展示本研究使用的两个敏化剂 2G、FD 和两个湮灭剂 Perylene、BPEA；Scheme 1a 中的结构属于既往文献，不作为本研究样品。本次未获得 SI，未读取 Scheme 1b 之外的结构文件，也未自动运行图像识别模型。

## 提取结果

| 分子 | 研究角色 | 图中可辨结构 | 状态 |
| --- | --- | --- | --- |
| **2G** | 三重态敏化剂 | 两个 BODIPY 单元通过单键连接；两端各一个 Br。每个 BODIPY 的芳基带有 `OC₁₂H₂₅` 取代基。 | 主要模块、连接类型和溴的数量经原图目视核对；完整 SMILES、全部取代位点和立体信息未独立确认。 |
| **FD** | 三重态敏化剂 | 两个 BODIPY 单元与中央含 S 的稠合结构连接；图中两端各标有一个 Cl。 | 主要模块和氯的数量经原图目视核对；中央稠环、硼氮连接和全部键级尚待逐键核对。 |
| **Perylene（苝）** | 湮灭剂 | 多环稠合芳烃；分子式 C₂₀H₁₂。 | 与原图及公开数据库结构对照；下列字符串经 RDKit 解析。 |
| **BPEA** | 湮灭剂 | 9,10-双(苯乙炔基)蒽；分子式 C₃₀H₁₈。 | 与原图及公开数据库结构对照；下列字符串经 RDKit 解析。 |

论文将 FD 描述为不含 Br、I 的“heavy-atom-free”敏化剂，但 Scheme 1b 中明确标有两个 Cl。因此，在复述作者的术语时不能写成“FD 完全不含卤素”，其具体术语范围应保留原文语境。

## 机器可读结构

下列 SMILES 来源于公开标准化合物信息，经 **RDKit 2026.09.1** 解析并计算分子式；与正文 Scheme 1b 目视对照。RDKit 可解析不构成对原图每条键的独立证明。

### Perylene

```text
c1cc2cccc3c4cccc5cccc(c(c1)c23)c54
```

RDKit 分子式：`C20H12`。由 [NIST Perylene 的 InChI](https://webbook.nist.gov/cgi/cbook.cgi?ID=198-55-0) 转换。

### BPEA

```text
C1=CC=C(C=C1)C#CC2=C3C=CC=CC3=C(C4=CC=CC=C42)C#CC5=CC=CC=C5
```

RDKit 分子式：`C30H18`。SMILES 来源：[PubChem CID 82338](https://pubchem.ncbi.nlm.nih.gov/compound/82338)。

### 2G 和 FD

目前不提供确定的完整 SMILES 或重绘图：SI 和原始结构文件未提供，2G 的芳基与长链以及 FD 的稠环、配位连接、键级需逐项复核。若获得 SI 的结构图、合成表征或作者的 ChemDraw/Molfile，可进一步生成候选结构、RDKit 重绘并与原图逐键对照。

## 验证边界

- 目视读取和 RDKit 解析分别报告；只有 Perylene、BPEA 使用了外部数据库标准结构。
- 用户提供的正文 PDF 及其原图未上传到仓库；本文件仅记录文字、文献来源和候选结构信息。
- 未根据结构图单独推断 ISC 量子产率、轨道分布或能级，也未核验 SI 的数据。

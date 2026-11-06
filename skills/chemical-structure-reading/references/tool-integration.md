# 化学结构识别：工具联用指南

本指南依据上游公开文档整理，核查日期 2026-11-06。项目接口和模型会变化；运行前以各项目文档为准。这里只提供操作路径，仓库没有附带模型权重、第三方代码或自动安装器。首次使用模型可能联网下载权重；未发表结构优先使用本地模型，核对工具是否会上传数据。

## 选择与交接

| 输入或需求 | 选择 | 交接给下一步 |
| --- | --- | --- |
| 单个结构截图 | MolParser 识别模式；或 MolScribe、DECIMER | 原图、原始输出、候选结构、工具/版本 |
| 多结构图或 PDF | MolParser 检测+识别 | 页码、检测框、编号对应、原始和规范化结果；人工检查遗漏 |
| Markush、缩写或重复单元 | MolParser 的 E-SMILES；必要时查看上游扩展格式 skill | 完整 E-SMILES、原图、变量定义和附着点；不能省略扩展部分 |
| 有 SMILES/SDF 的具体分子 | RDKit 或已安装的 RDKit skill | 输入原文、解析结果、分子式、电荷、重绘；对原图逐项比对 |
| 已知化合物的公开信息 | 可用的 PubChem MCP 或 PubChem | 查询词、CID、结构、查询日期；核对是否为同一盐和立体异构体 |
| 多分子数据质量/建模 | 已配置的 ChemLint MCP | 清洗前后映射、处理策略及错误；不当作 OCSR |

普通 SMILES 有时不足以表示 Markush、聚合物、配位或模糊连接；保留原文件与扩展表示。候选不可靠时不要强行传给下游作确定的性质计算。各程序、权重和依赖的许可需分别查看。

## 1. MolParser：图像/PDF 与扩展格式

来源：[MolParser](https://github.com/dptech-corp/MolParser)、[视觉 OCSR skill](https://github.com/dptech-corp/MolParser/tree/main/skills/molparser-visual-ocsr)、[E-SMILES skill](https://github.com/dptech-corp/MolParser/tree/main/skills/molparser-extended-smiles)。上游项目含 Apache-2.0 代码许可；模型权重及其他依赖的许可另行确认。本指南未复制其 skill 文件或代码。

按上游说明安装包和图片/PDF 推理的可选依赖。先确认运行环境、存储空间及模型来源。上游默认检测与识别模型可能自动下载，记录实际模型和版本。以下是上游接口的最小示意，**需已有模型及依赖**：

```python
from molparser import MolParser

parser = MolParser()
results = parser.parse("single_molecule.png", rec_only=True)
for result in results:
    print(result.to_dict())
```

整页图像使用 `rec_only=False`；PDF 使用 `parser.parse("paper.pdf")`。结果包含 input_index、零起始 page_index、bbox、smi、esmi、cxsmiles 等字段。bbox 为页面坐标，具体参照上游；`confidence` 为检测框置信度，不是分子结构准确率。单图识别或检测不到分子时可能没有 bbox 或结果。检查结构是否完整，并在论文中以人工页码和图号交代来源。

务必保留 raw_caption、esmi、groups 等完整结果及图中 R 基定义。smi 为普通 SMILES 候选，Markush 未解析时不应替代完整 E-SMILES。E-SMILES 的扩展部分不能直接作为普通 SMILES 传入 RDKit；上游提供规范化、渲染和尽力转换为 CXSMILES 的工具，转换后的附着位点和变量仍需与原图核对。

## 2. MolScribe / DECIMER：可选的独立识别

- [MolScribe](https://github.com/thomas0809/MolScribe)：提供图片到 SMILES、molfile 以及可选的原子/键与置信度结果。上游示例需要安装包、PyTorch 和下载模型检查点；适合对单结构裁图形成另一个候选。置信度不是经校准的正确概率。
- [DECIMER Image Transformer](https://github.com/Kohulan/DECIMER-Image_Transformer)：上游提供 `from DECIMER import predict_SMILES` 和 `predict_SMILES(image_path)`。依版本首次预测可能下载权重；在线 [DECIMER.ai](https://decimer.ai/) 会涉及外部服务。
- 比较两种输出时先说明是同一裁图；记录版本、模型、输出及失败。差异逐项定位到原子、键、位置和立体信息，不能因两者一致直接判真。

## 3. RDKit：解析、记录与重绘

[RDKit 官方文档](https://www.rdkit.org/docs/)、[K-Dense RDKit skill](https://github.com/K-Dense-AI/scientific-agent-skills/tree/main/skills/rdkit)。后者是单独项目；如在工作环境中安装该 skill，可在本流程完成识别候选后调用它。RDKit 本身必须另行安装；在干净的虚拟环境中按官方安装说明选择 conda-forge 或 PyPI。

对**已确定为普通分子的候选**，例如：

```python
from rdkit import Chem, rdBase
from rdkit.Chem import Draw, rdMolDescriptors

candidate = "c1ccccc1O"  # 替换为实际候选；不要删除未解析的 Markush 信息
mol = Chem.MolFromSmiles(candidate)
if mol is None:
    raise ValueError("SMILES 无法解析或 sanitization 失败；回查候选")
print("RDKit", rdBase.rdkitVersion)
print("canonical isomeric SMILES", Chem.MolToSmiles(mol, isomericSmiles=True))
print("formula", rdMolDescriptors.CalcMolFormula(mol))
print("formal charge", Chem.GetFormalCharge(mol))
Draw.MolToFile(mol, "candidate.png")
```

打开重绘图，对照原图检查环系、键级、杂原子、取代位置、电荷和立体信息。解析成功只验证格式和部分化学规则，**不能证明识别正确**。对盐、互变异构体或立体异构体的比较应先明确策略；不自动改变输入。不确定的连接、R 基、金属配位或图中缺失的立体信息不能用此脚本“补全”。

## 4. PubChem 与 ChemLint

- [PubChem MCP 示例](https://github.com/PhelanShao/pubchem-mcp-server) 和 [PubChem PUG REST 文档](https://pubchem.ncbi.nlm.nih.gov/docs/pug-rest)：按名称、CID 或明确结构查询公开数据。MCP 需客户端事先配置且可联网；查询后核对 CID 对应的结构、立体化学和盐形式。避免向外部服务发送不应公开的结构。
- [ChemLint](https://github.com/molML/ChemLint)：MCP 工具适合分子数据标准化、去重、描述符和数据质量检查。采用前确认服务和客户端已连接；它不负责从图片提取结构，不能替代原图核对。
- [Google Cheminformatics Skills](https://deepmind.google/blog/agentic-cheminformatics-with-google-antigravity/) 面向 Antigravity 的化学工作台；不要假设其工具或技能已安装在本仓库或其他客户端。

## 5. 可选的 ECNU-ChemGPT 下游分析

[ECNU-ChemGPT](https://github.com/ilog-ecnu/ECNU-ChemGPT) 提供化学问答和逆合成相关代码；其 Retro 示例使用带反应类别标记的 SMILES，不能取代图片识别。仅在用户需要化学问答或路线讨论、且已配置相应模型时使用。输入先经原图及 SI 核对；保存模型版本、输入、类别标记和候选输出。逆合成预测是待检验假设，不是已验证条件或论文实际路线。未明确类别、权重、接口或运行环境时报告缺失，不猜测输入格式或声称已运行。QA 72B 需评估硬件；本 skill 不包含模型权重。仓库示例路径和 GPU 设置需要适配。

## 6. 交付给论文精读

对每个化合物交付：编号和图/页码；原图与候选结构；变量定义；已识别、解析、原图/SI 核对三种状态；工具与版本；仍存疑的原子/键；相对其他化合物的结构差异。论文精读再核对作者提出的设计、实测和计算，分别标注出处。无法确认的结构不得用于确定性结构–性质结论。

## 验收示例

1. **普通分子**：图中明确、已提供 SMILES；解析与重绘后逐项检查。解析成功但重绘位置不符时标为存疑。
2. **位置异构体**：两个候选分子式相同，必须比对连接位点，不以分子式相同认定相同。
3. **Markush 图**：保留 R1 定义及 E-SMILES 扩展；未知定义时仅报告骨架和未定位置。
4. **模糊楔形键**：标记立体化学未确认，不凭软件候选推定 R/S。
5. **依赖缺失**：报告未实际运行哪些工具，并基于原图提供可见结构与明确的不确定项。

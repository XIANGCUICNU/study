# 光谱科研、数据处理与办公 Skills

这套 skill 面向光谱测试和通用表格数据，提供 AI 工作流程、上游脚本和参考资料。文件加入仓库后，仍需在支持 skill 的 AI 工具中打开仓库，按需安装 Python 依赖；GitHub 网页自身不会执行 skill。

## 推荐组合

| Skill | 适用任务 | 来源与运行条件 |
| --- | --- | --- |
| [spectroscopy-analysis](../.agents/skills/spectroscopy-analysis/SKILL.md) | 吸收/发射/拉曼/红外光谱的单位、基线、平滑、峰位、峰宽、拟合、积分 | 项目定制；Python 科学计算工具 |
| [exploratory-data-analysis](../.agents/skills/exploratory-data-analysis/SKILL.md) | CSV/TSV 等的缺失值、分布和数据质量探索 | K-Dense；核心表格 CLI 只需 Python 3.11+ 标准库；其他格式按上游能力表 |
| [statistical-analysis](../.agents/skills/statistical-analysis/SKILL.md) | 组间比较、回归、效应量、置信区间和统计报告 | K-Dense；Python 3.12+，按上游说明配置独立统计环境 |
| [research-plotting](../.agents/skills/research-plotting/SKILL.md) | 光谱叠图、拟合残差、误差棒、热图、多面板图；PNG/PDF/SVG | 项目定制；Matplotlib；本地运行 |
| [research-office](../.agents/skills/research-office/SKILL.md) | Excel、Word、PPT、PDF 实验表格与报告 | 项目定制；按文件类型配置依赖；排版检查需要渲染工具 |
| [scientific-schematics](../.agents/skills/scientific-schematics/SKILL.md) | 实验装置、流程、机理等科研示意图草稿 | K-Dense；requests、OpenRouter Key、联网与调用费用；输出 PNG |
| [generate-image](../.agents/skills/generate-image/SKILL.md) | 通用插图、图片编辑、汇报视觉素材 | K-Dense；Python 3.9+ 标准库、OpenRouter Key、联网与调用费用 |

四个上游 skill 连同其目录中的脚本、参考资料和模板完整复制，固定来源版本并保留 [MIT 许可](../.agents/skills/K-DENSE-LICENSE.md)。三个项目 skill 是针对本仓库新写的指令，没有复制其他供应商的专有办公 skill。

## 在 Codex 中使用

合并 PR 后，在本地克隆或更新仓库：

```bash
git clone https://github.com/XIANGCUICNU/study.git
cd study
# 已克隆时，在相应工作副本更新 main。
```

使用支持项目级 `.agents/skills/` 的 Codex 版本打开 `study`，新建会话或重新加载以刷新 skill。选择 skill，或在提示中显式写 `$spectroscopy-analysis` 等名称。若当前客户端不识别该目录，可先让它读取对应 SKILL.md；这是手动使用说明，不代表自动发现已成功。

其他 AI 客户端的发现目录可能不同，应按该客户端文档加载整个 skill 目录，包含 scripts、references 和 assets，不能仅复制 SKILL.md。这里没有改动你本机的全局配置。

## 本地依赖

推荐 Python 3.12。光谱、绘图和办公任务可以建立独立环境：

```bash
python -m venv .venv-research
# macOS / Linux
source .venv-research/bin/activate
# Windows PowerShell 则使用：.venv-research\Scripts\Activate.ps1
python -m pip install -r env/requirements-research-skills.txt
```

依赖清单是起点，不是锁文件；完成实际分析后记录/锁定解析到的软件版本。上游 statistical-analysis 的特定库接口和版本在其 SKILL.md 中说明，使用其代码示例时优先按该说明建立独立环境，不将不同科学计算版本强行混装。贝叶斯库、Jupyter、仪器专有格式解析和 OCR 只在需要时配置。

Office 文件的公式重算和版面渲染可能需要 LibreOffice 等额外系统工具。Python 包生成 .docx/.pptx 不意味着已验证分页或字体，openpyxl 不会重算公式。

## 典型提示词

```text
$spectroscopy-analysis 读取 data/raw/ 中的发射光谱 CSV，横轴 nm，纵轴计数。
先检查背景和采样，再拟合指定区间的峰，比较 Gaussian 与 Voigt。
保留原始数据，输出峰参数、拟合和残差图、可复现脚本，列出全部预处理参数。
```

```text
$exploratory-data-analysis 检查这份实验表格的缺失值、重复记录和数据分布。
$statistical-analysis 比较两个独立样品组，区分技术重复与独立实验，
报告效应量、95% 置信区间和假设诊断，不只给 p 值。
```

```text
$research-plotting 将三组真实光谱画成论文用叠图，注明是否归一化，
导出 SVG、PDF、300 dpi PNG，并检查图例和标签。
$research-office 将已经验证的结果做成可编辑 Word 报告和组会 PPT。
```

## 独立命令行工具

从仓库根目录运行表格概览（data/raw 必须已存在）：

```bash
python .agents/skills/exploratory-data-analysis/scripts/tabular_profile.py \
  experiment.csv --root data/raw --reveal-identifiers
```

输入路径相对于 `--root`；CLI 的输出默认是汇总，不打印原始行。该工具只是数据概览，不进行峰拟合，也不自动替你清洗数据。

图片脚本使用 `OPENROUTER_API_KEY` 环境变量。不要把真实 Key 写入仓库或放在命令参数中。以下命令需要你的账户配置，会触发付费生成；本次添加 skill 没有执行它们：

```bash
mkdir -p results/figures
python .agents/skills/scientific-schematics/scripts/generate_schematic.py \
  "光致发光测试装置示意图：激光源、样品、收集透镜、单色仪、探测器；明确光路箭头" \
  -o results/figures/pl_setup.png --doc-type presentation

python .agents/skills/generate-image/scripts/generate_image.py \
  "用于光物理组会封面的抽象光谱插图，无坐标和实验数据" \
  -o results/figures/cover.png
```

科研示意图是草稿，需要逐一核对装置、箭头、文字和物理关系。生成图不能作为实验观测；真实曲线由 research-plotting 从实际数据绘制。上游示意图脚本输出栅格 PNG，转换成 PDF 不会使其成为矢量图。

如所在客户端有原生图片生成能力，也可以让它使用该能力；是否可用取决于客户端，仓库中的这两个上游脚本仍按 OpenRouter 工作。模型名称、接口和价格以执行时的服务端信息为准。

## 来源与验证

上游：[K-Dense-AI/scientific-agent-skills](https://github.com/K-Dense-AI/scientific-agent-skills)，固定提交 `a1043ef544e3a100c28e70a736b6cf32554d0677`，获取日期 2026-10-14。逐文件 Git blob 哈希及本地路径见 [sources manifest](research-skills-sources.json)。上游目录未修改，因此其中关于其他未安装 skill 的建议是可选参考，不保证那些 skill 在本仓库存在。

本次验证包括上游文件完整性、七个 skill 的发现结构、Python 脚本语法、表格/图片 CLI 帮助入口和模拟 CSV 的汇总结果。尚未验证完整统计依赖环境、实际仪器文件、Office 渲染或外部付费图片 API；真实任务仍需执行并核查结果。

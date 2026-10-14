# study

科研仓库 — 光物理与光谱数据处理、拟合与计算任务自动化

## 简介
本仓库用于存放光物理相关的研究代码与数据处理脚本，主要包括：
- 光谱数据预处理与拟合（吸收/发射/拉曼等谱线）
- 自动化脚本：批量提交计算任务（如 HPC/SLURM 作业）
- 数据分析与绘图脚本
- 实验/计算结果管理与示例数据

本仓库旨在组织、复现和共享常用的分析流程，便于长期维护与协作。

## AI 科研 Skills

已整理光谱分析、通用数据探索与统计、科研绘图、办公文件和图片生成共七个 skill，位于 `.agents/skills/`。查看 [安装与使用说明](docs/research-skills.md)；其中四个来自 K-Dense 的 MIT 许可 GitHub 仓库，三个为本项目定制。Skill 文件用于指导 AI 工作，不等于已安装 Python 依赖或自动完成分析。

## 目录（示例）
- `data/` — 原始与示例数据（通常不直接提交大型原始文件）
- `notebooks/` — Jupyter 笔记本，用于交互式分析与可视化
- `scripts/` — 可执行脚本（谱线拟合、批量处理、job 提交）
- `src/` — 可复用的 Python 模块（拟合模型、工具函数）
- `results/` — 输出结果（拟合参数、图像）
- `env/` — 环境配置示例（requirements.txt 或 conda 环境 yml）
- `docs/` — 说明文档与使用手册

## 快速开始

1. 克隆仓库
   git clone https://github.com/XIANGCUICNU/study.git

2. 建议创建虚拟环境（示例：conda）
   conda create -n study-env python=3.10
   conda activate study-env
   pip install -r requirements.txt

3. 运行光谱拟合脚本（示例）
   python scripts/fit_spectrum.py --input data/example_spectrum.csv --output results/example_fit.json --plot results/example_fit.png

（脚本选项以各自脚本的 `--help` 输出为准）

## 数据格式与命名约定（建议）
- 原始数据放在 `data/raw/`，处理后数据放在 `data/processed/`
- 文件名示例：sample1_YYYYMMDD_instrument.csv
- CSV 列建议：wavelength (nm), intensity (a.u.), [error]
- 结果保存在 `results/<sample>/`，包含拟合参数（JSON/CSV）与绘图（PNG/PDF）

## 谱线拟合（说明）
- 拟合函数与模型放在 `src/models.py`
- 常用拟合流程：
  1. 读取并预处理（去背景、平滑）
  2. 根据物理意义选择拟合模型（高斯、洛伦兹、Voigt、线性背景等）
  3. 初始参数估计（峰值、宽度）
  4. 使用最小二乘或贝叶斯方法拟合（scipy.optimize、lmfit、emcee 等）
  5. 结果保存与可视化

示例（命令行）：
python scripts/fit_spectrum.py --input data/processed/sample1.csv --model voigt --save results/sample1/

## 批量作业提交（HPC / SLURM 示例）
在 `scripts/` 中放置用于生成并提交作业的脚本，例如 `submit_jobs.sh` 或 `submit_job.py`。下面是一个基本 SLURM 作业脚本模板，供参考：

```bash
#!/bin/bash
#SBATCH --job-name=spec_fit
#SBATCH --output=logs/spec_fit_%j.out
#SBATCH --error=logs/spec_fit_%j.err
#SBATCH --time=02:00:00
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=8G

module load anaconda
source activate study-env

python scripts/fit_spectrum.py --input data/processed/sample1.csv --output results/sample1/
```

批量提交示例（简单循环）：
```bash
for file in data/processed/*.csv; do
  sbatch --export=INPUT="$file" scripts/slurm_template.sh
done
```

## 自动化建议
- 将样本列表、参数模板放在 `config/` 中（YAML/JSON），脚本读取配置以实现无交互批处理。
- 用 `logs/` 记录每次运行的元数据（时间、命令、随机种子、软件版本）。
- 若使用 Git 管理代码与小数据集，避免将大数据集直接推到仓库（使用 LFS 或外部数据存储）。

## 开发与贡献
- 代码风格：遵循 PEP8，使用黑（black）和 flake8 做格式化/静态检查
- 新增脚本请在 `scripts/` 下添加可复现的命令行接口（argparse）
- 提交代码前运行单元测试（推荐使用 pytest）
- 提交格式：feature 分支 -> Pull Request -> 至少一个 reviewer -> 合并

## 环境依赖（示例）
在 `requirements.txt` 中列出主要依赖，例如：
- numpy
- scipy
- pandas
- matplotlib
- lmfit
- jupyter
- emcee (可选, 若使用贝叶斯拟合)

或者使用 conda 环境文件 `env/environment.yml`。

## 许可证与引用
建议使用 MIT 许可证（或根据你的机构要求选择合适的许可证）。如果你在论文或报告中使用本仓库的代码/数据，请在引用中注明作者与仓库链接。

## 联系方式
仓库维护者：XIANGCUICNU  
GitHub: https://github.com/XIANGCUICNU

---

如果你希望，我可以：
- 把 README.md 直接提交到 `XIANGCUICNU/study` 仓库（需要确认并授权我进行写操作），
- 或根据你的具体脚本/目录结构把 README 中的示例命令改成与你当前代码一致的版本，
- 也可以帮你生成 `requirements.txt`、SLURM 模板脚本或 `fit_spectrum.py` 的示例实现。


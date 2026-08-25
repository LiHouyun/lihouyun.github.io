---
title: Python 项目到底该用 venv 还是 Poetry？
data: 2026-08-25 11:10:00 +0800
category: 环境配置
tag: [poetry, venv, python]
description: Python 项目到底该用 venv 还是 Poetry？
---

每次新开一个Python项目，我都会花5分钟纠结同一个问题：**这次用Poetry，还是老老实实用venv？**

这个问题没有标准答案，但根据我踩过的坑，有**更适合**的答案。

## 1. 先说结论

| 情况 | 选 |
| ----| ----|
| 写个几百行的小脚本、做一次性的数据分析 | **venv** |
| 写一个要发布到PyPI的库 | **Poetry** |
| 团队协作（≥3人）的Web项目 | **Poetry** |
| 运维脚本、CI/CD流水线里的简单任务 | **venv** |
| 刚学Python两周，还没搞懂`__init__.py`是啥 | **venv**（先别增加认知负担） |
| 项目预期维护超过6个月 | **Poetry** |

**短平快的用venv，长生命周期的用Poetry。**

## 2. venv：简单可靠

venv是Python 3.3起自带的模块，不需要安装任何东西。

### 2.1 工作方式

```bash
# 创建环境
python -m venv .venv

# 激活（Linux/macOS）
source .venv/bin/activate

# 安装依赖
pip install flask requests

# 导出依赖
pip freeze > requirements.txt
```

### 2.2 优点

- **零依赖**：Python自带，不需要额外安装Poetry、pipx等任何工具
- **透明**：虚拟环境就是一个文件夹，想删就删，想挪就挪
- **学习曲线平缓**：新手5分钟就能掌握
- **CI/CD友好**：在Docker或GitHub Actions里，用venv的几行命令简单直接，不用考虑额外工具的缓存策略

### 2.3 缺点（而且是不小的缺点）

**依赖管理靠手动。** 这是所有问题的根源。

- `requirements.txt` 只记录顶层依赖的版本，不记录子依赖的版本。一个月后重新安装，可能得到一套完全不同的子依赖版本，项目莫名其妙就挂了。
- 解决依赖冲突全靠人工。`pip install` 报错说A需要B<2.0，C需要B>=2.0，得自己翻文档找兼容版本。
- 无法区分生产依赖和开发依赖（虽然可以用多个requirements文件凑合，但很别扭）。
- 卸载依赖不干净。`pip uninstall` 不会自动卸载子依赖，环境会越来越臃肿。

## 3. Poetry：工程化的选择

Poetry需要单独安装（推荐用`pipx install poetry`），它是当前Python社区最主流的现代化项目管理工具。

### 3.1 工作方式

```bash
# 初始化新项目
poetry new my-project
# 或在现有项目中初始化
poetry init

# 添加依赖（自动更新锁文件）
poetry add flask

# 添加开发依赖
poetry add pytest --group dev

# 安装所有依赖（自动创建虚拟环境）
poetry install

# 运行脚本（无需手动激活环境）
poetry run python main.py
```

### 3.2 优点

- **依赖解析靠谱**：Poetry内置了依赖解析器，添加包时会自动计算所有子依赖的兼容版本。版本冲突会当场报错，不会让运行到一半才发现问题。
- **锁文件保平安**：`poetry.lock` 记录了所有依赖的精确版本和哈希值，团队每个人、生产环境、半年后的你，安装到的依赖完全一致。这比`requirements.txt`靠谱一个量级。
- **依赖分组天然支持**：`--group dev` 区分开发工具和运行时依赖，部署时只装生产依赖，镜像体积更小。
- **打包发布一条龙**：`poetry build` 和 `poetry publish` 直接搞定，不用再学setuptools那套复杂配置。
- **`pyproject.toml` 是未来**：这是PEP 518/621规定的标准格式，Poetry用的就是这个，配置集中、可读性强。

### 3.3 缺点

- **额外学习成本**：要记住`add`、`install`、`update`、`remove`等一套新命令，理解"lock file"的概念也需要点时间。
- **依赖解析慢**：当项目依赖了上百个包时，`poetry add` 或 `poetry update` 可能会花上30秒甚至几分钟去做版本计算。venv没有这个"烦恼"，因为它根本不解析。
- **版本锁定的双刃剑**：锁文件保证了一致性，但也意味着**需要主动**执行`poetry update`才能获得依赖的安全更新。很多人会忘记这件事，导致生产环境长期运行着有漏洞的旧版本。
- **多了一层抽象**：出问题时，得搞清楚是Poetry的问题、PyPI源的问题、还是Python本身的问题。

## 4. 两个容易被忽略的实际问题

### 4.1 1. 依赖更新策略

这是很多人没意识到的关键差异。

- **venv**：因为没有锁文件，每次`pip install`天然就是"最新兼容版"。但缺点是**不稳定**，可能引入breaking change。
- **Poetry**：锁文件让环境**极度稳定**，但需要专门安排时间执行`poetry update`，否则依赖永远不会升级，**安全漏洞也永远不会自动修复**。

**推荐做法**：用Poetry的项目，设置Dependabot或Renovate来自动提依赖更新PR，CI跑通就合并。这样既有锁定的稳定性，又有自动化更新。

### 4.2 2. CI/CD里的体验

在GitHub Actions里：

- **venv**：直接`pip install -r requirements.txt`，直接就能缓存设置。
- **Poetry**：需要额外安装Poetry本身，配置缓存时需要把`~/.cache/pypoetry`也缓存起来。配置稍微复杂一些，但一旦配好，依赖安装速度其实比pip更快（因为锁文件让Poetry可以跳过版本解析步骤）。


**目前的选择标准**：

- 个人实验、一次性脚本、临时工具 → `venv`
- 公司项目、开源库、任何预期生命周期超过3个月的项目 → `Poetry`
- 给客户交付的DEMO → `venv`（因为客户那边的环境可能没装Poetry）
- 需要发布到PyPI的库 → `Poetry`（`pyproject.toml`是目前社区公认的标准写法）

如果你的项目还在纠结，不妨先用venv跑起来，等依赖数量超过10个、或者开始有第二个协作者时，用`poetry init`迁移到Poetry也只需要10分钟。不用在一开始就把自己框死。

---
title: conda 转为 venv
date: 2025-09-20 18:30:00 +0800
category: 软件
tags: [Python, 包管理, 环境配置]
description: 从 Conda 迁移到 venv 并使用 PyInstaller 打包的核心流程
---


## 1. 从已有环境导出依赖列表
在需要“复制”环境的源项目（如 `eeg-hand`）中，激活其环境（Conda 或 `venv` 均可），执行以下命令，将当前环境的所有 Python 包及版本导出到 `requirements.txt` 文件：
```bash
pip freeze > requirements.txt
```
该文件会记录项目运行所需的所有依赖（如 `bleak`、`winrt`、`pyinstaller` 等）。

## 2. 为目标项目创建 `venv` 环境
进入目标项目（如 `eeg-typing`）的根目录，执行以下命令，创建一个新的 `venv` 环境（环境名称可自定义，这里以 `venv_project` 为例）：
```bash
python -m venv venv_project
```
执行后，项目根目录会生成一个名为 `venv_project` 的文件夹，其中包含独立的 Python 解释器和依赖存储目录。

## 3. 激活 `venv` 环境
激活命令因操作系统而异：
- **Windows（CMD 终端）**：
  ```cmd
  venv_project\Scripts\activate.bat
  ```
- **Windows（PowerShell 终端）**：
  ```powershell
  .\venv_project\Scripts\Activate.ps1
  ```
  若提示执行策略问题，先执行 `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` 并按提示输入 `Y`，再重新激活。
- **Linux/macOS**：
  ```bash
  source venv_project/bin/activate
  ```
激活成功后，终端前缀会显示 `(venv_project)`，表示当前处于新环境中。

## 4. 安装项目依赖
将步骤 1 中生成的 `requirements.txt` 复制到目标项目根目录，执行以下命令，安装所有依赖：
```bash
## 先升级 pip 到最新版本，避免安装过程中出现问题
python -m pip install --upgrade pip

## 通过 requirements.txt 安装所有依赖
pip install -r requirements.txt
```
这一步会确保目标项目的 `venv` 环境与源项目有完全一致的依赖包及版本。

## 5. 使用 `PyInstaller` 打包项目
在激活的 `venv` 环境中，执行 `PyInstaller` 打包命令。以常见的单文件、带控制台、指定名称的打包为例：
```bash
pyinstaller -F -c app.py --name "项目名称"
```
- `-F`：表示生成单个独立的可执行文件，方便分发。
- `-c`：表示显示控制台窗口，便于查看程序运行时的输出和报错信息（若为 GUI 程序，后续可改为 `-w` 隐藏控制台）。
- `--name "项目名称"`：用于指定生成的可执行文件的名称。

打包完成后，可执行文件会生成在 `dist` 文件夹中，直接运行该文件即可测试项目功能。

## 6. 若有一些资源文件（如图片）
### 6.1 在代码中添加资源路径处理函数（解决图片加载问题）
在项目代码（如 `app.py` 或工具类文件）中添加：
```python
import sys
import os

def get_resource_path(relative_path):
    """获取资源文件路径（兼容开发和打包环境）"""
    if getattr(sys, 'frozen', False):
        ## 打包后环境：资源在临时目录 _MEIPASS 中
        base_path = sys._MEIPASS
    else:
        ## 开发环境：资源在当前脚本所在目录
        base_path = os.path.dirname(os.path.abspath(__file__))
    return os.path.join(base_path, relative_path)
```

### 6.2 使用资源路径函数加载图片
修改代码中加载图片的逻辑：
```python
## 示例：加载 res 文件夹下的图片
image_path = get_resource_path("res/右手张开.png")

## 后续用 image_path 加载图片（以 PyQt 为例）
from PyQt5.QtGui import QPixmap
pixmap = QPixmap(image_path)
```


### 6.3 执行 `pyinstaller` 打包命令（包含资源文件）
```bash
pyinstaller -F -c app.py --name "气动手控制系统" --add-data "res/*;res"
```
- `--add-data "res/*;res"`：将项目根目录的 `res` 文件夹（含所有图片）打包到程序中，保持 `res` 文件夹结构。


### 7.4 测试打包结果
删除旧的 `dist` 和 `build` 文件夹，重新打包后，进入 `dist` 目录运行 `气动手控制系统.exe`，验证功能和图片加载是否正常。

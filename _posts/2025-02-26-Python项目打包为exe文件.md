---
title: Python 项目打包为 exe 文件
date: 2025-02-26 16:10:00 +0800
---

## 前言

项目写完之后需要打包、分发、安装。

对于 Python 项目，了解到有 PyInstaller，cx_Freeze，py2exe。PyInstaller 简单尝试后，生成的 exe 文件双击后闪退，便没有深究。后尝试使用 cx_Freeze，很顺利。下面介绍 cx_Freeze 打包为 exe 文件的步骤。

## 使用 cx_Freeze 打包项目(好用)
以下是使用 cx_Freeze 打包 Python 项目的步骤：

1. 安装 cx_Freeze
在命令行中运行以下命令来安装 cx_Freeze：

```sh
pip install cx_Freeze
``` 
2. 创建 setup.py 文件
在你的项目目录中创建一个 setup.py 文件，内容如下：

```python
import sys
from cx_Freeze import setup, Executable

# 要打包的Python脚本路径
script = "app.py"

# 不在运行程序的时候出现cmd后台框
base = None
if sys.platform == "win32":
    base = "Win32GUI"

# 创建可执行文件的配置
exe = Executable(
    script=script,
    base=base,
    target_name="MyProgram", # 自定义
    icon='./src/icon/icon_2_kao.ico' # 自定义
)

# 打包的参数配置
options = {
    "build_exe": {
        "packages": [],
        "excludes": []
    }
}

setup(
    name="MyProgram", # 自定义
    version="1.0", # 自定义
    description="My Program Description", # 自定义
    options=options,
    executables=[exe]
)
```
3. 打包项目
在命令行中运行以下命令来打包项目，这步会有很多输出，耐心等待。

```sh
python setup.py build
```

4. 查看生成的 .exe 文件
打包完成后，会在项目目录下生成一个 build 文件夹，.exe 文件就在这个文件夹中，双击 exe 即可运行

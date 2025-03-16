---
title: 为 Markdown 添加序号的 Python 程序
data: 2025-03-15 00:42:00 +0800
category: 软件
tag: [Python, 小工具]
description: 使用 Markdown 记笔记时发现偶尔会需要调换顺序，那么手动编号就要多处修改，很不方便，于是有了这个程序。
---

## 1. 前言
使用 Markdown 记笔记时发现偶尔会需要调换顺序，那么手动编号就要多处修改，很不方便，于是有了这个程序。

## 2. 程序特点
1. 文件导入，文件导出。从哪里导入，导出到哪里。
2. 根据标题的相对级别生成序号，而不是单纯的通过“#”的数量盘底那个级别。
3. 使用 `argparse` 模块解析命令行参数，不必修改 Python 文件使用。
4. 使用 `argparse` 自动生成帮助信息，并在用户输入 `-h` 或 `--help` 时显示。

## 3. Code

```python
"""
3-为markdown添加序号.py

功能：
    该程序用于为 Markdown 文件中的标题添加序号。支持多级标题（如 #, ##, ### 等），
    并根据标题的相对级别动态生成序号。输出文件会在输入文件名的后面加上 "-output" 后缀。

用法：
    在命令行中运行程序，并通过 -input 参数指定输入的 Markdown 文件路径。例如：
    - python 3-为markdown添加序号.py -input input.md
    - python 3-为markdown添加序号.py -input ../input.md
    - python 3-为markdown添加序号.py -input f:/input.md

    程序会在导入文件的同级位置生成一个带有 "-output" 后缀的 Markdown 文件，例如 input-output.md。

    查看帮助信息：
    - python 3-为markdown添加序号.py -h
    - python 3-为markdown添加序号.py --help

作者：lihouyun
时间：Mar 2025
版本：1.0
"""

import re
import os
import argparse

def add_markdown_numbers(text):
    # 按行分割文本
    lines = text.split('\n')
    
    # 找到最小标题级别
    min_level = float('inf')
    for line in lines:
        match = re.match(r'(#+)\s*(.*)', line)
        if match:
            level = len(match.group(1))
            if level < min_level:
                min_level = level
    
    # 如果没有标题，直接返回原文
    if min_level == float('inf'):
        return text
    
    # 初始化计数器
    counters = [0] * 6  # 支持最多6级标题
    
    # 处理每一行
    for i in range(len(lines)):
        line = lines[i]
        # 检查是否为标题行
        match = re.match(r'(#+)\s*(.*)', line)
        if match:
            level = len(match.group(1))  # 标题级别
            title = match.group(2).strip()  # 标题内容
            
            # 计算相对级别
            relative_level = level - min_level + 1
            
            # 更新计数器
            counters[relative_level-1] += 1
            for l in range(relative_level, len(counters)):
                counters[l] = 0
            
            # 生成编号
            if relative_level == 1:
                # 一级标题：编号后加一个“.”
                number = f"{counters[0]}."
            else:
                # 二级及以下标题：编号后不加“.”
                number = '.'.join(str(counters[l]) for l in range(relative_level) if counters[l] != 0)
            
            # 替换标题行
            lines[i] = f"{'#' * level} {number} {title}"
    
    # 重新组合文本
    return '\n'.join(lines)

def process_markdown_file(input_file):
    # 读取输入文件
    with open(input_file, 'r', encoding='utf-8') as file:
        content = file.read()
    
    # 处理内容
    processed_content = add_markdown_numbers(content)
    
    # 生成输出文件名
    base_name, ext = os.path.splitext(input_file)
    output_file = f"{base_name}-output{ext}"
    
    # 写入输出文件
    with open(output_file, 'w', encoding='utf-8') as file:
        file.write(processed_content)
    
    print(f"处理完成！结果已保存到 {output_file}")

def main():
    # 设置命令行参数解析
    parser = argparse.ArgumentParser(
        description="为 Markdown 文件添加序号。输入文件通过 -input 参数指定，输出文件会在输入文件名后加上 '-output' 后缀。",
        epilog="示例：\n"
               "  python 3-为markdown添加序号.py -input input.md\n"
               "  python 3-为markdown添加序号.py -input ../input.md\n"
               "  python 3-为markdown添加序号.py -input f:/input.md",
        formatter_class=argparse.RawTextHelpFormatter  # 保留格式
    )
    parser.add_argument(
        '-input', 
        required=True, 
        help="输入的 Markdown 文件路径（支持相对路径和绝对路径）"
    )
    
    # 解析命令行参数
    args = parser.parse_args()
    
    # 处理文件
    process_markdown_file(args.input)
    
if __name__ == '__main__':
    main()

```

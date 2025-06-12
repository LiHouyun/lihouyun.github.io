---
title: Excel 中定位 NaN 值
data: 2025-06-12 23:40:00 +0800
category: 软件
tag: [Python, 小工具]
description: Excel 中定位 NaN 值
---

# 1. 前言
最近使用 SVM 的过程中出现了“NaN”的报错，但数据量太大，于是写了这个小工具定位问题数据。

# 2. Code
## 2.1 报错
```sh
(time_series_classification) F:\Project\YSNeuro\svm_test\svm_test_1>python train.py
正在训练 SVM 模型...
Traceback (most recent call last):
  File "F:\Project\YSNeuro\svm_test\svm_test_1\train.py", line 193, in <module>
    classifier.run('./data/real_data_sheet_cls_5_artificial_cut_fill0_random01.xlsx')
  File "F:\Project\YSNeuro\svm_test\svm_test_1\train.py", line 157, in run
    self.train(train_df)
  File "F:\Project\YSNeuro\svm_test\svm_test_1\train.py", line 83, in train
    self.model.fit(X_train, y_train)
  File "F:\anaconda3\anaconda3\envs\time_series_classification\lib\site-packages\sklearn\base.py", line 1389, in wrapper
    return fit_method(estimator, *args, **kwargs)
  File "F:\anaconda3\anaconda3\envs\time_series_classification\lib\site-packages\sklearn\svm\_base.py", line 197, in fit
    X, y = validate_data(
  File "F:\anaconda3\anaconda3\envs\time_series_classification\lib\site-packages\sklearn\utils\validation.py", line 2961, in validate_data
    X, y = check_X_y(X, y, **check_params)
  File "F:\anaconda3\anaconda3\envs\time_series_classification\lib\site-packages\sklearn\utils\validation.py", line 1370, in check_X_y
    X = check_array(
  File "F:\anaconda3\anaconda3\envs\time_series_classification\lib\site-packages\sklearn\utils\validation.py", line 1107, in check_array
    _assert_all_finite(
  File "F:\anaconda3\anaconda3\envs\time_series_classification\lib\site-packages\sklearn\utils\validation.py", line 120, in _assert_all_finite
    _assert_all_finite_element_wise(
  File "F:\anaconda3\anaconda3\envs\time_series_classification\lib\site-packages\sklearn\utils\validation.py", line 169, in _assert_all_finite_element_wise
    raise ValueError(msg_err)
ValueError: Input X contains NaN.
SVC does not accept missing values encoded as NaN natively. For supervised learning, you might want to consider sklearn.ensemble.HistGradientBoostingClassifier and Regressor which accept missing values encoded as NaNs natively. Alternatively, it is possible to preprocess the data, for instance by using an imputer transformer in a pipeline or drop samples with missing values. See https://scikit-learn.org/stable/modules/impute.html You can find a list of all estimators that handle NaN values at the following page: https://scikit-learn.org/stable/modules/impute.html#estimators-that-handle-nan-values
```

## 2.2 源码
```python
import pandas as pd
import sys
import os

def right_align_data(excel_path, sheet_name, align_position, fill_value):
    """
    将Excel表格中的数据右对齐到指定位置，并在左侧填充指定值
    
    参数:
    excel_path (str): Excel文件路径
    sheet_name (str): 要处理的sheet名称
    align_position (int): 右对齐的目标位置
    fill_value: 要填充的值
    
    返回:
    pd.DataFrame: 处理后的数据
    """
    # 读取Excel文件，指定header=None表示没有标题行
    df = pd.read_excel(excel_path, sheet_name=sheet_name, header=None)
    
    # 计算最大列数
    max_cols = align_position
    
    # 创建新的DataFrame，预先分配所有列
    new_data = []
    
    # 处理每一行数据
    for _, row in df.iterrows():
        # 获取当前行的非空值
        valid_values = [v for v in row.values if pd.notna(v)]
        
        # 计算需要填充的数量
        fill_count = max(0, align_position - len(valid_values))
        
        # 创建新的数据列表：先填充指定值，再添加原始数据
        new_row = [fill_value] * fill_count + valid_values
        
        # 确保行长度一致
        if len(new_row) < max_cols:
            new_row += [fill_value] * (max_cols - len(new_row))
        
        new_data.append(new_row[:max_cols])
    
    # 创建新的DataFrame
    result_df = pd.DataFrame(new_data)
    
    return result_df

def main():
    """主函数，处理命令行参数并执行右对齐操作"""
    # 检查命令行参数
    if len(sys.argv) != 5:
        print("用法: python excel_align.py <excel_path> <sheet_name> <align_position> <fill_value>")
        print("示例: python excel_align.py data.xlsx Sheet1 12 0")
        sys.exit(1)
    
    # 获取命令行参数
    excel_path = sys.argv[1]
    sheet_name = sys.argv[2]
    
    try:
        align_position = int(sys.argv[3])
    except ValueError:
        print("错误: 右对齐位置必须是整数")
        sys.exit(1)
    
    # 尝试将填充值转换为适当的类型
    try:
        fill_value = int(sys.argv[4])
    except ValueError:
        try:
            fill_value = float(sys.argv[4])
        except ValueError:
            fill_value = sys.argv[4]
    
    # 检查文件是否存在
    if not os.path.exists(excel_path):
        print(f"错误: 文件 '{excel_path}' 不存在")
        sys.exit(1)
    
    # 执行右对齐操作
    try:
        result_df = right_align_data(excel_path, sheet_name, align_position, fill_value)
        
        # 构建输出文件名
        base_name, ext = os.path.splitext(excel_path)
        output_path = f"{base_name}_aligned{ext}"
        
        # 保存结果，指定header=False表示不写入列标题
        result_df.to_excel(output_path, index=False, header=False)
        print(f"处理完成! 结果已保存到: {output_path}")
        
    except Exception as e:
        print(f"处理过程中发生错误: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()

```

## 2.3 结果
```sh
(time_series_classification) F:\Project\YSNeuro\svm_test\svm_test_1>python tools\check_nan_in_excel.py

❗ 工作表 '3' 存在缺失值：
          1         2         3          4    ...  597  598  599  600
35 -15.812798 -8.514636 -3.373178 -14.932694  ...  NaN  NaN  NaN  NaN

[1 rows x 600 columns]

❗ 工作表 '4' 存在缺失值：
           1         2        3         4         5    ...  596  597  598  599  600
152 -10.049347  1.082593  1.00075  1.892554  4.177223  ...  NaN  NaN  NaN  NaN  NaN

[1 rows x 600 columns]
```

好的，进一步精简，聚焦于操作代码行，并增加您提到的行的选择、按列切分与匹配，以及 `apply` `lambda` 的用法。

---

### Pandas 上机考试核心操作速查 (精简版)

**1. 基础导入**

```python
import os
import pandas as pd
import numpy as np
```

---

**2. 多文件批量处理 (统一使用 `df.append()` 方式)**

```python
path = 'E:\\1. Python\\00 Python总复习\\data' # 替换为你的数据文件夹路径
zdf = pd.DataFrame() 

for file_name in os.listdir(path):
    full_path = os.path.join(path, file_name)
    if file_name.endswith('.xlsx'):
        subdf = pd.read_excel(full_path, sheet_name=0)
    elif file_name.endswith('.csv'):
        subdf = pd.read_csv(full_path)
    elif file_name.endswith('.txt'):
        subdf = pd.read_table(full_path)
    else:
        continue
    zdf = zdf.append(subdf, ignore_index=True)
```

---

**3. 列的选择 (提取特定列)**

```python
series_col = df['列名A']
df_selected_by_name = df[['列名A', '列名B']]
df_selected_by_pos = df.iloc[:, [0, 2, 5]]
```

---

**4. 行的选择 (提取特定行)**

```python
# 基于条件筛选行
df_filtered_rows = df[df['数值列'] > 100]
df_multi_cond_rows = df[(df['类别'] == 'A') & (df['数量'] < 50)]
df_isin_rows = df[df['城市'].isin(['北京', '上海'])]

# 基于行索引位置选择行
df_first_row = df.iloc[0] # 返回 Series
df_first_five_rows = df.iloc[0:5]
df_specific_rows = df.iloc[[0, 2, 4]]

# 基于行标签选择行 (如果索引不是默认数字而是有意义的标签)
df_single_label_row = df.loc['行标签A']
df_label_range_rows = df.loc['起始标签':'结束标签']
```

---

**5. 字符串操作：切片与模式匹配 (类正则)**

```python
# 字符串切片
df['新列_前三位'] = df['字符串列'].str.slice(0, 3)

# 字符串匹配：是否包含 (可接受正则)
df_contains_pattern = df[df['文本列'].str.contains('关键词', na=False, regex=True)] # regex=True 启用正则

# 字符串匹配：是否以...开头
df_starts_with = df[df['文本列'].str.startswith('前缀', na=False)]

# 字符串匹配：是否以...结尾
df_ends_with = df[df['文本列'].str.endswith('后缀', na=False)]
```

---

**6. 按列切分 (Split) 与 合并 (`str.split`)**

```python
# 将一列按分隔符拆分成多列
new_cols_df = df['单一信息列'].str.split('|', expand=True) # expand=True 会创建新的DataFrame
new_cols_df.columns = ['新列1', '新列2', '新列3'] # 为新列命名

# (可选) 将拆分后的新列合并回原DataFrame
df = pd.concat([df, new_cols_df], axis=1)

# 如果想拆分后只取某部分 (例如只取拆分后的第二部分)
df['部分信息'] = df['单一信息列'].str.split('-', expand=True)[1]
```

---

**7. 处理缺失值 (`.fillna()`)**

```python
df_filled_const = df.fillna(0)
df['数值列'] = df['数值列'].fillna(df['数值列'].mean())
df_ffill = df.fillna(method='ffill')
df_bfill = df.fillna(method='bfill')
```

---

**8. 设置索引 (`.set_index()`)**

```python
df_indexed = df.set_index('作为索引的列名')
df_indexed_keep = df.set_index('作为索引的列名', drop=False)
df_multi_indexed = df.set_index(['列名A', '列名B'])
```

---

**9. `apply` 与 `lambda` (高级数据转换)**

`apply` 用于对DataFrame的行或列进行函数操作，`lambda` 用于定义简洁的匿名函数。

```python
# 案例一：基于单列的简单转换 (等价于直接向量化操作，但lambda更灵活)
df['转换列'] = df['数值列'].apply(lambda x: x * 2 + 5)

# 案例二：基于多列的行级条件判断或计算 (axis=1 表示按行应用)
df['新类别'] = df.apply(lambda row: '高收入' if row['收入'] > 10000 else '低收入', axis=1)

# 案例三：复杂字符串处理 (结合其他方法)
df['姓名长度'] = df['姓名'].apply(lambda x: len(str(x).strip())) # strip()去除空格

# 案例四：格式化列 (日期、电话等)
# 假设 '电话' 列可能是数字或字符串，需要统一格式
df['格式化电话'] = df['电话'].apply(lambda x: f'{str(x)[:3]}-{str(x)[3:7]}-{str(x)[7:]}' if pd.notna(x) and len(str(x)) == 11 else str(x))
```

---

**10. 合并日期列 (`yyyy`, `mm`, `dd` → `yyyy-mm-dd`)**

```python
df['日期'] = df['yyyy'].astype(str) + '-' + \
             df['mm'].astype(str).str.zfill(2) + '-' + \
             df['dd'].astype(str).str.zfill(2)

df['日期时间'] = pd.to_datetime(df['日期']) # 将字符串转换为日期时间对象
```
链接：[[核心考点]] 

整理思路：
![[Pasted image 20250605210005.png]] 

## 1. 读取数据

```python
import os
import pandas as pd
import re          
```

### 1.1 读取单个文件

```python
# 读取 Excel 文件 (默认读取第一个工作表)
df_excel = pd.read_excel('文件路径.xlsx', sheet_name=0)

# 读取 CSV 文件
df_csv = pd.read_csv('文件路径.csv')

# 读取 TXT 表格文件 (默认制表符分隔，无表头，常见中文编码)
df_txt = pd.read_table('文件路径.txt', header=None, encoding='ANSI')
```

### 1.2 多表读取 (多 Sheet 同表 & 多 Excel 文件)

```python
# 读取 Excel 文件所有工作表 (返回字典)
dfdict = pd.read_excel('文件路径.xlsx', sheet_name=None)

# 多 Excel 文件合并读取 (统一使用 df.append() 方式)
path = 'E:\\你的数据文件夹路径' # **请替换为你的实际数据文件夹路径**
zdf = pd.DataFrame() 

for f in os.listdir(path):
    subdf = pd.read_excel(path+‘//’+f)
    zdf = zdf.append(subdf)
```

---

## 2. 处理数据

假设操作对象为 `df`。

### 2.1 数据筛选 
#### 2.1.1 行的选择

```python
# 基于布尔条件筛选行
df_filtered_rows = df[df['数值列'] > 100]
df_multi_cond_rows = df[(df['类别'] == 'A') & (df['数量'] < 50)]
df_isin_rows = df[df['城市'].isin(['北京', '上海'])]

# 基于整数位置选择行 (.iloc)
df_first_row = df.iloc[0]          # 返回 Series
df_first_five_rows = df.iloc[0:5]  # 返回 DataFrame
df_specific_rows = df.iloc[[0, 2, 4]] # 返回 DataFrame

# 基于行标签选择行 (.loc, 适用于非默认数字索引)
df_single_label_row = df.loc['行标签A']
df_label_range_rows = df.loc['起始标签':'结束标签']
```

#### 2.1.2 列的选择

```python
# 根据列名选择单列 (返回 Series)
series_col = df['列名A']

# 根据列名选择多列 (返回 DataFrame)
df = df[['列名A', '列名B']]

# 根据列的位置选择多列 (返回 DataFrame)
df = df.iloc[:, [0, 2, 5]] # 选择第1、3、6列 (索引0, 2, 5)
```

### 2.3 数据结构调整与排序

#### 2.3.1 更改列名

```python
# 更改所有列名 (按顺序赋值)
df.columns = ['新列名1', '新列名2', '新列名3']

# 针对性更改部分列名
df = df.rename(columns={'旧列名A': '新列名A', '旧列名B': '新列名B'})

# 重新排列列的顺序 (即“交换”列值的位置，通过重新选择列)
df = df[['列名C', '列名A', '列名B']] # 按新顺序选择所有列

# 交换两列的内容 (而非名称或顺序)
df['列A'], df['列B'] = df['列B'], df['列A']
```

#### 2.3.2 设置/重置索引

```python
# 将某一列设置为索引，原列默认会被删除
df_indexed = df.set_index('作为索引的列名')

# 将某一列设置为索引，并保留原列
df_indexed_keep = df.set_index('作为索引的列名', drop=False)

# 将多列设置为复合索引 (多级索引)
df_multi_indexed = df.set_index(['列名A', '列名B'])

# 重置索引 (将索引变回普通列)
df_reset_index = df_indexed.reset_index()
df_reset_index_no_old_index = df_indexed.reset_index(drop=True) # 不保留旧索引列
```

#### 2.3.3 删除行/列

```python
# 根据行索引删除行
df_rows_dropped_by_index = df.drop(index=[0, 2, 4]) # 删除指定索引的行

# 根据条件删除行 (通常通过布尔索引完成)
# 例如，删除所有数量小于10的行
df_filtered_by_condition = df[df['数量'] >= 10] # 等价于 df.drop(df[df['数量'] < 10].index)

# 根据列名删除列
df_cols_dropped_by_name = df.drop(columns=['不想要的列A', '不想要的列B'])
# 或者使用 axis=1
df_cols_dropped_by_name_axis = df.drop(['不想要的列A', '不想要的列B'], axis=1)
```

#### 2.3.4 数据排序

```python
# 按单列升序排序
df_sorted_asc = df.sort_values(by='数值列')

# 按单列降序排序
df_sorted_desc = df.sort_values(by='数值列', ascending=False)
```

### 2.4 数据分列/行

#### 2.4.1 数据分列(split)

```python
# 将一列按分隔符拆分成多列，并直接命名
new_cols_df = df['单一信息列'].str.split('|', expand=True)
new_cols_df.columns = ['新列1', '新列2', '新列3']

# 将拆分后的新列合并回原DataFrame
df = pd.concat([df, new_cols_df], axis=1)

# 拆分后只取某一部分 (例如只取拆分后的第二部分)
df['部分信息'] = df['单一信息列'].str.split('-', expand=True)[1]
```

#### 2.4.2 数据分行(explode)

将单个单元格内包含多个值的条目（如用逗号分隔的字符串）拆分成多行，同时复制其他列的数据。主要通过 `df.explode()` 方法实现。

**前提：** 需要拆分的列必须是列表（list）或类列表（如 Series）。如果原始数据是字符串，需先用 `str.split()` 转换为列表。

```python
# 假设原始 DataFrame 有一列 'Tags' 包含逗号分隔的字符串
# 例如: df = pd.DataFrame({'ID': [1, 2], 'Tags': ['tag1,tag2', 'tag3,tag4,tag5']})

# 步骤1: 将字符串列转换为列表
# 如果 'Tags' 列已经是列表，则跳过此步
df['Tags'] = df['Tags'].str.split(',')

# 步骤2: 使用 explode() 将列表内容拆分成多行
df_exploded = df.explode('Tags')

# df_exploded 将是新的 DataFrame，其中 'Tags' 列的每个列表元素都变为一行
# 并且原始 DataFrame 的其他列数据会相应地复制到新行中
```


### 2.5 使用正则表达式提取数据并分列/计算 (结合 apply/lambda)

```python
# 提取多个数字序列并格式化 (如日期 yyyy-mm-dd)
pat_digits = r'\d+' # 匹配一个或多个数字
df['日期字符串'] = df['原始文本列'].apply(lambda x: f"{re.findall(pat_digits, str(x))[0]}-{re.findall(pat_digits, str(x))[1].zfill(2)}-{re.findall(pat_digits, str(x))[2].zfill(2)}")

# **模糊匹配并筛选 (使用 bool(re.findall(...)) 明确判断)**
pat_fuzzy = r'关键词1|关键词2|关键词3' # **替换为你的模糊匹配模式**

# 筛选不包含任何匹配模式的行 (正常数据)
df_normal = df[df['文本列'].apply(lambda x: not bool(re.findall(pat_fuzzy, str(x))))]

# 筛选包含任何匹配模式的行 (异常数据)
df_abnormal = df[df['文本列'].apply(lambda x: bool(re.findall(pat_fuzzy, str(x))))]
```

### 2.6 数据统计 (聚合)

```python
# 按单列分组计数
df_group_count = df.groupby('分类列')['计数列'].count()

# 按多列分组求和
df_group_sum = df.groupby(['分类列A', '分类列B'])['数值列'].sum()

# 统计列中唯一值的数量和频次
value_counts_series = df['列名'].value_counts()
unique_count = df['列名'].nunique() # 唯一值的数量
```

### 2.7 数据清洗 (重复值、空值、补0)：str/astype
str：访问器，“借你访问字符函数，不能转换”，如果有非str输入就会报错
astype：转换器，将非str转为str

```python
# 删除所有列都重复的行
df = df.drop_duplicates()

# 根据指定列删除重复行
df = df.drop_duplicates(subset='学号')

# 使用固定值填充所有缺失值
df = df.fillna(0)

# 使用某列的平均值填充该列的缺失值
df['数值列'] = df['数值列'].fillna(df['数值列'].mean())

# 使用前一个有效值填充 (Forward Fill)
df_ffill = df.fillna(method='ffill')

# 使用后一个有效值填充 (Backward Fill)
df_bfill = df.fillna(method='bfill')

# zfill 补0
df['yyyy-mm'] = df['yyyy']+'-'+df['mm'].str.zfill(2)
```

### 2.8 行列次序变换

```python
# DataFrame 行列转置
df_transposed = df.T

# 将某一列设置为索引，原列默认会被删除
df_indexed = df.set_index('作为索引的列名')

# 将某一列设置为索引，并保留原列
df_indexed_keep = df.set_index('作为索引的列名', drop=False)

# 多个sheet导入并旋转：转-表名赋值-地域赋值-表格填充-删除
import pandas as pd
df=pd.read_excel('E:\\1. Python\\00 Python总复习\\地区产值（行列旋转）.xlsx',sheet_name=None)
del df['直辖市']
for k,v in df.items():
	v=v.T # 先旋转
	v.columns=v.iloc[0] # 给表名赋值第一行
	v['地域']=k # 给地域赋值，先让表格完整；如果先赋值再转，地域会变成一行
	df[k]=v # 完整表格赋予键值
zdf=pd.concat(df.values())
zdf=zdf.drop('指标')
```

### 2.9 表合并，表拆分

```python
# 加到末尾 类似数据库union（axis=0/1 控制union还是join）
df = pd.concat([df1, df2, df3]) # 常与文件读取sheet_name=None配合使用

# 基于条件拆分 DataFrame（就是上面“列的选择”）
df_part_A = df[df['分类'] == 'A']
df_part_B = df[df['分类'] == 'B']

# 直接做字符连接操作 (例如 yyyy, mm, dd 合并成 yyyy-mm-dd)
df['yyyy-mm-dd'] = df['yyyy']+'-'+df['mm']+'-'+df['dd']

df['yyyy-mm-dd'] = df['yyyy']+'-'+df['mm'].astype(str).str.zfill(2)+'-'+df['dd'].astype(str).str.zfill(2)
```

### 2.10 `apply` 与 `lambda` (通用数据转换)

```python
# 基于单列进行简单转换
df['转换列'] = df['数值列'].apply(lambda x: x * 2 + 5)

# 基于多列进行行级判断或计算 (axis=1 表示按行应用)
df['新类别'] = df.apply(lambda row: '通过' if row['分数'] >= 60 and row['出勤'] == '全勤' else '未通过', axis=1)

# 结合字符串方法进行复杂处理
df['首字母'] = df['姓名'].apply(lambda x: str(x)[0] if pd.notna(x) else '')
```

---

## 3. 数据保存

```python
# 写入单表到 Excel 文件 (不保存索引列)
df.to_excel('输出文件.xlsx', index=False)

# 写入单表到 Excel 文件 (保存索引列)
df.to_excel('输出文件_带索引.xlsx', index=True)

# 写入单表到 CSV 文件 (不保存索引列)
df.to_csv('输出文件.csv', index=False, encoding='utf-8')

# 写入单 Excel 文件多 Sheet
with pd.ExcelWriter('多Sheet输出.xlsx') as writer:
    df1.to_excel(writer, sheet_name='正常数据', index=False)
    df2.to_excel(writer, sheet_name='异常数据', index=False)
```
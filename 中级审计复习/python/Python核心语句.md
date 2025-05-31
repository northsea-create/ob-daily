
# Pandas
> Reference [[Python pandas]] 

## **1. 读写**
```python
import pandas as pd
df=pd.read_excel('文件路径/文件名.xlsx',sheet_name=1)   
df.to_excel('文件路径/文件名.xlsx')
```


## **2. 筛选**
选择行
```python
row=df.loc[df['成绩']<60]
```

选择列 
```python
columns=df[['学号','成绩','姓名']]
```

## **3. 赋值**：交换列、拆分表格

## **4. 填空/重复**

## **5. 聚合函数**：分类汇总、求和
 

## **1. `df.loc[df['Sales'] > 100]` 像SQL的 `SELECT ... WHERE`**

*   **完全正确！** 这是Pandas进行数据筛选最核心、最直观的方式之一。
    *   `df['Sales'] > 100`：这部分生成一个布尔Series (True/False)，相当于SQL `WHERE Sales > 100` 这个条件判断。
    *   `df.loc[...]` (或者直接 `df[...]` 用于布尔索引时)：这部分用布尔Series来选择行，相当于SQL的 `SELECT *` (如果后面不指定列) 或者 `SELECT col1, col2` (如果后面用 `.loc[..., ['col1', 'col2']]` 指定了列)。
*   **超高频率使用是必然的。** 数据分析的很多工作都始于筛选和子集获取。

## **2. 赋值语句 `df['列名'] = ...` 与 `df.loc[condition, '列名'] = ...`**

*   **`df['新列名'] = df['原列名'].apply(lambda ...)`**:
    *   **用途：** 基于'原列名'的每个值，通过`apply`和一个`lambda`函数（或普通函数）计算出一个新值，并将这些新值赋给一个**新的列** '新列名' (如果'新列名'已存在，则会覆盖)。
    *   **核心：** 对一整列进行元素级别的转换，并将结果存入另一列（或自身）。
    *   **示例：** `df['销售额_美元'] = df['销售额_人民币'].apply(lambda x: x / 6.8)`

*   **`df.loc[df['Sales'] > 100, '目标列名'] = value_or_series`**:
    *   **用途：** **只对满足条件 `df['Sales'] > 100` 的那些行**，将其 '目标列名' 的值设置为 `value_or_series`。其他不满足条件的行，其 '目标列名' 的值保持不变（如果是新列，则为NaN）。
    *   **核心：** **条件性赋值 (Conditional Assignment)**。
    *   **示例：**
        *   `df.loc[df['Sales'] > 10000, '客户等级'] = 'VIP'` (对销售额大于10000的客户，将其客户等级设为VIP)
        *   `df.loc[df['成绩'] < 60, '是否及格'] = '否'`
        *   `df.loc[df['城市'] == '北京', '运费'] = 10`

*   **`df.loc[df['Sales'] > 100]` vs `df[df['Sales'] > 100]` 用于筛选行：**
    *   **在单纯用于筛选行（即获取满足条件的行的所有列）时，这两者是等效的，都返回一个包含满足条件行的新DataFrame。**
        ```python
        condition = df['Sales'] > 100
        selected_rows_loc = df.loc[condition]
        selected_rows_direct = df[condition]
        # selected_rows_loc 和 selected_rows_direct 内容是一样的
        ```
    *   **为什么会有 `.loc`？**
        *   **更明确：** `.loc` 显式地表明你是在进行基于标签的索引。
        *   **功能更强大：** `.loc` 不仅可以接受布尔数组，还可以接受行标签、行标签列表、切片等，并且可以同时选择列。例如：
            *   `df.loc[condition, '列A']` (选择满足条件的行的'列A')
            *   `df.loc[condition, ['列A', '列B']]` (选择满足条件的行的'列A'和'列B')
            *   直接用 `df[condition]['列A']` 也能工作，但有时可能会触发 `SettingWithCopyWarning`，因为它可能涉及到链式索引，Pandas不确定你是在修改副本还是原始DataFrame。**使用 `.loc[row_indexer, column_indexer]` 是推荐的避免此警告的方式。**
    *   **简单筛选行，用哪个？**
        *   为了简洁，`df[condition]` 很多人用。
        *   为了更规范和避免潜在的 `SettingWithCopyWarning` (尤其是在后续可能进行赋值操作时)，推荐使用 `df.loc[condition]`。
        *   **在考试中，如果只是简单筛选行且不涉及后续赋值，`df[condition]` 应该是可以接受的。**

## **3. `df` 什么时候用双层中括号 `df[['列名1', '列名2']]`？**

*   **核心区别：返回的是 DataFrame 还是 Series。**
    *   **`df['列名']` (单层中括号，单个字符串列名):**
        *   **返回：一个 Pandas Series 对象。** Series 是一维的带标签数组，你可以把它看作是DataFrame中的一列。
        *   **用途：** 当你只需要操作或获取单一一列的数据时。
    *   **`df[['列名']]` (双层中括号，列名字符串组成的列表，即使列表里只有一个元素):**
        *   **返回：一个 Pandas DataFrame 对象。** 即使你只选择了一列，它也会以DataFrame的形式返回（即一个只有一列的二维表）。
        *   **用途：**
            1.  **选择多列：** `df[['列名1', '列名2', '列名3']]` 这是选择多列的标准方式，返回一个包含这些列的DataFrame。
            2.  **即使只选一列，也想得到DataFrame：** 有时后续操作需要输入一个DataFrame而不是Series，这时即使只选一列也要用双方括号。
            3.  **保持一致性：** 有些人习惯于总是使用双方括号来选择列，以确保返回的总是DataFrame，避免在Series和DataFrame之间切换。

*   **简单总结：**
    *   想拿 **一列数据**，得到 **Series**：`df['col_name']`
    *   想拿 **一列或多列数据**，得到 **DataFrame**：`df[['col_name']]` 或 `df[['col1', 'col2']]`

**考试中的应用：**

*   **赋值给新列或修改现有列**：
    *   `df['新列'] = ...` (直接用单中括号指定列名)
*   **筛选行**：
    *   `filtered_df = df[df['某列'] > 100]` (简洁)
    *   `filtered_df = df.loc[df['某列'] > 100]` (更规范)
*   **从筛选结果中选择特定列**：
    *   `result = df.loc[df['某列'] > 100, ['目标列1', '目标列2']]` (推荐！一步到位)
    *   或者分两步：
        `temp_df = df[df['某列'] > 100]`
        `result = temp_df[['目标列1', '目标列2']]`
*   **当你需要将单列传递给一个期望DataFrame的函数时，或者你想对单列结果执行DataFrame特有的方法时，记得用双方括号。**

你的这些洞察非常好，说明你对Pandas的理解正在深入！继续保持这种思考和总结的习惯。


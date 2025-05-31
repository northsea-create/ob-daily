
***by 东莞市审计局 魏榕（其实是AI）***

**核心原则：先理解问题 -> 识别题型 -> 套用模板 -> 填充细节**

---

## **题型1：数据完整性与勾稽关系校验**

*   **目标：** 检查数据是否符合预定义的计算规则或逻辑关系。
*   **业务场景：** 金额校验 (数量×单价=总价)，比例校验，特定计算公式校验。
*   **代码块模板：**
    ```sql
    SELECT
        *  -- 或者只选择需要关注的关键列和计算结果，方便核对
        -- , columnA, columnB, columnC, (columnA * columnB) AS calculated_C -- 可选，用于展示计算过程
    FROM
        your_table
    WHERE
        columnA * columnB <> columnC  -- 实际的勾稽关系表达式
        -- 或: column_total * (1 - column_ratio) <> column_payment
        -- 或: sales_price / purchase_price > upper_limit_ratio
    ;
    ```
*   **关键点：** `WHERE` 子句中的算术和比较运算符。

---

## **题型2：阈值与频率限制校验 (异常行为筛选)**

*   **目标：** 找出超出规定限额或不符合发生频率要求的数据。
*   **业务场景：** 单日/单周期消费超限，操作间隔过短/过长，数量范围超限。医保、财政等

    *   **子题型2.1：周期内总量超限 (如：单日消费 > 200)**
        *   **代码块模板 (使用窗口函数)：**
            ```sql
            WITH PeriodTotal AS (
                SELECT
                    *, -- 保留所有原始列
                    SUM(value_to_sum) OVER (PARTITION BY grouping_key1, grouping_key2, period_column) AS calculated_period_total
                    -- grouping_key: 如用户ID
                    -- period_column: 如日期 (如果是单日，就按日期分区；如果是月份，则按月份分区)
                FROM
                    your_table
            )
            SELECT
                * -- 或者选择你需要的列
            FROM
                PeriodTotal
            WHERE
                calculated_period_total > threshold_value
            ;
            ```
        *   **关键点：** `SUM() OVER (PARTITION BY ...)`，外层 `WHERE` 筛选。

    *   **子题型2.2：行为频率/间隔限制 (如：隔日才能操作)**
        *   **代码块模板 (使用 `LAG()` 窗口函数 - 推荐)：**
            ```sql
            WITH LaggedData AS (
                SELECT
                    *, -- 保留所有原始列
                    LAG(action_date_column, 1, NULL) OVER (PARTITION BY subject_id_column ORDER BY action_date_column ASC) AS previous_action_date
                    -- subject_id_column: 如用户ID
                    -- action_date_column: 操作发生的日期/时间
                FROM
                    your_table
            )
            SELECT
                * -- 或者选择你需要的列
            FROM
                LaggedData
            WHERE
                previous_action_date IS NOT NULL -- 确保有上一次操作
                AND DATEDIFF('day', previous_action_date, action_date_column) < minimum_interval_days -- DATEDIFF语法因数据库而异
                --  '< minimum_interval_days' 表示间隔太短，违规
                --  '= 1' 表示连续两天操作，如果规则是“不能连续两天”
            ;
            ```
        *   **代码块模板 (使用自连接 - 如果不熟悉窗口函数或考试要求)：**
            ```sql
            SELECT DISTINCT -- 可能需要DISTINCT，取决于你想如何展示违规
                a.* -- 或者选择a表的关键列
            FROM
                your_table a
            JOIN
                your_table b ON a.subject_id_column = b.subject_id_column
            WHERE
                a.primary_key <> b.primary_key -- 确保不是同一条记录 (primary_key可以是唯一ID或时间戳)
                AND DATEDIFF('day', b.action_date_column, a.action_date_column) > 0 -- a在b之后
                AND DATEDIFF('day', b.action_date_column, a.action_date_column) < minimum_interval_days
                -- 为了避免 (A,B) 和 (B,A) 都出现，可以加 a.action_date_column > b.action_date_column
                -- 并调整DATEDIFF的比较逻辑
            ORDER BY
                a.subject_id_column, a.action_date_column;
            ```
        *   **关键点：** `LAG()` 或自连接，日期/时间差函数 (`DATEDIFF`, `JULIANDAY`, `EXTRACT(EPOCH FROM ...)`等)。

    *   **子题型2.3：数量/值在特定范围之外**
        *   **代码块模板：**
            ```sql
            SELECT
                *
            FROM
                your_table
            JOIN -- 如果计算范围的参数在其他表
                another_table ON your_table.key = another_table.key
            WHERE
                (calculated_value < lower_bound OR calculated_value > upper_bound) -- 范围之外
                -- 或者: calculated_value NOT BETWEEN lower_bound AND upper_bound
                -- calculated_value: 可能是直接的列，也可能是计算出来的，如: quantity * unit_usage / daily_dose
            ;
            ```
        *   **关键点：** `WHERE ... BETWEEN ... AND ...` 或 `WHERE ... < ... OR ... > ...`。

---

## **题型3：数据唯一性与连续性校验**

*   **目标：** 检查是否存在重复记录或序列中断。
*   **业务场景：** 流水号、发票号、订单号等是否有重复或断裂。

    *   **子题型3.1：重复值校验**
        *   **代码块模板：**
            ```sql
            SELECT
                column_to_check_for_duplicates,
                COUNT(*) AS occurrence_count
            FROM
                your_table
            GROUP BY
                column_to_check_for_duplicates
            HAVING
                COUNT(*) > 1
            ORDER BY
                occurrence_count DESC, column_to_check_for_duplicates;
            ```
        *   **关键点：** `GROUP BY`, `COUNT(*)`, `HAVING COUNT(*) > 1`。

    *   **子题型3.2：连续性/断号校验**
        *   **代码块模板 (使用 `LEAD()` 或 `LAG()` 窗口函数 - 推荐)：**
            ```sql
            -- 假设 serial_column 是可以转换为整数进行比较的序列号
            WITH NumberedSequence AS (
                SELECT
                    CAST(serial_column AS INT) AS current_val,
                    LEAD(CAST(serial_column AS INT), 1, NULL) OVER (ORDER BY CAST(serial_column AS INT) ASC) AS next_val
                FROM
                    your_table
            )
            SELECT
                current_val AS missing_after_this_number,
                (current_val + 1) AS expected_next_number
                -- , next_val AS actual_next_number -- 可选，用于查看实际的下一个号
            FROM
                NumberedSequence
            WHERE
                next_val IS NOT NULL            -- 排除最后一个号没有下一个的情况
                AND next_val <> (current_val + 1) -- 核心：下一个号不是当前号+1
            ORDER BY
                current_val;
            ```
        *   **关键点：** `LEAD()` (或 `LAG()`), `ORDER BY` 保证序列顺序, `CAST` (如果序列号是文本)。

---

## **题型4：排名与Top N / Bottom N 问题**

*   **目标：** 找出按某个指标排序后，排在前面或后面的N条记录/N个分组。
*   **业务场景：** 贷款金额最高的N个地市，最早的N笔交易。
*   **代码块模板 (使用窗口函数进行排名)：**
    ```sql
    WITH RankedData AS (
        SELECT
            *, -- 或只选需要的列
            SUM(value_to_aggregate_by) AS aggregated_metric, -- 如果需要先聚合再排名
            ROW_NUMBER() OVER (ORDER BY SUM(value_to_aggregate_by) DESC) as rn_desc, -- 降序排名 (Top N)
            ROW_NUMBER() OVER (ORDER BY SUM(value_to_aggregate_by) ASC) as rn_asc   -- 升序排名 (Bottom N)
            -- 如果是按原始列排名，不需要SUM()和GROUP BY
            -- ROW_NUMBER() OVER (PARTITION BY category_column ORDER BY metric_column DESC) as rank_within_category
        FROM
            your_table
        -- WHERE ... -- 可选的预筛选
        GROUP BY -- 如果有聚合，需要GROUP BY所有非聚合的SELECT列
            -- columns_to_group_by
    )
    SELECT
        *
    FROM
        RankedData
    WHERE
        rn_desc <= N -- 取Top N
        -- OR rn_asc <= N -- 取Bottom N
        -- OR rank_within_category <= N -- 取每个分类下的Top N
    ORDER BY
        -- 根据需要排序
        rn_desc;
    ```
*   **代码块模板 (使用 `LIMIT` / `TOP` - 如果数据库支持且逻辑简单)：**
    ```sql
    SELECT
        grouping_column,
        AGGREGATE_FUNCTION(metric_column) AS aggregated_value
    FROM
        your_table
    -- WHERE ...
    GROUP BY
        grouping_column
    ORDER BY
        aggregated_value DESC -- 或 ASC
    LIMIT N; -- PostgreSQL, MySQL, SQLite
    -- SELECT TOP N ... FROM ... ORDER BY ... ; -- SQL Server
    -- SELECT * FROM (SELECT ... FROM ... ORDER BY ...) WHERE ROWNUM <= N; -- Oracle
    ```
*   **关键点：** `GROUP BY` + 聚合函数 (如果需要先聚合), 窗口函数 (`ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`) + `ORDER BY`, `LIMIT` / `TOP`.

---

## **题型5：多表关联与条件聚合筛选 (复杂业务规则判断)**

*   **目标：** 综合多个表的信息，进行筛选、计算，并根据复杂的业务规则找出目标数据。
*   **业务场景：** 财务审计中的多表核对、预算执行分析、跨表数据一致性检查。
*   **代码块模板 (使用CTE逐步分解)：**
    ```sql
    -- CTE 1: 从表A提取和预处理数据
    WITH DataFromTableA AS (
        SELECT
            key_column,
            relevant_column_A1,
            AGGREGATE_FUNCTION(value_A) as aggregated_value_A
        FROM
            table_A
        WHERE
            condition_A
        GROUP BY
            key_column, relevant_column_A1
    ),
    -- CTE 2: 从表B提取和预处理数据
    DataFromTableB AS (
        SELECT
            key_column,
            relevant_column_B1,
            AGGREGATE_FUNCTION(value_B) as aggregated_value_B
        FROM
            table_B
        WHERE
            condition_B
        GROUP BY
            key_column, relevant_column_B1
    )
    -- 主查询: 关联CTE结果，进行最终计算和筛选
    SELECT
        a.key_column,
        a.relevant_column_A1,
        b.relevant_column_B1,
        a.aggregated_value_A,
        b.aggregated_value_B,
        (a.aggregated_value_A / b.aggregated_value_B) AS ratio -- 示例计算
        -- 其他需要的列和计算
    FROM
        DataFromTableA a
    JOIN -- 或 LEFT JOIN, RIGHT JOIN, FULL JOIN 根据需求
        DataFromTableB b ON a.key_column = b.key_column
                         -- AND a.another_join_key = b.another_join_key (其他连接条件)
    WHERE
        -- 最终的筛选条件，可能基于CTE中的计算结果
        (a.aggregated_value_A / b.aggregated_value_B) < threshold_ratio
        OR a.aggregated_value_A > some_value
    HAVING -- 如果主查询中还有聚合，并且需要对聚合结果筛选
        -- COUNT(*) > X
    ORDER BY
        -- 排序
    ;
    ```
*   **关键点：**
    *   **清晰的 `JOIN` 条件：** 准确连接相关的表。
    *   **正确的 `GROUP BY`：** 在需要聚合的地方使用。
    *   **`WHERE` vs `HAVING`：** `WHERE` 作用于行级别数据（分组前），`HAVING` 作用于分组后的聚合结果。
    *   **CTE (`WITH` 子句)：** 用于分解复杂逻辑，提高可读性。

---

## **题型6：特定模式识别 (如：非最优条件中标，共享资源冲突)**

*   **目标：** 找出符合某种特定模式或违反某种规则的数据。
*   **业务场景：** 招投标审计中的围标串标、非最低价中标。
    *  **子题型6.1：非最优条件中标 (如：非最低价中标)**
        *   **代码块模板 (使用窗口函数)：**
            ```sql
            WITH PricedBids AS (
                SELECT
                    *, -- 所有原始列
                    MIN(bid_price_column) OVER (PARTITION BY segment_id_column, product_spec_column) AS min_price_in_group
                    -- segment_id_column, product_spec_column: 分组条件
                    -- bid_price_column: 用于比较的价格列
                FROM
                    bidding_table
            )
            SELECT
                *
            FROM
                PricedBids
            WHERE
                is_winning_bid_column = 1 -- 或 TRUE, 'Y' 等表示中标的条件
                AND bid_price_column > min_price_in_group
            ;
            ```
        *   **关键点：** `MIN() OVER (PARTITION BY ...)`，外层 `WHERE` 比较。

    *   **子题型6.2：共享资源冲突 (如：不同公司用相同MAC地址)**
        *   **代码块模板 (分组计数法 - 推荐)：**
            ```sql
            WITH ConflictingResourceUsage AS (
                SELECT
                    grouping_condition_column1, -- 如：标段号
                    grouping_condition_column2, -- 如：产品规格号
                    shared_resource_column,     -- 如：MAC地址
                    COUNT(DISTINCT subject_id_column) AS distinct_subjects_using_resource
                    -- subject_id_column: 如：投标企业ID或名称
                FROM
                    your_table
                GROUP BY
                    grouping_condition_column1,
                    grouping_condition_column2,
                    shared_resource_column
                HAVING
                    COUNT(DISTINCT subject_id_column) > 1 -- 核心：多个不同主体用了同一个资源
            )
            SELECT DISTINCT -- 如果只需要冲突的条件组合
                grouping_condition_column1,
                grouping_condition_column2
                -- 或者 SELECT * FROM ConflictingResourceUsage; 如果需要看具体的共享资源
            FROM
                ConflictingResourceUsage
            ;
            ```
        *   **代码块模板 (自连接法)：**
            ```sql
            SELECT DISTINCT
                a.grouping_condition_column1,
                a.grouping_condition_column2
            FROM
                your_table a
            JOIN
                your_table b ON a.grouping_condition_column1 = b.grouping_condition_column1
                            AND a.grouping_condition_column2 = b.grouping_condition_column2
                            AND a.shared_resource_column = b.shared_resource_column
            WHERE
                a.subject_id_column <> b.subject_id_column -- 确保是不同的主体
            ;
            ```
        *   **关键点：** `GROUP BY ... HAVING COUNT(DISTINCT ...)` 或自连接 + 主体不同的条件。


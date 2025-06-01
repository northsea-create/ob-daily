
#### 核心概念：
*   **节点 (Nodes):** `(node_variable:Label {property_key: property_value})`
*   **关系 (Relationships):** `-[relationship_variable:TYPE {property_key: property_value}]->` (方向性)
*   **属性 (Properties):** 键值对，存储在节点或关系上。

---

#### 1. 批量导入数据 (Using `LOAD CSV`)

*   **从 CSV 文件批量创建节点：**
    假设 `people.csv` 文件内容如下：
    ```csv
    name,age,city
    Alice,30,London
    Bob,25,Paris
    Charlie,35,New York
    ```
    ```cypher
    // files:/// 是 Neo4j 数据库服务器上文件系统的根目录
    // 确保你的 CSV 文件放在 Neo4j 数据库的 import 目录下
    LOAD CSV WITH HEADERS FROM 'file:///people.csv' AS row
    CREATE (p:Person {name: row.name, age: toInteger(row.age), city: row.city});
    ```
    *   `WITH HEADERS`: 表示 CSV 文件第一行是标题行，可以作为属性名。
    *   `AS row`: 将每行数据映射为 `row` 变量。
    *   `toInteger()`: 转换数据类型。

*   **从 CSV 文件批量创建关系 (通常结合 `MERGE` 先确保节点存在)：**
    假设 `friendships.csv` 文件内容如下：
    ```csv
    person1,person2,since_year
    Alice,Bob,2020
    Charlie,Alice,2019
    ```
    ```cypher
    LOAD CSV WITH HEADERS FROM 'file:///friendships.csv' AS row
    // 确保节点存在（如果不存在则创建）
    MERGE (p1:Person {name: row.person1})
    MERGE (p2:Person {name: row.person2})
    // 创建关系
    CREATE (p1)-[:FRIENDS_WITH {since: toInteger(row.since_year)}]->(p2);
    ```

---

#### 1. 读取数据 (Querying Data)

*   **读取所有节点（带标签和属性）：**
    ```cypher
    // 匹配所有标签为 Person 的节点，并返回它们的名字和年龄
    MATCH (p:Person)
    RETURN p.name, p.age;
    ```

*   **读取带有特定属性的节点：**
    ```cypher
    // 匹配名字为 'Alice' 的 Person 节点，并返回其所有属性
    MATCH (p:Person {name: 'Alice'})
    RETURN p;
    ```

*   **读取节点和它们之间的关系：**
    ```cypher
    // 匹配所有 Person 节点，以及它们之间类型为 FRIENDS_WITH 的关系
    MATCH (p1:Person)-[r:FRIENDS_WITH]->(p2:Person)
    RETURN p1.name AS Friend1, p2.name AS Friend2, type(r) AS RelationshipType;
    ```

---

#### 2. 创建节点 (Creating Nodes)

*   **创建单个节点：**
    ```cypher
    // 创建一个标签为 Person，名字为 'Alice'，年龄为 30 的节点
    CREATE (p:Person {name: 'Alice', age: 30});
    ```

*   **创建多个节点：**
    ```cypher
    // 创建一个 Person 节点 'Bob' 和一个 Movie 节点 'Inception'
    CREATE (p:Person {name: 'Bob', city: 'London'})
    CREATE (m:Movie {title: 'Inception', year: 2010});
    ```

---

#### 3. 创建关系 (Creating Relationships)

*   **创建两个现有节点之间的关系：**
    ```cypher
    // 匹配 'Alice' 和 'Bob'，然后创建它们之间的 FRIENDS_WITH 关系
    MATCH (a:Person {name: 'Alice'}), (b:Person {name: 'Bob'})
    CREATE (a)-[:FRIENDS_WITH]->(b);
    ```

*   **同时创建节点和关系：**
    ```cypher
    // 创建一个 Person 节点 'Charlie'，一个 Movie 节点 'The Matrix'，并创建他们之间的 ACTED_IN 关系
    CREATE (c:Person {name: 'Charlie'})-[:ACTED_IN {roles: ['Neo']}]->(m:Movie {title: 'The Matrix'});
    ```

*   **MERGE (合并)：** 如果节点或关系存在就匹配，不存在就创建。常用于避免重复创建。
    ```cypher
    // 如果名字为 'David' 的 Person 节点不存在就创建，然后创建他与 'Inception' 电影的 REVIEWED 关系
    MERGE (d:Person {name: 'David'})
    MERGE (m:Movie {title: 'Inception'})
    CREATE (d)-[:REVIEWED {rating: 5, date: date('2023-10-26')}]->(m);
    ```

---

#### 4. 更新数据 (Updating Data)

*   **设置/修改节点属性：**
    ```cypher
    // 找到名字为 'Alice' 的 Person 节点，设置其年龄为 31
    MATCH (p:Person {name: 'Alice'})
    SET p.age = 31;

    // 添加新的属性
    MATCH (p:Person {name: 'Alice'})
    SET p.city = 'New York';
    ```

*   **移除节点或关系属性：**
    ```cypher
    // 移除 'Alice' 的 city 属性
    MATCH (p:Person {name: 'Alice'})
    REMOVE p.city;
    ```

---

#### 5. 删除数据 (Deleting Data)

*   **删除关系：**
    ```cypher
    // 找到 'Alice' 和 'Bob' 之间的 FRIENDS_WITH 关系，并删除它
    MATCH (a:Person {name: 'Alice'})-[r:FRIENDS_WITH]->(b:Person {name: 'Bob'})
    DELETE r;
    ```

*   **删除节点（无关系时）：**
    ```cypher
    // 找到名字为 'Frank' 的 Person 节点，并删除它（如果它没有关系）
    MATCH (p:Person {name: 'Frank'})
    DELETE p;
    ```

*   **删除节点及其所有关系 (最常用)：**
    ```cypher
    // 找到名字为 'Charlie' 的 Person 节点，删除它以及它所有的关系
    MATCH (p:Person {name: 'Charlie'})
    DETACH DELETE p;
    ```

*   **删除所有节点和关系（清空数据库 - **谨慎使用！**）：**
    ```cypher
    // 清空数据库中所有数据
    MATCH (n) DETACH DELETE n;
    ```

---

### 区分和记忆方法 (纯 Cypher 视角)

记忆 Cypher 语句的关键在于理解其**声明性、模式匹配**的特点：

1.  **`MATCH` (读取/查询):**
    *   **用途：** 寻找符合特定模式（节点、关系）的数据。
    *   **记忆点：** "匹配" 或 "找到" 数据库中你想要的数据。这是所有查询和更新操作的起点。
    *   **结构：** `MATCH (pattern) [WHERE condition]`

2.  **`CREATE` (创建):**
    *   **用途：** 创建新的节点或关系。
    *   **记忆点：** "创建" 新的数据。
    *   **结构：** `CREATE (new_node_pattern)` 或 `CREATE (node1)-[new_relationship]->(node2)`

3.  **`MERGE` (合并/创建或匹配):**
    *   **用途：** 尝试匹配现有模式，如果不存在则创建。
    *   **记忆点：** "合并" 现有数据，避免重复。当你需要确保某个节点或关系存在，而不管它是新创建的还是已经存在的，就用它。
    *   **结构：** `MERGE (pattern)`

4.  **`SET` (更新属性):**
    *   **用途：** 修改节点或关系的属性值，或添加新属性。
    *   **记忆点：** "设置" 属性的值。
    *   **结构：** `SET variable.property = value`

5.  **`REMOVE` (移除属性/标签):**
    *   **用途：** 移除节点或关系的属性，或移除节点的标签。
    *   **记忆点：** "移除" 某些东西。
    *   **结构：** `REMOVE variable.property` 或 `REMOVE variable:Label`

6.  **`DELETE` / `DETACH DELETE` (删除):**
    *   **用途：** `DELETE` 删除节点或关系。`DETACH DELETE` 更强力，先删除节点的所有关系，再删除节点本身。
    *   **记忆点：** "删除" 数据。`DETACH` 意味着“脱离所有关系再删除”。
    *   **结构：** `DELETE variable` 或 `DETACH DELETE variable`

7.  **`RETURN` (返回结果):**
    *   **用途：** 指定你想要从查询中返回什么数据。
    *   **记忆点：** "返回" 你查询的结果。
    *   **结构：** `RETURN variable`, `RETURN variable.property`, `RETURN count(variable)` 等。

8.  **`LOAD CSV` (导入 CSV):**
    *   **用途：** 从 CSV 文件批量导入数据到数据库。
    *   **记忆点：** "加载 CSV" 文件。
    *   **结构：** `LOAD CSV [WITH HEADERS] FROM 'file:///path.csv' AS row ...`

通过理解这些 Cypher 动词的语义和它们所操作的对象（节点、关系、属性），你就能更好地记忆和区分它们了。它们各自代表了图数据操作中明确的“意图”。
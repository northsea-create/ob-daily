首先，关于日期类型，文档中提到了几种与时间相关的数据类型，虽然没有直接定义它们之间的根本区别，但从操作符和函数的使用示例中可以看出它们代表的不同粒度：

•

DATE: 表示日期，例如 date '2020-10-1'1。

•

TIME: 表示时间，例如 time '9:8:7 AM'2 或 time '9:8:7'3。

•

TIMESTAMP: 表示日期和时间的组合1。例如 timestamp '2020-10-1 9:8:7 AM'2 或 timestamp '2020-10-1 9:8:7'1。SYSDATE 函数返回的就是系统当前的 TIMESTAMP4。

•

INTERVAL: 表示一个时间间隔或持续时间5，可以精确到年/月 (INTERVAL YEAR TO MONTH) 或 日/秒 (INTERVAL DAY TO SECOND)5。例如 interval '1' year1，interval '1-1'year to month3，interval '1 2:3:4.123456' day to second6。

简单来说，从用法上看，DATE 关注年、月、日，TIME 关注时、分、秒等时间部分，而 TIMESTAMP 则包含完整的日期和时间信息。INTERVAL 则用于表示两个时间点之间的跨度。

接下来，文档详细介绍了一些用于处理日期和时间的操作符和函数：

日期/时间/时间戳/时间间隔操作符

文档中介绍了三个主要的操作符用于处理这些类型1：

1.

+ 操作符:

◦

可以在日期/时间/时间戳类型数据的基础上加上一个数值型数据（表示天数）或者一个时间间隔类型数据，得到一个新的日期/时间/时间戳类型数据1。

▪

示例：SELECT date'2020-10-1' +1;，SELECT date '2020-10-1' + interval '1' year;1。

◦

也可以把日期和时间类型数据组合成一个新的时间戳类型数据1。

▪

示例：SELECT date'2020-10-1'+time '9:8:7';1。

2.

- 操作符:

◦

可以在日期/时间戳类型数据基础上减去一个数值（表示天数）或者时间间隔，得到一个新的日期/时间间隔类型数据3。

▪

示例：SELECT date'2020-10-1' -1;，SELECT timestamp'2020-10-1 9:8:7'-interval '1' month;3。

◦

可以计算出两个日期/时间/时间戳类型数据之间的间隔3。

▪

示例：SELECT date'2020-10-1'-date '2019-10-1';，SELECT time'9:8:7'-'19:8:1';，SELECT timestamp'2020-10-1 9:8:7'- timestamp '2020-10-3 9:0:7';3。

3.

OVERLAPS 操作符:

◦

用于判断左右两个时间段之间是否有重叠5。如果重叠，返回 TRUE，否则返回 FALSE5。

◦

操作符两侧的操作数分别表示一个时间段5。时间段的起点是第一个时间，终点是第二个时间5。

◦

如果操作数中包含 INTERVAL YEAR TO MONTH 或 INTERVAL DAY TO SECOND 类型，则时间段的起点是第一个时间，终点是第一个时间加上其后的 Interval 值5。

◦

示例：SELECT (date '2003-10-1',date'2013-10-1') OVERLAPS (date'2008-8-8', date'2020-10-1');5，SELECT (date '2008-8-8',interval '10-1'year to month) OVERLAPS (date '2020-10-1',date '2022-2-4');5。

时间和日期函数

文档详细列举并说明了以下日期和时间函数4...：

1.

SYSDATE:

◦

说明：返回系统当前时间4。

◦

示例：SELECT SYSDATE;4。

2.

ADD_MONTHS:

◦

说明：返回给定日期之前或之后几个月的时间4。

◦

语法：ADD_MONTHS(date_expr,num_expr)4。

◦

示例：SELECT ADD_MONTHS('2020-10-1',-2);4。

3.

MONTHS_BETWEEN:

◦

说明：返回给定的两个日期之间的月份差4。

◦

语法：MONTHS_BETWEEN(date_expr1, date_expr2)4。

◦

示例：SELECT MONTHS_BETWEEN('2008-8-8 20:00:00 ', '2022-2-4 20:00:00');4。

4.

DATE_TRUNC:

◦

说明：给定一个日期，截断成指定的精度（field），如年、月、日等8。

◦

语法：DATE_TRUNC(field, date_expr)8。

◦

示例：SELECT DATE_TRUNC('month',date '2022-2-4');，SELECT DATE_TRUNC('day',timestamp '2022-2-4 7:8:9');8。该函数也支持对 INTERVAL 类型进行截断8。

5.

DATENAME:

◦

说明：给定一个日期，返回指定的域（field）的整数值8。

◦

语法：DATENAME(field, date_expr)8。

◦

示例：SELECT DATENAME('month',date '2022-2-4');8。

6.

DATEADD:

◦

说明：返回指定日期加上一个时间间隔后的新日期值，field 指明了需要加到哪个域上9。

◦

语法：DATEADD(field, date_expr, num_expr) 或 DATEADD(field, num_expr, date_expr)9。注意参数顺序可以调换9。

◦

示例：SELECT DATEADD('day',2,timestamp '2022-2-4 7:8:9');9。

7.

DATEDIFF:

◦

说明：计算两个日期之间的间隔，由 field 指定间隔单位（如年、月、日、秒等）910。

◦

语法：DATEDIFF(field, date_expr1, date_expr2)9。

◦

示例：SELECT DATEDIFF('year','2022-12-31', '2023-1-1');，SELECT DATEDIFF('second','2022-12-31 23:59:59', '2023-1-1 00:00:01');10。

8.

YEAR, MONTH, HOUR, MINUTE, SECOND:

◦

说明：分别返回指定日期/时间中的年、月、时、分、秒域10。

◦

返回值：都是 INT 类型10。

◦

示例：SELECT YEAR(SYSDATE), MONTH(SYSDATE);10。

9.

DAYOFWEEK, DAYOFMONTH, DAYOFYEAR:

◦

说明：分别返回指定日期为一周中的第几天、一个月中的第几天、一年中的第几天10。

◦

返回值：都是 INT 类型10。

◦

注意点: DAYOFWEEK 返回 1 到 7 之间的一个整数，1 代表星期日，2 代表星期一，依此类推11。

◦

示例：SELECT DAYOFWEEK(SYSDATE);11。

除了上述专门的日期时间函数，类型转换函数也可以用于日期类型：

•

TO_CHAR: 转换成 CHAR 类型，可以用于 TIMESTAMP 类型和 DATE 类型，通过格式表达式指定输出格式12。

•

TO_DATE: 将文本型数据转换成 DATE 类型12。输入的文本必须是 TIMESTAMP 类型的格式（间隔符可变），格式表达式可选12。

•

TO_TIMESTAMP: 将文本类型数据转换成 TIMESTAMP 类型13。输入的文本必须是 TIMESTAMP 数据类型的格式（间隔符可变），格式表达式可选1314。

•

TRUNC: 除了截断数字，也可以按指定的日期格式截断日期值15。

•

CAST: 通用类型转换函数，可以将表达式转换成指定的数据类型14，包括日期和时间类型。


# 函数
好的，根据您提供的文档，神通数据库支持多种类型的函数。函数与操作符功能类似，都接受参数并返回结果，可以在查询语句和表达式中使用1。调用函数的通用格式是 function_name(parameters)，参数之间用逗号隔开，可以有零个到多个参数1。

神通数据库支持的函数主要包括以下几类：

1. 数学函数1

◦ABS: 返回给定数字表达式的绝对值1。

◦

CEIL (和 CEILING): 返回大于或等于给定数字表达式的最小整数（取顶函数）1。

◦

FLOOR: 返回小于或等于给定数字表达式的最大整数（取底函数）1。对于 interval day to second 类型，返回值为整数天2。

◦

MOD: 求余数（模）2。

◦

POWER: 返回给定表达式乘指定次方的值2。

◦

ROUND: 返回数字表达式并四舍五入为指定的长度或精度2。可以指定小数点后的位数，也可以指定为负数（例如 -2 表示四舍五入到百位）2。

◦

TRUNC: 截断为指定小数位置的数字23。也可以按指定的日期格式截断日期值23。

2.

字符函数3

◦

CHARINDEX: 返回字符或者字符串在另一个字符串中的起始位置3。可以指定开始查找的位置3。

◦

CONCAT: 字符串连接函数，将多个字符串连接成一个新的字符串34。

◦

LEFT: 返回从字符串左边开始指定字符个数的子串4。

◦

RIGHT: 返回从字符串右边开始指定字符个数的子串，语法与 LEFT 相同4。

◦

LENGTH: 返回字符串长度（字符数）4。

◦

LTRIM: 字符串左截断函数4。

▪

特点/注意点: 如果指定第二个参数 (char_expr2)，它会删除第一个字符串 (char_expr1) 左边出现的 char_expr2 中任意字符，而不是按整个 char_expr2 字符串进行匹配删除45。如果不指定第二个参数，默认删除起始空格4。

◦

RTRIM: 字符串右截断函数5。

▪

特点/注意点: 与 LTRIM 类似，如果指定第二个参数，删除的是右边出现的 char_expr2 中任意字符5。如果不指定第二个参数，默认删除尾随空格5。

◦

REPLACE: 用一个字符串替换另一个字符串的指定子串5。可以将子串替换为空字符串5。

◦

SUBSTR: 返回字符表达式从指定位置开始、指定长度的字符6。起始位置可以指定为负数6。

3.

日期和时间函数6

◦

SYSDATE: 返回系统当前时间6。

◦

ADD_MONTHS: 返回给定日期之前或之后几个月的时间6。

◦

MONTHS_BETWEEN: 返回给定的两个日期之间的月份差67。

◦

DATE_TRUNC: 给定一个日期，截断成指定的精度（field），如年、月、日等7。支持日期、时间戳和时间间隔类型7。

◦

DATENAME: 给定一个日期，返回指定的域（field）的整数值78。

◦

DATEADD: 返回指定日期加上一个时间间隔后的新日期值，field 指明了需要加到哪个域上8。语法支持 DATEADD(field, date_expr, num_expr) 和 DATEADD(field, num_expr, date_expr)8。

◦

DATEDIFF: 计算两个日期之间的间隔，由 field 指定间隔单位（如年、月、日、秒等）89。

◦

YEAR, MONTH, HOUR, MINUTE, SECOND: 分别返回指定日期/时间中的年、月、时、分、秒域，返回值为 INT 类型910。

◦

DAYOFWEEK, DAYOFMONTH, DAYOFYEAR: 分别返回指定日期为一周中的第几天、一个月中的第几天、一年中的第几天，返回值为 INT 类型910。

▪

特点/注意点: DAYOFWEEK 返回 1 到 7 之间的整数，1 代表星期日，2 代表星期一，依此类推10。

4.

聚集函数10

◦

AVG: 计算一组值的平均值10。

◦

SUM: 返回表达式中所有值的和11。

◦

MAX: 返回表达式中的最大值12。

◦

MIN: 返回表达式中的最小值12。

▪

特点/注意点: AVG, SUM, MAX, MIN 在计算时都会忽略空值10...。

▪

语法通常支持 ALL (默认) 和 DISTINCT 参数10...。ALL 对所有值进行计算，DISTINCT 只对唯一值进行计算10...。

▪

num_expr 参数必须是数值类型（对 AVG, SUM）或任何类型（对 MAX, MIN）的表达式，不允许使用聚合函数或子查询10...。

◦

COUNT: 计算输入参数中元素的个数，常用于计算行数11。

▪

特点/注意点: COUNT(*) 返回表或组中的所有行的数量，包括含有空值的行，不需要表达式参数11。

▪

COUNT([ALL|DISTINCT] expr) 计算表达式非空值的数量11。ALL 对组中每一行计算表达式的值，返回非空值的数量（默认）11。DISTINCT 返回组中表达式唯一非空值的数量11。expr 不允许使用聚合函数或子查询11。

5.

其他函数13

◦

类型转换函数:

▪

TO_CHAR: 转换成 CHAR 类型13。可用于数值型和 TIMESTAMP/DATE 类型13。格式表达式 (text_expr) 可指定输出格式13。

▪

TO_DATE: 将文本型数据转换成 DATE 类型1314。输入的文本 (char_expr) 必须是 TIMESTAMP 类型数据的格式，但间隔符可以不是 "-"1314。格式表达式 (text_expr) 可指定输入文本的格式14。

▪

TO_NUMBER: 将整型或者文本数据转换成 NUMERIC 类型14。输入的文本 (expr) 可以包含数字、符号14。需要格式表达式 (text_expr) 来指定转换格式14。

▪

TO_TIMESTAMP: 将文本类型数据转换成 TIMESTAMP 类型1415。输入的文本 (char_expr) 必须是 TIMESTAMP 数据类型的格式，间隔符可以不是 "-"1415。格式表达式 (text_expr) 可选，不指定时参考 TIMESTAMPFORMAT 参数格式15。

▪

CAST: 通用类型转换函数15。语法为 CAST(expr AS data_type)15。

◦

ISNULL:

▪

特点/注意点:

•

一个参数时 (ISNULL(expr1))：如果参数值为空，返回真；否则返回假1516。

•

两个参数时 (ISNULL(expr1, expr2))：如果第一个参数 (expr1) 不为空，返回第一个参数值；否则返回第二个参数值 (expr2)15。若两个参数都为空，则返回空15。

◦

DECODE: 相当于一个条件语句 (IF)16。它将输入值 (expr) 与参数列表中的搜索值 (serach_expr) 依次比较16。如果匹配成功，返回对应的结果值 (result_expr)16。如果未能与任何搜索值匹配成功，则返回默认值 (default)16。

◦

窗口函数 (排序编号函数)1617:

▪

包括 ROW_NUMBER, RANK, DENSE_RANK1718。

▪

与 OVER 子句一起使用17。OVER 子句可以包含 PARTITION BY expr1 (将记录分成多个区，默认所有记录为一个区) 和 ORDER BY expr2 (确定分区内行的排序顺序)17。

▪

ROW_NUMBER: 针对 SELECT 语句返回的每一行，从 1 开始分配连续的编号1617。

•

特点/注意点: 排序时，不考虑并列（ties）的情况，排序编号不会重复，总数不变1718。

▪

RANK:

•

特点/注意点: 排序时，考虑并列的情况，排序编号会重复，但会跳过（skip）重复的序号（例如，如果两个人并列第一，那么下一个序号是第三）1718。总数不变17。

▪

DENSE_RANK:

•

特点/注意点: 排序时，考虑并列的情况，排序编号会重复，但不会跳过（skip）重复的序号（例如，如果两个人并列第一，那么下一个序号是第二）1718。总数会减少（当排序编号有重复时）1718。

这些就是文档中详细介绍的各种函数及其特点和注意点。
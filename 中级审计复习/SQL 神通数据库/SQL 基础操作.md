# 文件导入
不同格式文件导入方式、数据清洗

## 逻辑备份与还原
增量备份、完全备份、差异备份
在数据库维护工具中，还原操作需要在DBA管理工具的控制台中将数据库实例暂停？
物理备份、创建数据库实例
归档数据库/非归档数据库

有这么多筛选条件和语句，什么场合用什么？




# 创建表格、创建数据
根据对象找用户，根据用户找

---创建模式
CREATE SCHEMA test1 AUTHORIZATION a;

---创建用户
CREATE USER lfk WITH PASSWORD '123456' VALID UNTIL '2030-12-31 23:59:59' ROLE sysdba;
ALTER USER lfk WITH PASSWORD '12345';
DROP USER lfk CASCADE;

---创建角色
CREATE ROLE lfk;
DROP ROLE lfk;

GRANT SELECT,UPDATE,DELETE ON loan TO user_lfk WITH GRANT OPTION;


### 创建表
CREATE TABLE 表格 ***直接列变量的时候不用AS!***(
sfz CHAR(18) PRIMARY KEY,
loan VARCHAR(10),
FOREIGN KEY (client_loan) REFERENCES bank(client_loan),
xm VARCHAR(5) NOT NULL UNIQUE,
xb CHAR(2) DEFAULT '男',
income INT CHECK (income>1)); ---TABLESPACE 指定表空间;

```SQL
CREATE TABLE SYSDBA.sale (
sno CHAR(3) CHECK (LEFT(sno,1) BETWEEN 'a' AND 'z'),       # 注意check()，多用()总没错
cph VARCHAR(10),
sdate DATE DEFAULT(sysdate) NOT NULL,
samount INT,
sdj SMALLINT, CHECK (CASE WHEN samount>=1000 THEN sdj<100 ELSE TRUE 
END));
```

---修改表
ALTER TABLE 表格 ADD COLUMN birth DATE;
ALTER TABLE 表格 DROP COLUMN birth;
---主键、外键区别？外键为什么不能直接添加？

ALTER TABLE 表格 ADD CONSTRAINT birth FOREIGN KEY;
ALTER TABLE 表格 ADD CONSTRAINT xb  CHECK (xb='男' OR xb='女');

### 修改列的约束
```SQL
ALTER TABLE TMP ADD FOREIGN KEY (列名) REFERENCES TMP2(列名);    ---外键与参照
ALTER TABLE TMP ADD UNIQUE(列名);
```

### 设置列的索引
设置 sale 表的函数索引 sale_abc，要求先按照产品号的后两位排序，再按销售日期的年降序排序。
```SQL
CREATE INDEX sale_abc ON SYSDBA.SALE RIGHT(RTRIM(CPH,2), YEAR(XSRQ) DESC)
```


---插入数据
INSERT INTO 表格(sfz, loan) ---可以不写
VALUES('201320293084','124')；

---更新数据
UPDATE TABLE 表格
SET xm='林霏开'
WHERE sfz='201320293084';


---创建视图
CREATE VIEW 表格_1 ==AS==
SELECT * FROM 表格 WHERE len(sfz)=18
WITH CHECK OPTION;


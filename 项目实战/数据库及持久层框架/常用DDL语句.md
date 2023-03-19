## PostgreSQL

```sql
-- 创建表
CREATE TABLE my_table_name (
id int8 NOT NULL,
column_1 int8 NULL,
column_2 varchar NULL,
CONSTRAINT pk_id_name PRIMARY KEY(id)
);
-- 修改表名称
ALTER TABLE my_table_name RENAME TO my_change_table_name;
-- 表注释
COMMENT ON TABLE csc_event_enclosure IS '表注释';
-- 删除表
DROP TABLE my_table_name;


-- 添加字段
ALTER table my_table_name ADD COLUMN my_column_name VARCHAR;
COMMENT ON COLUMN my_table_name.my_column_name IS '列注释';
-- 修改字段类型
ALTER TABLE my_table_name ALTER COLUMN my_column_name type VARCHAR;
-- 修改字段名
ALTER TABLE my_table_name RENAME my_column_name TO my_change_column_name;
-- 删除字段
ALTER TABLE my_table_name DROP COLUMN my_column_name;


-- 添加索引
CREATE INDEX index_name ON my_table_name (my_column_name);
-- 删除索引
DROP INDEX index_name;
-- 查看单张表有哪些索引
SELECT * FROM pg_indexes WHERE tablename = 'table_name';
```



## MySQL

```sql
-- 创建表
CREATE TABLE my_table_name (
id int(11) NOT NULL,
column_1 int(11) NULL,
column_2 varchar(255) NULL,
CONSTRAINT pk_id_name PRIMARY KEY(id)
);
-- 修改表名称
ALTER TABLE my_table_name RENAME TO my_change_table_name;
-- 表注释
ALTER TABLE my_table_name COMMENT '表注释';
-- 删除表
DROP TABLE my_table_name;


-- 添加字段
ALTER table my_table_name ADD COLUMN my_column_name VARCHAR(10);
ALTER TABLE my_table_name MODIFY COLUMN my_column_name VARCHAR (10) COMMENT '列注释';
-- 修改字段类型
ALTER TABLE my_table_name MODIFY COLUMN my_column_name VARCHAR(10);
-- 修改字段名
ALTER TABLE my_table_name CHANGE my_column_name my_change_column_name VARCHAR(10);
-- 删除字段
ALTER TABLE my_table_name DROP COLUMN my_column_name;


-- 添加索引
CREATE INDEX index_name ON my_table_name (my_column_name);
-- 删除索引
DROP INDEX INDEX_ACCOUNT ON my_table_name;
-- 查看单张表有哪些索引
SHOW INDEX FROM my_table_name;
-- 查看 MySQL「指定库」/ [指定表]的容量大小
SELECT
	CONCAT(
		table_schema,
		'.',
		table_name
	) AS '表名',
	table_rows AS '总条目数',
	CONCAT(
		ROUND(data_length /(1024 * 1024), 4),
		'MB'
	) AS '数据大小',
	CONCAT(
		ROUND(index_length /(1024 * 1024), 4),
		'MB'
	) AS '索引大小',
	CONCAT(
		ROUND(
			(data_length + index_length) / (1024 * 1024),
			4
		),
		'MB'
	) AS '总大小'
FROM
	information_schema.TABLES
WHERE
	table_schema = 'schema_name'
and table_name = 'table_name';
```


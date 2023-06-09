# Oracle设置自增加主键

> 前提:oracle版本需要为 12c,若之前版本,可以使用trigger+sequence实现.

步骤如下
### 1.配置dialec为org.hibernate.dialect.Oracle12cDialect

### 2.新建表结构
关键字:generated by default as identity (默认值,推荐)
或generated always as identity (必须)
```oracle
create table t1(id number(11) generated by default as identity primary key ,name varchar2(55));
```

### 3.旧表修改(假设表结构为t3(id number(11) primary key ,name varchar2(55)))
```oracle
ALTER TABLE T3 DROP PRIMARY KEY;
alter table t3 add idd number(11) generated by default as identity ;
update t3 set idd = id ;
ALTER TABLE T3 ADD CONSTRAINT T3_pk PRIMARY KEY (idd);
alter table t3 drop column id;
alter table t3 rename column idd to id;
```

### 4.hibernate的话,在表名指定增长策略
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
@Column(name = "id")
```


# 新建表如下 ex即可   ->   `generated as identity` 或 `GENERATED BY DEFAULT AS IDENTITY` 【注：最好使用后者，这里具体百度了解几种主键类型的区别】
```oracle
-- auto-generated definition
create table T_APP_MSG
(
    MSG_ID      NUMBER(11) GENERATED BY DEFAULT AS IDENTITY,
    USER_ID     NUMBER(11),
    MSG_TYPE    NUMBER(2),
    MSG_TIME    TIMESTAMP(6),
    MSG_TYPE_ID NUMBER(11)
)
/

comment on table T_APP_MSG is '消息表'
/

comment on column T_APP_MSG.MSG_ID is '主键ID'
/

comment on column T_APP_MSG.USER_ID is '用户id'
/

comment on column T_APP_MSG.MSG_TYPE is '消息类型 1:空气质量预警 2：监管任务消息 3：信息发布消息 4：信息督办消息'
/

comment on column T_APP_MSG.MSG_TIME is '消息已读时间'
/

comment on column T_APP_MSG.MSG_TYPE_ID is '记录不同消息类型对应的表ID'
/

```

### 修改表主键
```oracle
ALTER TABLE 数据库名.表名 MODIFY(ID Generated as Identity (START WITH 1));
```

### 修改oracle库下所有表的主键ID从最大值开始自增
```oracle
SELECT
	'SELECT ''ALTER TABLE SEWAGE_GY.' || t1.table_name || ' MODIFY(' || t1.Column_Name || ' Generated as Identity (START WITH '' || MAX( ' || t1.Column_Name || '+1 ) || ''));'' FROM ' || t1.table_name || ' UNION ALL' AS FINAL_SQL
FROM cols t1
LEFT JOIN user_col_comments t2 ON t1.Table_name = t2.Table_name AND t1.Column_Name = t2.Column_Name
LEFT JOIN user_tab_comments t3 ON t1.Table_name = t3.Table_name
WHERE
	NOT EXISTS (
        SELECT t4.Object_Name
        FROM User_objects t4
        WHERE
            t4.Object_Type = 'TABLE'
            AND t4.TEMPORARY = 'Y'
            AND t4.Object_Name = t1.Table_Name
	)
	AND t1.IDENTITY_COLUMN = 'YES'
ORDER BY t1.Table_Name, t1.Column_ID
```


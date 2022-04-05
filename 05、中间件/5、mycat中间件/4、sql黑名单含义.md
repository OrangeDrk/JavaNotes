

`<blacklist check="true">selelctAllow</blacklist>`

| 配置项                 | 缺省值 | 描述                                                         |
| ---------------------- | ------ | ------------------------------------------------------------ |
| selelctAllow           | true   | 是否允许执行 SELECT 语句                                     |
| selectAllColumnAllow   | true   | 是否允许执行 SELECT * FROM T 这样的语句。<br />如果设置为 false，不允许执行 select * from t，但可以select * from (select id, name from t) a。<br />这个选项是防御程序通过调用 select * 获得数据表的结构信息。 |
| selectIntoAllow        | true   | SELECT 查询中是否允许 INTO 字句                              |
| deleteAllow            | true   | 是否允许执行 DELETE 语句                                     |
| updateAllow            | true   | 是否允许执行 UPDATE 语句                                     |
| insertAllow            | true   | 是否允许执行 INSERT 语句                                     |
| replaceAllow           | true   | 是否允许执行 REPLACE 语句                                    |
| mergeAllow             | true   | 是否允许执行 MERGE 语句，这个只在 Oracle 中有用              |
| callAllow              | true   | 是否允许通过 jdbc 的 call 语法调用存储过程                   |
| setAllow               | true   | 是否允许使用 SET 语法                                        |
| truncateAllow          | true   | truncate 语句是危险，缺省打开，若需要自行关闭                |
| createTableAllow       | true   | 是否允许创建表                                               |
| alterTableAllow        | true   | 是否允许执行 Alter Table 语句                                |
| dropTableAllow         | true   | 是否允许修改表                                               |
| commentAllow           | false  | 是否允许语句中存在注释，Oracle 的用户不用担心，Wall 能够识别 hints和注释的区别 |
| noneBaseStatementAllow | false  | 是否允许非以上基本语句的其他语句，缺省关闭，通过这个选项就能够屏蔽 DDL。 |
| multiStatementAllow    | false  | 是否允许一次执行多条语句，缺省关闭                           |
| useAllow               | true   | 是否允许执行 mysql 的 use 语句，缺省打开                     |
| describeAllow          | true   | 是否允许执行 mysql 的 describe 语句，缺省打开                |
| showAllow              | true   | 是否允许执行 mysql 的 show 语句，缺省打开                    |
| commitAllow            | true   | 是否允许执行 commit 操作                                     |
| rollbackAllow          | true   | 是否允许执行 roll back 操作                                  |



> 如果把 selectIntoAllow、deleteAllow、updateAllow、insertAllow、mergeAllow 都设置为 false，这就是一个只读数据源了。



# 拦截配置－永真条件

| 配置项                      | 缺省值 | 描述                                                         |
| --------------------------- | ------ | ------------------------------------------------------------ |
| selectWhereAlwayTrueCheck   | true   | 检查 SELECT 语句的 WHERE 子句是否是一个永真条件              |
| selectHavingAlwayTrueCheck  | true   | 检查 SELECT 语句的 HAVING 子句是否是一个永真条件             |
| deleteWhereAlwayTrueCheck   | true   | 检查 DELETE 语句的 WHERE 子句是否是一个永真条件              |
| deleteWhereNoneCheck        | false  | 检查 DELETE 语句是否无 where 条件，这是有风险的，但不是 SQL 注入类型的风险 |
| updateWhereAlayTrueCheck    | true   | 检查 UPDATE 语句的 WHERE 子句是否是一个永真条件              |
| updateWhereNoneCheck        | false  | 检查 UPDATE 语句是否无 where 条件，这是有风险的，但不是SQL 注入类型的风险 |
| conditionAndAlwayTrueAllow  | false  | 检查查询条件(WHERE/HAVING 子句)中是否包含 AND 永真条件       |
| conditionAndAlwayFalseAllow | false  | 检查查询条件(WHERE/HAVING 子句)中是否包含 AND 永假条件       |
| conditionLikeTrueAllow      | true   | 检查查询条件(WHERE/HAVING 子句)中是否包含 LIKE 永真条件      |



# 其他拦截配置

| 配置项                    | 缺省值 | 描述                                                         |
| ------------------------- | ------ | ------------------------------------------------------------ |
| selectIntoOutfileAllow    | false  | SELECT ... INTO OUTFILE 是否允许，这个是 mysql 注入攻击的常见手段，缺省是禁止的 |
| selectUnionCheck          | true   | 检测 SELECT UNION                                            |
| selectMinusCheck          | true   | 检测 SELECT MINUS                                            |
| selectExceptCheck         | true   | 检测 SELECT EXCEPT                                           |
| selectIntersectCheck      | true   | 检测 SELECT INTERSECT                                        |
| mustParameterized         | false  | 是否必须参数化，如果为 True，则不允许类似 WHERE ID = 1 这种不参数化的 SQL |
| strictSyntaxCheck         | true   | 是否进行严格的语法检测，Druid SQL Parser 在某些场景不能覆盖所有的SQL 语法，出现解析 SQL 出错，可以临时把这个选项设置为 false，同时把 SQL 反馈给 Druid 的开发者。 |
| conditionOpXorAllow       | false  | 查询条件中是否允许有 XOR 条件。XOR 不常用，很难判断永真或者永假，缺省不允许。 |
| conditionOpBitwseAllow    | true   | 查询条件中是否允许有"&"、"~"、"                              |
| conditionDoubleConstAllow | false  | 查询条件中是否允许连续两个常量运算表达式                     |
| minusAllow                | true   | 是否允许 SELECT * FROM A MINUS SELECT * FROM B 这样的语句    |
| intersectAllow            | true   | 是否允许 SELECT * FROM A INTERSECT SELECT * FROM B 这样的语句 |
| constArithmeticAllow      | true   | 拦截常量运算的条件，比如说 WHERE FID = 3 - 1，其中"3 - 1"是常量运算表达式。 |
| limitZeroAllow            | false  | 是否允许 limit 0 这样的语句                                  |

​            

# 禁用对象检测配置

| 配置项         | 缺省值 | 描述                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| tableCheck     | true   | 检测是否使用了禁用的表                                       |
| schemaCheck    | true   | 检测是否使用了禁用的 Schema                                  |
| functionCheck  | true   | 检测是否使用了禁用的函数                                     |
| objectCheck    | true   | 检测是否使用了“禁用对对象”                                   |
| variantCheck   | true   | 检测是否使用了“禁用的变量”                                   |
| readOnlyTables | 空     | 指定的表只读，不能够在 SELECT INTO、DELETE、UPDATE、INSERT、MERGE 中作 |


[toc]

# 1. mysql中的datatime慢13小时

> JDK8中的日期处理对应MySQL中的日期类型

| JDK8          | MySQL 5+ |
| ------------- | -------- |
| LocalDate     | date     |
| LocalTime     | time     |
| LocalDateTime | datetime |

其中LocalDateTime入库和取出时可能会慢/快 13小时/xx小时。这是因为MySQL时区问题

**设置MySQL时区即可修复该问题：**

```sql
set global time_zone='+08:00';
set time_zone='+08:00';
```

**查看MySQL当前时区：**

```sql
show variables like '%time_zone'
```


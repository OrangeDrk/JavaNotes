[TOC]

# 配置Maven

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase</artifactId>
        <version>1.3.1</version>
        <type>pom</type>
    </dependency>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-common</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-server</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
    </dependency>
</dependencies>
```



# API操作

## 创建表

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.junit.Test;

import java.io.IOException;

public class HBaseDemo {

    // 创建表
    @Test
    public void createTable() throws IOException {

        // create 'student', 'basic', 'info'
        // 获取HBase的配置
        Configuration conf = HBaseConfiguration.create();
        // 通过Zookeeper连接HBase，需要设置Zookeeper的连接地址
        conf.set("hbase.zookeeper.quorum", "10.42.0.33:2181,10.42.24.13:2181,10.42.104.102:2181");
        // 获取表的管理权
        HBaseAdmin admin = new HBaseAdmin(conf);
        // 构建一个表描述器
        HTableDescriptor table = new HTableDescriptor(TableName.valueOf("student"));
        // 构建列描述器
        HColumnDescriptor cf1 = new HColumnDescriptor("basic");
        HColumnDescriptor cf2 = new HColumnDescriptor("info");
        // 添加列族
        table.addFamily(cf1);
        table.addFamily(cf2);
        // 建表
        admin.createTable(table);
        // 关闭管理权
        admin.close();
    }

}

```

> 执行成功后，在HBase中通过list查看，会发现多了一张`student`表



## 添加/修改数据

```java
// 添加数据/修改数据
@Test
public void putData() throws IOException {

    // put 'student', 'p1', 'basic:name', 'Helen'
    // put 'student', 'p1', 'info:addr', 'beijing'
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置Zookeeper的连接地址
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    // 连接表
    HTable table = new HTable(conf, "student");
    // 封装Put对象
    // 参数表示的是行键
    Put put = new Put("s1".getBytes());
    // family - 列族
    // qualifier - 列
    // value - 数据值
    put.addColumn("basic".getBytes(), "name".getBytes(), "Helen".getBytes());
    put.addColumn("info".getBytes(), "addr".getBytes(), "beijing".getBytes());
    // 添加数据
    table.put(put);
    // 关闭连接
    table.close();
}
```



## 百万条数据写入

```java
// 测试：添加百万条数据
// 花费时间：17536 ~ 17.5s
@Test
public void putMillionData() throws IOException {

    long begin = System.currentTimeMillis();
    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    List<Put> list = new ArrayList<>();
    for (int i = 0; i < 1000000; i++) {
        Put put = new Put(("s" + i).getBytes());
        put.addColumn("basic".getBytes(), "id".getBytes(), ("no" + i).getBytes());
        list.add(put);
        // 每1w条数据，就向HBase表中添加一次
        if (list.size() >= 10000) {
            table.put(list);
            list.clear();
        }
    }
    table.close();
    long end = System.currentTimeMillis();
    System.err.println(end - begin);
}
```



## 获取数据

```java
// 查询数据
@Test
public void getData() throws IOException {

    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    // 封装Get对象
    Get get = new Get("s1".getBytes());
    get.addColumn("basic".getBytes(), "name".getBytes());
    // 查询数据，将查询的结果封装到Result中
    Result r = table.get(get);
    byte[] value = r.getValue("basic".getBytes(), "name".getBytes());
    System.err.println(new String(value));
    table.close();
}
```



## 获取结果集

```java
// 遍历数据
@Test
public void scanData() throws IOException {

    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    // 封装Scan对象
    // Scan scan = new Scan();
    // 从指定行键开始打印到最后
    // Scan scan = new Scan("s9".getBytes());
    // 指定范围
    Scan scan = new Scan("s90".getBytes(), "s91".getBytes());
    // 遍历数据，将结果封装成一个ResultScanner - 结果扫描器返回
    ResultScanner rs = table.getScanner(scan);
    Iterator<Result> it = rs.iterator();
    while (it.hasNext()) {
        Result r = it.next();
        System.err.println(new String(r.getValue("basic".getBytes(), "id".getBytes())));
    }
    table.close();
}
```



## 删除数据

```java
// 删除数据
@Test
public void deleteData() throws IOException {
    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    // 封装成Delete对象
    Delete del = new Delete("s1".getBytes());
    del.addColumn("info".getBytes(), "addr".getBytes());
    // 删除数据
    table.delete(del);
    // 关闭连接
    table.close();
}
```



## 删除表

```java
// 删除表
@Test
public void dropTable() throws IOException {
    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    // 获取管理权
    HBaseAdmin admin = new HBaseAdmin(conf);
    // 禁用表
    admin.disableTable("student");
    // 删除表
    admin.deleteTable("student");
    // 关流
    admin.close();
}
```



## 正则过滤器

```java
/*
 * 正则过滤器
 */
@Test
public void scanData() throws Exception {
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置zookeeper的地址
    conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
    // 获取表
    HTable table = new HTable(conf, "student");
    // 获取迭代器
    Scan scan = new Scan();
    // 正则过滤器，匹配行键含3的行数据
    Filter filter = new RowFilter(CompareOp.EQUAL, new RegexStringComparator(".*3.*"));
    // 加入过滤器
    scan.setFilter(filter);
    ResultScanner scanner = table.getScanner(scan);
    // 获取结果的迭代器
    Iterator<Result> it = scanner.iterator();
    while (it.hasNext()) {
        Result result = it.next();
        // 通过result对象获取指定列族的列的数据
        byte[] value = result.getValue("basic".getBytes(), "id".getBytes());
        System.out.println(new String(value));
    }
    scanner.close();
    table.close();
}
```



## 行键过滤器

```java
@Test
public void rowKeyCompare() throws Exception {
	// 获取配置
	Configuration conf = HBaseConfiguration.create();
	// 设置zookeeper的地址
	conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
	// 获取表
	HTable table = new HTable(conf, "student");
	Scan scan = new Scan();
	// 行键比较过滤器，下例是匹配小于或等于指定行键的行数据
	Filter filter = new RowFilter(CompareOp.LESS_OR_EQUAL, new BinaryComparator("100".getBytes()));
	scan.setFilter(filter);
	ResultScanner scanner = table.getScanner(scan);
	Iterator<Result> it = scanner.iterator();
	while (it.hasNext()) {
		Result result = it.next();
		byte[] value = result.getValue("basic".getBytes(), "id".getBytes());
		System.out.println(new String(value));
	}
	scanner.close();
	table.close();
}
```



## 行键前缀过滤器

```java
@Test
public void prefix() throws Exception {
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置zookeeper的地址
    conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
    // 获取表
    HTable table = new HTable(conf, "student");
    Scan scan = new Scan();
    // 行键前缀过滤器
    Filter filter = new PrefixFilter("3".getBytes());
    scan.setFilter(filter);
    ResultScanner scanner = table.getScanner(scan);
    Iterator<Result> it = scanner.iterator();
    while (it.hasNext()) {
        Result result = it.next();
        byte[] value = result.getValue("basic".getBytes(), "id".getBytes());
        System.out.println(new String(value));
    }
    scanner.close();
    table.close();
}
```



## 列值过滤器

```java
@Test
public void colScan() throws Exception {
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置zookeeper的地址
    conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
    // 获取表
    HTable table = new HTable(conf, "student");
    Scan scan = new Scan();
    // --列值过滤器
    Filter filter = new SingleColumnValueFilter("basic".getBytes(), "name".getBytes(), CompareOp.EQUAL,
                                                "rose".getBytes());
    scan.setFilter(filter);
    ResultScanner scanner = table.getScanner(scan);
    // --获取结果的迭代器
    Iterator<Result> it = scanner.iterator();
    while (it.hasNext()) {
        Result result = it.next();
        byte[] name = result.getValue("basic".getBytes(), "name".getBytes());
        byte[] age = result.getValue("basic".getBytes(), "age".getBytes());
        System.out.println(new String(name) + ":" + new String(age));
    }
    scanner.close();
    table.close();
}
```




# MySQL支持的索引类型

## B-tree索引的特点

B-tree过引以`B+树`的结构存储数据

![image-20200707081614163](C:\Users\12605\Desktop\PHP_notes\.img\image-20200707081614163.png)
B-tree索引能够加快数据的`查询速度`
B-tree索弓更适合进行`范围查找`，`顺序存储`

​	

### 在什么情况下可以用到B树索引

- 全值匹配的查询
  	`order_sn= '9876432119900'`

- 匹配最左前缀的查询 `第一列` 第二列无法匹配

  order_sn与order_date的联合索引

- 匹配列前缀查询
  `order_sn like '9876%'`

- 匹配范围值的查询
  `order_sn > '9876432119900'`
  `and order_sn < ’9876432119999'`

- `精确`匹配左前列并`范围`匹配另外一列
- 只访问索引的查询

### Btree索引的使用限制

- 如果不是按照索引|最左列开始查找，则无法使用索引
- 使用索引时不能跳过索引中的列
  - 如order_sn、`name`、order_date

- `Not in`和`<>`操作无法使用索引
- 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引

## Hash索引的特点

- Hash索引是基于`Hash表`实现的,只有查询条件`精确`匹配Hash索引中的`所有`列时，才能够使用到hash索引。

- 对于Hash索引中的所有列，存储引擎都会为每一行计算一个Hash码，Hash索引中存储的就是Hash码。

### Hash索引的限制

- Hash索引必须进行`二次查找`
- Hash索引无法用于`排序`
- Hash索引不支持`部分索引查找`也不支持`范围查找`

- Hash索引中Hash码的计算可能存在Hash冲突

## 为什么要使用索引

- 索引大大减少了存储引擎需要扫描的`数据量`

- 索引可以帮助我们进行排序以避免使用`临时表`
- 索引可以把`随机`I/O变为`顺序`I/O

### 索引是不是越多越好

- 索引会增加`写操作`的成本
- 太多的索引会增加`查询优化器的选择时间`

# 索引优化策略

## 索引优化策略

### `索引列`上不能使用`表达式`或`函数`

![image-20200707083502346](C:\Users\12605\Desktop\PHP_notes\.img\image-20200707083502346.png)

### `前缀索引`和`索引列`的选择性

![image-20200707083746987](C:\Users\12605\Desktop\PHP_notes\.img\image-20200707083746987.png)

### 联合索引

#### 如何选择索引列的顺序

- `经常会被使用到的`列优先	 `优先：从左到右，所以放到左边`
- `选择性高`的列优先
- `宽度小`的列优先

### 覆盖索引

#### 优点

- 可以优化缓存，减少`磁盘IO`操作
- 可以减少`随机`IO ,变随机IO操作变为顺序IO操作
- 可以避免对Innodb`主键索引`的二次查询

- 可以避免MyISAM表进行`系统调用`

#### 无法使用覆盖索引的情况

- 存储引擎不支持覆盖索引
- 查询中使用了太多的`列`
- 使用了`双%号`的`like`查询

### 查看是否使用了索引

```mysql
explain select lanauage_id id from film where lanauage_id=1\G
```

![image-20200707084908773](C:\Users\12605\Desktop\PHP_notes\.img\image-20200707084908773.png)

覆盖索引的使用

![image-20200707085009492](C:\Users\12605\Desktop\PHP_notes\.img\image-20200707085009492.png)

## 使用索引来优化查询

### 使用索引扫描来优化排序

- 通过排序操作
- 按照索引顺序扫描数据
- 索引的`列顺序`和`Order By`子句的顺序完全一致
- 索引中所有`列的方向`(升序降序)和`Order by`子句完全一致
- Order by中的字段全部在关联表中的第一-张表中

![image-20200707090626696](C:\Users\12605\Desktop\PHP_notes\.img\image-20200707090626696.png)

### 模拟Hash索引优化查询

- 只能处理键值的全值匹配查找
- 所使用的Hash函数决定着索引键的大小

### 利用索引优化锁

- 索引可以减少锁定的行数
-  索引可以加快处理速度，同时也加快了锁的释放

## 索引的维护和优化

### 删除重复和冗余的索引

![image-20200708110558241](C:\Users\12605\Desktop\PHP_notes\.img\image-20200708110558241.png)
#### 联合索引的冗余

![image-20200708110637317](C:\Users\12605\Desktop\PHP_notes\.img\image-20200708110637317.png)

#### 检查索引是否冗余

`pt-duplicate-key-checker h=127.0.0.1`

`create index idx_customerid_staffid on payment (customer_ id,staff id);`

<img src="C:\Users\12605\Desktop\PHP_notes\.img\image-20200708110734518.png" alt="image-20200708110734518" style="zoom:80%;" />

#### 查找未被使用过的索引

<img src="C:\Users\12605\Desktop\PHP_notes\.img\image-20200708110929784.png" alt="image-20200708110929784" style="zoom:80%;" />

#### 更新索引统计信息及减少索引碎片

```mysql
analyze table table_name
optimize table table_name #会锁表
```


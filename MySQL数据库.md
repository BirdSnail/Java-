# 1. create table 关键字

## 1.1 约束

### 1. primary key

### 2. key

### 3. unique key

### 4. foreign key



# 2. 索引

- 索引如何工作取决于选用的数据库引擎。

- 创建唯一约束时会自动创建索引。

## 2.1 索引优化

- 最左匹配原则

- 索引列尽可能的少，查询尽量走索引

- 联合索引时考虑将所有字段都加入`where`条件中。

- `select`查找的列中只有联合索引的列，可以避免回表。

  ```mysql
  select id, name, sex from user where id = '123' and name='lisan' and sex='01'
  ```

  因为联合索引的形成的key中已经包含了要查找的数据，不需要再根据查找到的主键索引再查找一次B+树。

  

# 3. 事务

## 3.1 ACID	

### Automicity：原子性

- 失败是丢弃所有的变更，要么提交，要么回滚

### Consistency：一致性

- 对数据一致性的约束（外键约束，Unique约束，NOT NULL 约束...）要始终成立：帐要平

### Isolation：隔离性

- 并发条件下，事务之间互相独立

### Durability：持久性

- 数据的变更进入持久化的设备（落盘）

## 3.2 事务的隔离等级

- Read uncommitted ：读未提交
- Read committed ：读已提交
- Repeatable read ：重复读
- Serializable ：序列化（串行）

### 3.2.1 多事务并发场景下的问题

- 脏读

  - 一个事务读取到了另一个事务未提交的写入

- 不可重复读

  - 一个事务多次读取到的数据不一样。

    如一个事务第一次读取到数据后，另一个事务修改了数据，头一个事务第二次读取的时候就会发现数据不一致。

- 幻读

  - check then act：

### 3.2.2 不同隔离等级可以解决的问题

![image-20200426105339845](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200426105339845.png)

## 3.3 分布式事务

两阶段提交，需要引入一个协调者`Coordinator`

![image-20200426155211293](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200426155211293.png)

- 第一阶段
  - Coordinator询问参与事务的节点是否准备好
- 第二阶段
  - 参与者必须提交自己的事务
  - 无论发生什么都要保证自己事务的提交

# 4. 数据存储结构

## B树

- 自平衡，高度很低的树（查找时尽可能的读取多的数据，减少磁盘IO）

### 度 order

- 每个节点的子节点数量在`[order/2,order]`之间。

- 每个节点有一定的空闲
  - 无需频繁的旋转
  - 插入时只需要移动少量的数据

### 磁盘预读：

- 可以将一个节点的所有数据一次性的读取到内存中。
  - 一个节点的数据放在相邻的页中（一页4k）

## B+树

- B树的改进
  - 非叶子节点只存放索引，所有的数据都放在叶子节点中(数据存放在树的最底层)
  - 为每个叶子节点增加链接的指针
- 特点
  - 每次查询都到叶子节点
    - 性能稳定
  - 非叶子节点只存放索引，因此可以容纳更多的索引
    - 树的高度更低
  - 叶子节点的指针使得范围查找分为方便



# 5. 存储引擎

## MyISAM引擎

- B+树
- 叶子节点存储数据的地址
- 主索引和次级索引没有区别

![image-20200428164031901](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200428164031901.png)

![mysqlæ°æ®è¡¨å­å¨å¼æç±»ååç¹æ§](https://imgedu.lagou.com/6c01bcc415e34fe8b06850b09f15700c.jpg)

## InnoDB引擎

- B+树
- 叶子节点的数据域就是数据本身

![image-20200428164602256](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200428164602256.png)

- 次级索引指向了主索引

![image-20200428164708991](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200428164708991.png)

### 两者对比

![image-20200428164950539](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200428164950539.png)



# 6. 日志系统

- MySQL服务层的日志：`binlog`，记录的是数据的变更，不是sql语句。
  - 恢复
  - 复制
- 引擎层的日志：
  - redo log：用于在崩溃时恢复未完成的事务。
    在内存中修改的数据没有落到磁盘中，存在于buffer pool中的也数据为`脏数据（dirty page）`，将buffer pool中的变更记录到redo log中可以防止此时还存在内存中没有落到磁盘中的数据的丢失。
  - undo log：保存事务发生前数据的版本，用于撤销修改（回滚）。
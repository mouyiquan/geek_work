
### 题目 01- 完成 ReadView 案例，解释为什么 RR 和 RC 隔离级别下看到查询结果不一致
要求：

* 完成案例 01- 读已提交 RC 隔离级别下的可见性分析
* 完成案例 02- 可重复读 RR 隔离级别下的可见性分析
* 用通俗易懂的方式记录整个案例过程，可以画图与截图
* 做完案例给出结论，并对结论进行分析

回答范式：

**1.案例 01- 读已提交 RC 隔离级别下的可见性分析**
目标：验证存在 ***不可重复读***  问题
操作步骤：
更新同一条数据，数据值为**老王**。
| 时间 | 事务1(trx_id = 1) | 事务2(trx_id = 2) | 事务3(trx_id = 3) |
| --- | --- | --- | --- |
| T1 | 开启事务 | 开启事务  | 开启事务  |
| T2 | 更新为小张 |  |  |
| T3 |  | 更新为小四 |  |
| T4 |  |  | SELECT 1|
| T5 | 提交事务01 |  |  |
| T6 |  |  | SELECT 1|
| T7 |  | 提交事务02 |  |
| T8 |  |  | SELECT 1|
实践过程：
按照时间线执行sql
* T4: 老王
* T6: 小张
* T8: 小四

结论：
在同一个事务中多次查询同一条记录，会出现 ***不可重复读*** 现象。

**2.案例 02- 可重复读 RR 隔离级别下的可见性分析**
目标：验证MySQK是否存在 ***幻读***  问题，并验证是否存在 ***不可重复读*** 现象
操作步骤：
表中只有一条数据，数据值为**老王**。
| 时间 | 事务1(trx_id = 1) | 事务2(trx_id = 2) | 事务3(trx_id = 3) |
| --- | --- | --- | --- |
| T1 | 开启事务 | 开启事务  | 开启事务  |
| T2 | 更新为小张 |  |  |
| T3 |  |  |  SELECT 1|
| T4 | 提交事务01 |  |  |
| T5 |  |  | SELECT 1 |
| T6 |  |  | SELECT All|
| T7 |  | 插入数据 |  |
| T8 |  | 提交事务02 |  |
| T9 |  |  | SELECT All|
实践过程：
按照时间线执行sql
* T3: 老王
* T5: 老王
* T6: 一条数据
* T9: 一条数据

结论：
在同一个事务中多次查询同一条记录，不会出现 ***不可重复读*** 现象，多次查询值都一样，多次查询所有记录验证未出现幻读现象，仅针对于MySQL数据库。


**3.结论分析**

### 题目 02- 什么是索引？
要点：

**1. 优点是什么？**

* 查询更快
* 如果包含所需要的数据甚至不需要回表

**2. 缺点是什么？**

* 会有空间上的消耗
* 数据删除或者更新需要同步维护索引
* 索引太多，加锁也是开销

**3. 索引分类有哪些？**

| 索 | 引 | 分 | 类 | |
| --- | --- | --- | --- | --- |
| 数据结构 | B+树 | Hash | Full-text | |
| 物理存储 | 聚簇索引（主键索引） | 二级索引（辅助索引） |  | |
| 字段特性 | 主键索引 | 唯一索引 | 普通索引 | 前缀索引 |
| 字段个数 | 单列索引 | 联合索引 |  | |
**4. 特点是什么？**

主键索引：叶子节点存储的是完整的数据
辅助索引：叶子节点存储的是主键索引，需要二次查询主键索引B+树，这种情况就叫回表；如果在二级索引找到了需要的数据，则直接返回，这种场景叫做索引覆盖。

**5. 索引创建的原则是什么？**

1.使用联合索引时，存在最左匹配原则，也就是按照最左优先的方式进行索引的匹配，所以在创建的时候需要注意。
2.经常用于 GROUP BY 和 ORDER BY 的字段
3.经常用于 WHERE 查询条件的字段
4.left join on的字段


**6. 有哪些使用索引的注意事项？**

1.范围查询的字段可以用到联合索引，但是在范围查询字段的后面的字段无法用到联合索引。
2.主键索引最好是自增，避免插入中间数据造成页分裂
3.索引覆盖优化，避免回表
4.防止索引失效

**7. 如何知道 SQL 是否用到了索引？**
1.使用explain解析sql语句。如果key列下面有索引就代表使用了索引。
2.当你的查询语句的执行计划里，出现了 Extra 为 Using index condition

**8. 请你解释一下索引的原理是什么？「重点」说清楚为什么要用 B+Tree**

索引就像字典的目录一样，可以快速定位。磁盘访问的速度是毫秒级别的，但是内存的访问速度是纳秒级别的，差了上万倍，索引索引的结构尤为重要。
Redis用的Hash索引，Hash就需要Hash碰撞，通常为数组+链表，因为不能的值可能出现相同的hash值，就需要用到链表，但是数组的插入性能很低。

为什么不用二叉树：二叉树根的深度会无限的向下延伸，I/O 次数越多，查询不稳定。
为什么不用二叉查询树：当每次插入的元素都是二叉查找树中最大的元素，二叉查找树就会退化成了一条链表，查找数据的时间复杂度变成了 O(n)。
为什么不用平衡二叉树，平衡二叉树也会有甚多的空叶子接点，I/O 次数越多。

为什么不用B树: B树的每个节点都会存放索引和记录，这就会大大的减少索引的数量也会导致1页的索引数量过低，增加了树的层级，导致I/O次数增加。

B+树和B树区别：

* 叶子节点才会存放实际数据（索引+记录），非叶子节点只会存放索引；
* 所有索引都会在叶子节点中，叶子节点构成一个有序的链表。
* B+树索引节点不存放数据，一个页可以存放更多的索引，所以层级更低，I/O次数更少。
* B+树存在冗余的非叶子节点。所以叶子节点删除，树的变形概率比B树更小，因为B树没有冗余数据。所以删除效率更高。



存在大量范围检索的场景，适合使用 B+树，比如数据库。而对于大量的单个索引查询的场景，可以考虑 B 树，比如 nosql 的MongoDB。

- 索引查询次数稳定，所需要的I/O次数更少
-

### 题目 03- 什么是 MVCC？
要点：

**1. Redo 日志**
作用：

* 实现事务的持久性，可用于宕机恢复数据。
* WAL的技术落地，先写日志再写数据。
* 随机写变顺序写。

redo log不是直接落盘的，在buffer pool中有自己的区域。有三种策略用于落盘：

* 每次提交
* 文件超过设置大小的一半
* 每隔1s钟落盘

**2. ReadView**

作用：
* 跟undo log配合实现可重复读和读已提交的实现。
* 在MVCC中解决了幻读。


ReadView不可或缺的几个要点：
- min_trx_id 创建时活跃事务的最小trx_id。
- max_trx_id 下一次创建trx_id的值。
- m_ids 当前数据库中活跃事务的事务id列表，表示已启动而未提交的事务。
- trx_id 当前事务的id
- roll_pointer 指向undo log里面的旧数据的指针



**3. 如何判断可见性**
可重复读和读已提交的区别在于，读的时候是否新建ReadView. 新建则会读到已提交数据。
可重复读：通过ReadView里面的m_ids判断，如果trx_id在m_ids中，则表示在当前事务开始时，该trx_id是活跃且未提交状态，需要根据undo log的roll_pointer链往下继续找，直到找到的trx_id小于min_trx_id为止。

读已提交：事务在每次查询时，重新创建ReadView，如果是已提交的数据，trx_id则不会出现在新建的ReadView的m_ids里面。
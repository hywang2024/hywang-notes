## MYSQL

### MVCC
`MVCC`的实现依赖于：隐藏字段、`Read View`、`undo log`

`InnoDB`通过数据行的`DB_TRX_ID`和`Read View`来判断数据的可见性，如不可见，则通过数据行的 DB_ROLL_PTR 找到 undo log 中的历史版本

每个事务读到的数据版本可能是不一样的，在同一个事务中，用户只能看到该事务创建`Read View`之前已经提交的修改和该事务本身做的修改

#### 隐藏字段
`InnoDB`为每行数据都添加了三个隐藏字段
* `DB_TRX_ID`：最后一次执行的事务ID
* `DB_ROLL_PTR`：回滚指针，指向该行的`undo log`日志
* `DB_ROW_ID`：没有唯一非空索引事，会通过该字段生成聚簇索引

#### `ReadView`
`ReadView`主要是用来做可见性判断。
* `m_low_limit_id`：目前出现过的最大的事务 ID+1，即下一个将被分配的事务 ID。大于等于这个 ID 的数据版本均不可见
* `m_up_limit_id`：活跃事务列表`m_ids`中最小的事务 ID，如果`m_ids`为空，则`m_up_limit_id`为`m_low_limit_id`。小于这个 ID 的数据版本均可见
* `m_ids`：`ReadView`创建时其他未提交的活跃事务 ID 列表。创建`ReadView`时，将当前未提交事务 ID 记录下来，后续即使它们修改了记录行的值，
对于当前事务也是不可见的。`m_ids`不包括当前事务自己和已提交的事务（正在内存中）
* `m_creator_trx_id`创建该`ReadView`的事务 ID

`ReadView`是什么时候生成的
* `READ-COMMITTED`(读取已提交)：每次`SELECT`查询之前都生成一个`ReadView`
* `REPEATABLE-READ`(可重复读)：第一次`SELECT`数据时生成一个`ReadView`

#### `undo log`
`undo log`主要有两个作用
* 当事务回滚时用于将数据恢复成修改之前
* MVCC中当读取记录时，若该记录被其他事务占用或当前版本对该事务不可见，`undo log`读取之前版本的数据

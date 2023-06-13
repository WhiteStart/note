# 数据类型

## 数值类型

|   类型    |  大小  |         有符号范围         |   无符号范围   |      描述      |
| :-------: | :----: | :------------------------: | :------------: | :------------: |
|  tinyint  | 1bytes |         (-128,127)         |    (0,255)     |     Short      |
| smallint  | 2bytes |       (-32768,32767)       |   (0,65535)    |                |
| mediumint | 3bytes |     (-8388608,8388607)     |  (0,16777215)  |                |
|    int    | 4bytes |  (-2147483648,2147483647)  | (0,4294957295) |      Int       |
|  bigint   | 8bytes |      (-2^63, 2^63-1)       |   (0,2^64-1)   |      long      |
|   float   | 4bytes |                            |                | 单精度浮点数值 |
|  double   | 8bytes |                            |                | 双精度浮点数值 |
|  decimal  |        | 依赖于M(精度)和D(标度)的值 |                |     小数值     |

## 字符串类型

|    类型     |       大小       |            描述             |
| :---------: | :--------------: | :-------------------------: |
|  **char**   |  **0-255bytes**  |       **定长字符串**        |
| **varchar** | **0-65535bytes** |       **变长字符串**        |
|  tinyblob   |    0-255bytes    | 不超过255个字符的二进制数据 |
|  tintytext  |    0-255bytes    |        短文本字符串         |
|    blob     |   0-65535bytes   |      二进制长文本数据       |
|    text     |   0-65535bytes   |         长文本数据          |
| mediumblob  | 0-16777215bytes  |   二进制中等长度文本数据    |
| mediumtext  | 0-16777215bytes  |      中等长度文本数据       |
|  longblob   |                  |                             |
|  longtext   |                  |                             |

## 日期类型

|     类型     | 大小  |                    范围                    |          格式           |           描述           |
| :----------: | :---: | :----------------------------------------: | :---------------------: | :----------------------: |
|   **date**   | **3** |        **1000-01-01 至 9999-12-31**        |     **YYYY-MM-DD**      |        **日期值**        |
|   **time**   | **3** |        **-838:59:59 至 838:59:59**         |      **HH:MM:SS**       |   **时间值或持续时间**   |
|     year     |   1   |                1901至 2155                 |          YYYY           |          年分值          |
| **datetime** | **8** |   **1000-01-01 至 9999-12-31 23:59:59**    | **YYYY-MM-DD HH:MM:SS** |     混合日期和时间值     |
|  timestamp   |   4   | 1970-01-01 00:00:01 至 2038-01-19 03:14:07 |   YYYY-MM-DD HH:MM:SS   | 混合日期和时间值，时间戳 |

# 函数

## 聚合函数

```mysql
# 去重
select distinct {value} from {table_name}

## 聚合函数 将一列作为一个整体进行计算

# 统计总数
select count(*) from {table_name}

# 寻找最大值
select max({value}) from {table_name}

# 寻找最小值
select min({value}) from {table_name}

# 寻找平均值
select avg({value}) from {table_name}

# 统计和
select sum({value}) from {table_name}

## 分组查询
# 根据type分组，统计各种type的数量,再取大于10的根据count排序
# 执行顺序 where > 聚合函数 > having
# asc 升序 desc 倒序
select type, count(*) as count from holy_relic group by type having count > 10 order by count

# 分页查询 从第a个开始 共显示b个
select * from {table_name} limit a,b

# DQL 编写顺序
1 select
			字段列表  			
2 from 
			表名列表  			
3 where(分组之前过滤)
			条件列表  				
4 group by
			分组字段列表  		 
5 having(分组之后过滤)
			分组后条件列表 	 
6 order by
			排序字段列表 		  
7 limit
			分页参数 		

# 执行顺序
2 3 4 5 1 6 7
```

## 字符串函数

```mysql
## 字符串函数
# 字符串拼接
select concat("123", "456", ...)

# 大小写转换
select lower("Abc")
select upper("aBc")

# 字符串填充
# 左填充(字符串, 填充后的长度, 用于填充的字符)
# rpad 右填充
select lpad('1', '5', '0') => 00001

# 去除两侧空格
select trim(" a b c  ")

# 截取(从1开始，闭区间)
select substring("abcdefg", 1, 4)
```

## 数值函数

```mysql
## 数值函数
# 向上取整
select ceil(1.1) => 2

# 向下取整
select floor(1.9) => 1

# 取模
select mod(9,4) => 1

# 返回0-1内随机数
select rand()

# 求x的四舍五入的值，保留y位小数
round(x, y)

# 生成一个6位数的随机验证码
# 如果rand是0.x 则会只生成5位，需要左边补0
select lpad(round(rand() * 1000000, 0), 6, '0')
```

## 日期函数

```mysql
## 日期函数
select curdate() 2022-04-03
select curtime() 12:05:44
select now() 2022-04-03 12:05:44

# 2022-04-03
select day(now())    3
select month(now())  4
select year(now())   2022

# date_add 延后若干时间 2022-04-03 12:05:44
select date_add(now(), INTERVAL 5 day) => 2022-04-08 12:05:44

# 获取时间差值
select datediff("2021-10-11", "2021-10-08") => 3
```

## 流程函数

```mysql
## 流程函数
select if(true, '1', '2') => 1
select if(false, '1', '2') => 2

select ifnull('','default') => 
select ifnull('null','default') => null
select ifnull(null,'default') => default

# case when then else end
select name,
(case workspace when '上海' then '一线城市' when '北京' then '一线城市' else '二线城市' end) as '工作地点'
from emp
=> 
name   工作地点
  a    一线城市
  b    一线城市
  c    二线城市
 
select employee_id, case when employee_id % 2 = 0 then 0 else salary end as bonus
from Employees
```

## 窗口函数(8.0)

```mysql
### 一般用于求排名，前N高等

# 1.rank() over 排名相同的两名并列，但是占两个名次，1 1 3 4 4 6
# 2.dense_rank() over 排名相同的两名并列，但只占一个名次，1 1 2 3 4 4 5
# 3.row_number() over 不需要考虑是否并列，只进行连续排名
	
输入: 
Scores 表:
+----+-------+
| id | score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+

select
	score,
	dense_rank() over (order by score desc) as 'rank'
from
	score
	
输出: 
+-------+------+
| score | rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

```mysql
# 例2 求部门工资前三高的所有员工
Employee
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |
+--------------+---------+
Id是该表的主键列。
departmentId是Department表中ID的外键。
该表的每一行都表示员工的ID、姓名和工资。它还包含了他们部门的ID。
 
表: Department
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
Id是该表的主键列。
该表的每一行表示部门ID和部门名。

公司的主管们感兴趣的是公司每个部门中谁赚的钱最多。一个部门的<高收入者>是指一个员工的工资在该部门的<不同>工资中 排名前三 。

select
    d.name as Department,
		temp.name as Employee,
	  salary
from(
    select 
        name,
        salary,
        departmentId,
        dense_rank() over(partition by departmentId order by salary desc) as rk
    from 
        Employee
	) as temp
left join
	Department d
on 
	d.id = temp.departmentId
where
	temp.rk <= 3
```



# 约束

```mysql
## 约束
# check
check( age > 18 && age <= 35)

# 添加外键 保证数据的一致性和完整性
alter table emp add constraint fk_emp_dept_id foreign key(dept_id) references dept(id) 

# 删除外键
alter table drop foreign key fk_emp_dept_id
```

# 修改表结构

```mysql
# 添加字段
alter table {table_name} add {value} varchar(10) comment ''

# 修改字段
alter table {table_name} change {old_value} {new_value} varchar(10) comment ''

# 删除字段
alter table {table_name} drop {value}

# 修改表名
alter table {table_name} rename to {table_name2}

# 删除表
drop table {table_name}

# 删除指定表并重新创建
truncate table {table_name}
```



# 多表查询

![截屏2022-04-15 下午2.56.35](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.56.35.png)

![截屏2022-04-15 下午2.57.07](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.57.07.png)

```mysql
## 多表查询
# 笛卡尔积
select * from table1, table2
select * from table1 cross join table2
```

## 内连接

```mysql
## 内连接 查询两个表的交集
# 隐式内连接
select emp.name, dept.name from emp, dept WHERE dept_id = dept.id
# 显式内连接
select emp.name, dept.name from emp inner join dept on dept_id = dept.id
```

## 外连接

```mysql
## 外连接
# 左连接(左表及与右边的交集)
select emp.*, dept.name from emp left (outer) join dept on dept.id = dept_id
# 右连接
select emp.*, dept.name from emp left (outer) join dept on dept.id = dept_id
```

## 自连接

```mysql
## 自连接
# 查询各员工的领导
select e1.name as '员工', e2.name as '领导' from emp e1 left join emp e2 on e1.manager_id = e2.id
```

## 联合查询union

```mysql
## 联合查询 union 联合两个表的查询结果(类似java || )
# 字段与列数必须保持一致
# union(无重复) union all(有重复)
# 查询薪水小于5000或年龄大于50的员工信息
select * from emp where salary < 5000 union select * from emp where age > 50
```

## 子查询

```mysql
## 子查询
# 查询销售部和市场部所有的员工信息
# in
select * from emp WHERE dept_id in (select id from dept WHERE name in('销售部','市场部'))

# 查询比销售部所有员工工资都高的员工信息
# all
1.select * from emp WHERE salary > (select max(salary) from emp WHERE dept_id = (select id from dept where name = '销售部'))
 
2.select * from emp WHERE salary > all(select salary from emp WHERE dept_id = (select id from dept where name = '销售部'));

# 查询比销售部任意一个员工工资高的员工信息
# any
select * from emp WHERE salary > any(select salary from emp WHERE dept_id = (select id from dept where name = '销售部'));

# 查询与张无忌的工资及直属领导相同的员工信息
select * from emp where (salary, managerid) = (select salary, managerid from emp where name = '张无忌')

# 查询与鹿杖客，宋远桥的职位，薪水相同的员工信息
select * from emp where (job, salary) in (select job, salary from emp WHERE name = '鹿杖客' or name = '宋远桥')
```

```mysql
-- 1.查询员工的姓名 年龄 职位 部门信息
select emp.name, age, job, dept.name from emp, dept WHERE dept_id = dept.id

-- 2.查询年龄小于30岁的员工姓名 年龄 职位 部门信息
select emp.name, age, job, dept.name from emp, dept WHERE dept_id = dept.id and age < 30

-- 3.查询拥有员工的部门ID，部门名称
select distinct dept.id, dept.name from dept left join emp on dept.id = dept_id

-- 4.查询所有年龄大于40的员工及其归属的部门名称，如果没员工没有分配部门，也需要展示出来
select dept.name, temp.name, age from (select * from emp where age > 40) as temp left join dept on dept.id = dept_id;

select * from emp left join dept on dept.id = dept_id where age > 40

-- 5. *** 查询所有员工的工资等级
select e.name, salary, grade from emp as e, salgrade as s where e.salary >= s.losal and e.salary <= s.hisal

select e.name, salary, grade from emp as e, salgrade as s where e.salary between s.losal and s.hisal

-- 6.查询研发部所有员工的信息及工资等级
SELECT
	emp.*,
	grade 
FROM
	emp,
	salgrade 
WHERE
	dept_id = ( SELECT id FROM dept WHERE NAME = '研发部' ) 
	AND salary BETWEEN losal AND hisal

-- 7.查询研发部员工的平均工资
select avg(salary) from emp WHERE dept_id = (select id from dept where name = '研发部')

-- 8.查询公子比灭绝高的原因
select * from emp where salary > (select salary from emp where name = '灭绝')

-- 9.查询比平均薪资高的员工信息
select * from emp where salary > (select avg(salary) from emp)

-- 10. ***** 查询低于本部门平均工资的员工信息
select *, (select avg(e1.salary) from emp as e1 where e1.dept_id = e2.dept_id) as '平均'
from emp as e2 where e2.salary < (select avg(e1.salary) from emp as e1 where e1.dept_id = e2.dept_id)

-- 11.查询所有的部门信息，并统计部门的员工人数
select dept.name, count(*) from dept, emp where dept_id = dept.id group by dept.name

-- 12.查询所有学生的选课情况，展示出学生名称，学号，课程名称
select student.name, student.id, course.name from student, course, student_course 
where student.id = student_course.studentid
and course.id = student_course.courseid
```



# $\textcolor{orange}{事务操作}$

```mysql
## 事务操作
# 开启事务
start transaction

update account set money = money - 1000 where name = '张三';

update account set money = money + 1000 where name = '李四';

# 提交
COMMIT;

# 回滚
ROLLBACK;
```

## 事务四大特性(ACID)

### 1.原子性(Atomicity)

```mysql
# 事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
```

### 2.一致性(Consistency)

```mysql
# 事务完成时，必须使所有的数据都保持一致状态。
```

### 3.隔离性(Isolation)

```mysql
# 数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
```

### 4.持久性(Durability)

```mysql
# 事务一旦提交或回滚它对数据库中的数据的改变就是永久的。
```



## 如何保证ACID？

![截屏2023-03-10 下午3.04.54](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-10 下午3.04.54.png)**==Redo Log是记录数据库的事务操作，用来保证事务的持久性(Durability)==**。当一个事务进行提交时，Redo Log会记录下所有被修改的数据，并将这些记录写入到磁盘上的Redo Log文件中。这样，在数据库崩溃或断电等异常情况下，可以通过Redo Log恢复数据，从而保证数据的一致性和持久性。

**==Undo Log则是记录数据库事务操作的逆操作==**，用来保证事务的原子性(Atomicity)和一致性(Consistency)。当一个事务进行回滚时，Undo Log会记录下所有被修改的数据的原始值，并将这些记录写入到磁盘上的Undo Log文件中。这样，在事务回滚时，可以通过Undo Log将数据恢复到事务开始之前的状态，从而保证事务的原子性和一致性。



## 并发事务问题

### 1.脏读

- 一个事务读到了另外一个事务还没有提交的数据
- 隔离级别:读未提交(Read uncommitted)
  - A事务更新salary为2000但还未提交，此时B事务直接读取了2000并进行了其他操作。但A事务随后故障，salary回滚为1000，可B事务已经进行了salary为2000的操作。



### 2.不可重复读

- **在一个事务内多次读取同一个数据，如果出现前后两次读到的数据不一样的情况，就意味着发生了「不可重复读」现象。**

- 隔离级别:读已提交(Read Committed)

  - 假设有 A 和 B 这两个事务同时在处理，事务 A 先开始从数据库中读取小林的余额数据，然后继续执行代码逻辑处理，**在这过程中如果事务 B 更新了这条数据，并提交了事务，那么当事务 A 再次读取该数据时，就会发现前后两次读到的数据是不一致的，这种现象就被称为不可重复读。**

    ![图片](https://cdn.xiaolincoding.com//mysql/other/f5b4f8f0c0adcf044b34c1f300a95abf.png)



### 3.幻读

- **在一个事务内多次查询某个符合查询条件的「记录数量」，如果出现前后两次查询到的记录数量不一样的情况，就意味着发生了「幻读」现象。**

-  隔离级别:可重复读(Repeatable-read)

  - 假设有 A 和 B 这两个事务同时在处理，事务 A 先开始从数据库查询账户余额大于 100 万的记录，发现共有 5 条，然后事务 B 也按相同的搜索条件也是查询出了 5 条记录。

    ![图片](https://cdn.xiaolincoding.com//mysql/other/d19a1019dc35dfe8cfe7fbff8cd97e31.png)

    接下来，事务 A 插入了一条余额超过 100 万的账号，并提交了事务，此时数据库超过 100 万余额的账号个数就变为 6。

    然后事务 B 再次查询账户余额大于 100 万的记录，此时查询到的记录数量有 6 条，**发现和前一次读到的记录数量不一样了，就感觉发生了幻觉一样，这种现象就被称为幻读。**

    

## 事务隔离级别

### 1.读未提交(Read Uncommitted)

```mysql
# 即可以读到其他事务未提交的数据，但无法保证读到的数据一定是最后提交的数据。

# *** 若中间发生回滚，就会出现脏数据。
```

### 2.读已提交(Read Committed)

```mysql
# 一个事务只能读到其他事务已经提交过的数据，即commit之后的数据

# *** 不同时刻事务的同一操作可能会得出不一样的结果
```

### 3.可重复读(Repeatable-read)

```mysql
# 事务不会读到其他事务对已有数据的修改，即事务开始时读到的数据是什么，在提交前也是什么

# 事务A，B同时开始。
A普通查询返回salary = 1000
B修改了salary = 2000但未提交
A普通查询返回salary = 1000
B提交
A普通查询返回salary = 1000
```

### 4.串行化

```mysql
# 读的时候加共享锁，允许其他事务并发读，但不能写。
# 写的时候加排他锁，其他事务不能读也不能写。
```



```mysql
select @@transaction_isolation;

set global transaction isolation level repeatable read;
```



# 存储引擎

## InnoDB

```mysql
## 存储引擎
# InnoDB
特点：
1、DML操作遵循ACID模型，支持事务
2、行锁，提高并发访问性能
3、支持外键约束，保证数据的完整性和正确性

MySql默认存储引擎，支持事务和外键，适用于
1.对事务的完整性具有较高要求，并发条件下要求数据的一致性
2.更新，删除操作多。
```

## MyISAM

```mysql
# MyISAM(MongoDB替代)
1、不支持事务，不支持外键
2、支持表锁，不支持行锁
3、访问速度快

MyISAM适用于
1.以读和插入操作为主，更新和删除操作少
2.对事务的完整性，并发性要求不是很高
```

## Memory

```mysql
# Memory(redis替代)
1、内存存放
2、哈希索引

将数据保存在内存中，访问速度快。
受到硬件或断电问题的影响，只能作为临时表或缓存使用。
对表的大小有限制，太大的表无法缓存在内存中，无法保证数据的安全性。
```

## InnoDB与MyISAM对比

|     特点     |   InnoDB    | MyISAM |
| :----------: | :---------: | :----: |
|   存储限制   |    64TB     |   有   |
| **事务安全** |  **支持**   |   -    |
|    锁机制    |    行锁     |  表锁  |
|  B+tree索引  |    支持     |  支持  |
|   Hash索引   |    支持     |  支持  |
|   全文索引   | 支持(5.6后) |  支持  |
|   空间使用   |     高      |   低   |
|   内存使用   |     高      |   低   |
| 批量插入速度 |     低      |   高   |
| **支持外键** |  **支持**   |   -    |



# $\textcolor{orange}{索引}$

```mysql
## 索引
```

| 优势                                                        | 劣势                                                   |
| ----------------------------------------------------------- | ------------------------------------------------------ |
| 提高数据检索的效率，降低数据库的IO成本                      | 索引列占用空间                                         |
| 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗 | 大大提高查询效率，但降低了更新表的速度，增删改效率降低 |

## 索引分类

```mysql
## 索引分类
```

|   分类   |                         含义                         |           特点           |  关键字  |
| :------: | :--------------------------------------------------: | :----------------------: | :------: |
| 主键索引 |               针对于表中主键创建的索引               | 默认自动创建，只能有一个 | primary  |
| 唯一索引 |            避免同一个表中数据列中的值重复            |        可以有多个        |  unique  |
| 常规索引 |                   快速定位特定数据                   |        可以有多个        |          |
| 全文索引 | 全文索引查找的是文本中的关键词，而不是比较索引中的值 |        可以有多个        | fulltext |

```mysql
## 索引分类
```

|           分类            |                            含义                            |       特点       |
| :-----------------------: | :--------------------------------------------------------: | :--------------: |
| 聚簇索引(Clustered index) | 将数据存储与索引放到了一块，索引结构的叶子结点保存了行数据 | 必须有且只有一个 |
| 二级索引(Secondary Index) | 将数据与索引分开存储，索引结构的叶子结点关联的是对应的主键 |   可以存在多个   |

## B-树和B+树

```mysql
## B-树
https://www.cs.usfca.edu/~galles/visualization/BTree.html
4阶B-树
```

![截屏2022-04-07 下午8.08.02](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-07 下午8.08.02.png)

```mysql
## B+树
https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html
4阶B+树
```

![截屏2022-04-07 下午8.08.42](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-07 下午8.08.42.png)



```mysql
1.为什么InnDB存储引擎选择使用B+tree索引结构？
相对于二叉树，B+树层级更少，搜索效率高。
相对于B-树，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低。
B+树的叶子节点具有双向链表，方便搜索和排序
```

![截屏2022-04-08 下午4.22.22](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-08 下午4.22.22.png)

## 查询索引

```mysql
# 查询索引
show index from {table_name}
```

## 创建索引

```mysql
# 创建索引
# name字段为姓名字段,该字段的值可能会重复，为该字段创建索引
create index idx_user_name on tb_user(name);
# 索引name的前n个字符
create index idx_user_name on tb_user(name(n));

# phone手机号字段的值，是非空且唯一的，为该字段创建唯一索引
create unique index idx_user_phone on tb_user(phone)

# 为profession、age、status创建联合索引
create index idx_user_pro_age_sta on tb_user(profession, age, status)
```

## 删除索引

```mysql
# 删除索引
drop index idx_user_email on tb_user
```

## 最左前缀法则

```mysql
## ****** 索引使用
# 最左索引法则(联合索引)
# 查询从索引的最左列开始，不跳过索引中的列(与位置顺序无关)。如果跳过某一列，索引将部分失效(即后面的字段索引失效)
explain select * from tb_user where profession = '软件工程' and status = '0' and age = 31;
```

## 索引失效情况

```mysql
# 联合索引中，出现范围查询>或<，范围查询右侧的索引失效。而>=,<=不失效，因此最好选用=。
# *** 索引失效 
# 在索引列上进行运算操作，会导致索引失效。
# 字符串类型字段不加引号，会导致索引失效。
# 头部模糊匹配，会导致索引失效=>全表扫描。
# or两侧必须都有索引，索引才生效，否则失效。
# 如果MySQL评估使用索引比使用全表搜索更慢，则不使用索引。(如全表扫描)
```

## SQL提示

```mysql
## SQL提示
# 若一个字段有单独索引，又有联合索引，sql可能会评估使用联合索引
# 此时可以使用SQL提示来
# use 建议
# ignore 忽略
# force 强制使用
explain select * from tb_user use index(idx_user_pro) where profession = '软件工程'
```

## 索引覆盖&回表

![截屏2022-04-12 下午7.29.56](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-12 下午7.29.56.png)

```mysql
## 索引覆盖
覆盖索引不需要回表
尽量少使用select *, 较大可能出现回表查询

# 该条语句中，对username,password建立联合索引，效率最高
# 二级索引下面挂的是id，因此不需要联合id
select id, username, password from tb_user where username = 'test'
```

## 前缀索引

```mysql
## 前缀索引(避免当前字段过长)
# 取前n的字符作为索引
create index id_xxx fon table(column(n))

# 前缀长度
# 根据索引的选择性来确定，指不重复的索引值(基数)和数据表的总记录数，索引选择性越高则查询效率越高，唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的
select count(dinstinct email) / count(*) from table
```

## 索引设计原则

```mysql
## 索引设计原则
1.针对数据量较大，且查询比较频繁的表建立索引。
2.针对于常作为查询条件(where)、排序(order by)、分组(group by)操作的字段建立索引。
3.尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4.如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5.尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
6.要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改查的效率。
7.如果索隐裂不能存储NULL值，在创建表时用not null约束它，当优化器知道每列是否包含NULL时，它可以更好的确定那个索引最有效的用于查询。
```



# SQL优化

```mysql
## SQL优化
```

## 性能分析

```mysql
# 查询sql语句的执行频次(7个_)
show global status like 'Com_______'
```

![截屏2022-04-08 下午4.24.45](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-08 下午4.24.45.png)

### 慢查询日志

```mysql
# 开启慢查询日志开关
set global slow_query_log = 'on'
# 设置超过1s就记录
set global long_query_time = 1
```

### show profiles

```mysql
# 查询是否支持profile
select @@have_profiling
# 查询profile是否打开
select @@profiling 
```

### explain

|  id  | select_type | table | type | possible_keys | key  | key_len | ref  | Rows | Extra |
| :--: | :---------: | :---: | ---- | ------------- | ---- | ------- | ---- | ---- | ----- |

#### 1.id

```mysql
# id （select查询的序列号，表示查询中执行select子句或者是操作表的顺序）
1.id相同，顺序执行
2.id不同，值越大余越先执行
```

#### 2.select_type

| 序号 |        列名        |                            含义                            |
| :--: | :----------------: | :--------------------------------------------------------: |
|  1   |       SIMPLE       |       简单的select语句(不包括UNION操作或子查询操作)        |
|  2   |      PRIMARY       |                    查询中最外层的select                    |
|  3   |       UNION        | UNION操作中，处于内层的select(与外层的select没有依赖关系)  |
|  4   |  DEPENDENT UNION   |  UNION操作中，处于内层的select(与外层的select有依赖关系)   |
|  5   |    UNOIN RESULT    |                        UNION的结果                         |
|  6   |      SUBQUERY      |                   子查询中的第一个select                   |
|  7   | DEPENDENT SUBQUERY |          子查询中的第一个select，但依赖于外层的表          |
|  8   |      DERIVED       |          被驱动的select子查询(子查询位于from子句)          |
|  9   |    MATERIALIZED    |                       被物化的子查询                       |
|  10  |    UNCACHEABLE     | 对于外层的主表，子查询不可被物化，每次都需要计算(耗时操作) |
|  11  | UNCAHCEABLE UNION  |           UNION操作中，内层的不可被物化的子查询            |

#### 3.table

```mysql
# 当前表名
```

#### 4.partitions

```mysql
# 匹配的分区
```

#### 5.type

```mysql
# ***
表示连接类型。
性能由好到差依次为
```

|  id  |   列名   |                             含义                             |
| :--: | :------: | :----------------------------------------------------------: |
|  1   |  system  |                         表中只有一行                         |
|  2   |  const   |           单表中最多一个匹配行，pk或者unique index           |
|  3   |  eq_ref  |        多表连接中被驱动表的连接列上有pk或unique index        |
|  4   |   ref    | 与eq_ref类似，但使用的是普通索引，也可以是单表上的non-unique索引 |
|  5   | fulltext |                   使用fulltext索引执行连接                   |
|  6   |  range   | 单表索引中的范围查询，使用索引查询出单个表的一些行数据，ref列会变为null |
|  7   |  index   |                             索引                             |
|  8   |   all    |                           全表扫描                           |

#### 6.possible_keys

```mysql
# 显示可能应用在这张表上的索引，一个或多个。
```

#### 7.key

```mysql
# 实际使用的索引，如果为NULL，则没有索引。
```

#### 8.key_len

```mysql
# 表示索引中使用的字节数，为索引字段的最大可能长度。在不损失精度的情况下越短越好。
```

#### 9.rows

```mysql
# Mysql预估的查询行数。
```

#### 10.filtered

```mysql
# 查询返回的行数占总行数的百分比。
```

#### 11.extra

```mysql
using where

using temporary
使用了临时表

using index condition 
查找使用了索引，但是需要回表查询数据

using where；using index 
查找使用了索引，但是需要的数据都在索引列中能查到，所以不需要回表查询数据 

using filesort
使用filesort来进行order by排序
```

## 插入优化

### 1.批量插入(小于1000条)

```mysql
insert into table values(1, 'a'),(2, 'b'),(3, 'c')
```

### 2.手动提交事务

```mysql
start transaction;
insert...
insert...
commit;
```

### 3.主键顺序插入

```mysql
1 2 3 4 .... 15 21 ...
```

### 4.大批量插入load

```mysql
# 大批量插入使用insert性能较低，应使用load指令
set global local_infile = 1;
load data local infile {file_path} into table {table_name} fields terminated by ',' lines terminated by '\n'
```

## 主键优化

```mysql
## 主键优化
```

![截屏2022-04-15 下午2.33.28](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.33.28.png)

### 页分裂

```mysql
# 页分裂
页可以为空，也可以填充50%，100%。
每个页包含2-N行数据，根据主键排列。
```

![截屏2022-04-15 下午2.32.17](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.32.17.png)

![截屏2022-04-15 下午2.32.35](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.32.35.png)

![截屏2022-04-15 下午2.32.53](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.32.53.png)

### 页合并

```mysql
# 页合并
当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记为(flaged)删除并且它的空间变得允许被其他记录声明使用
当页中删除的记录达到MERGE_THRESHOLD(默认为页的50%)，InnoDB会开始寻找最靠近的页(前或后)看看时是否可以将两个页合并以优化使用空间
```

![截屏2022-04-15 下午2.35.26](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.35.26.png)

![截屏2022-04-15 下午2.36.21](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午2.36.21.png)

### 主键设计原则

```mysql
# 主键设计原则
1、满足业务需求的情况下，尽量降低主键的长度
2、插入数据时，尽量顺序插入，使用AUTO_INCREMENT
3、尽量不要使用UUID或身份证等作主键
4、避免对主键的修改
```

## order by优化

```mysql
# using filesort
通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫FileSort排序。

# using index
通过有序索引顺序扫描直接返回有序数据，不需要额外排序，操作效率高。
```

```mysql
# 创建联合索引(默认均为升序)
create index idx_user_age_phone on user(age, phone)

explain select id, age, phone from user order by age, phone
=> Using index

explain select id, age, phone from user order by age desc, phone desc
=> Backward index scan; Using index

explain select id, age, phone from user order by age, phone desc
=> Using index; Using filesort
```

```mysql
# 创建联合索引(默认均为升序)
create index idx_user_age_phone on user(age desc, phone)

explain select id, age, phone from user order by age, phone
=> Using index; Using filesort

explain select id, age, phone from user order by age desc, phone
=> Using index
```

```mysql
1、根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则
2、尽量使用覆盖索引
3、多字段排序，一个升序，一个降序，此时需要注意联合索引在创建时的规则(ASC/DESC)
4、如果补课避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小sort_buffer_size(默认256k)

show variables like 'sort_buffer_size' 
```

![截屏2022-04-15 下午4.53.36](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-15 下午4.53.36.png)

## group by优化

```mysql
create index idx_user_pro_age_sta on tb_user(profession, age, status)

explain select profession, count(*) from tb_user group by profession;
=> using index;

explain select age, count(*) from tb_user group by age;
=> using index; using temporary

explain select profession, count(*) from tb_user group by profession, age;
=> using index;

explain select profession, count(*) from tb_user where profession = '软件工程' group by age;
=> using index;
```

## limit优化

```mysql
# limit 2000000,10 此时需要MySQL排序前2000010条记录，但仅仅返回10条，其他记录丢弃，查询排序的代价非常大。
select * from tb_sku limit 2000000,10;

# 优化思路: 一般分页查询时，通过创建覆盖索引能够比较好的提高性能，可以通过覆盖索引加子查询形式进行优化。
select s.* from tb_sku s, (select id from tb_sku order by id limit 2000000,10) as a where s.id = a.id
```

## count优化

```mysql
# MyISAM引擎把一个表的总行数存在了磁盘上，因此执行count(*)的时候会直接返回这个数，效率很高
# InnoDB执行count(*)需要一行一行的把数据读出来然后累计技术，因此可以用redis计数

# count是一个聚合函数，对于返回的结果集，一行行的判断，***如果count函数的参数不是NULL***，累计值就加1，否则不加，最后返回累计值。

# 效率 count(*) > count(主键id) > count(1) ≈ count(字段) 
```

## update优化

```mysql
# InnoDB的行锁是针对索引加的锁，不是针对记录加的锁。
# 因此更新时最好根据索引(且索引不能失效)更新，否则行锁会升级为表锁，影响并发性能。
```



# 视图

## 创建视图

```mysql
create (or replace) view stu as select id, name from student where id <= 10;
```

## 查询视图

```mysql
# 查询创建视图的语句
show create view stu
# 查看视图数据
select * from stu
```

## 修改视图

```mysql
# 方法1
create or replace view stu as select id, name, no from student where id <= 10;

# 方法2 
alter view stu as select id, name, no from student where id <= 10;
```

## 删除视图

```mysql
drop view (if exists) stu
```

## 检查选项

### cascaded(默认)

```mysql
# 检查所有的视图
with cascaded check option 
```

### local

```mysql
# 只检查将要更新的视图本身
with local check option 
```

## 视图的更新

```mysql
# 要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系
# 如果视图包含一下任何一项，则该视图不可更新
1.聚合函数或窗口函数
2.distinct
3.group by
4.having
5.union
```

## 视图的作用

```mysql
# 1.简单
视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图。从而是的用户不必为以后的操作每次指定全部的条件。
# 2.安全
数据库可以授权，但不能授权到数据库特定行和特定列上。通过视图用户只能查询和修改他们所能见到的数据。
# 3.数据独立
视图可以帮助用户屏蔽真实表结构变化带来的影响。
```



# 触发器

## 创建触发器

```mysql
create trigger {trigger_name}
before/after insert/update/delete
on {table_name} for each row
begin
	...
end;
```

```mysql
-- 如果插入的money>1200，则将money改为1200
create trigger trigger_modify
before insert on account
for each row
begin
-- 	insert into account(id, name, money) values(
-- 		new.id,
-- 		new.name,
		set new.money = if(new.money>1200, 1200, new.money);
-- 		);
end
```



## 查看触发器

```mysql
show triggers
```

## 删除触发器

```mysql
drop trigger {schema_name} trigger name
# 如果没有指定schema_name，默认为当前数据库
```



# $\textcolor{orange}{锁}$

## 全局锁

```mysql
# 对整个数据库实例加锁，加锁后数据库处于只读状态
# DML，DDL，事务的更新操作都将被阻塞

# 典型场景: 全库的数据备份  
flush tables with read lock;
mysqldump -uroot -ppassword database > database.sql;
unlock tables;

# 特点
1.如果在主库上备份，那么在备份期间都不能执行更新，业务停摆
2.如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志，会导致主从延迟

# --single-transaction可以完成不加锁的一致性数据备份
mysqldump --single-transaction -uroot -ppassword database > database.sql

```

## 表级锁

### 1.表锁

```mysql
## 表共享读锁(read lock)
#  不阻塞其他客户端的读操作，但会阻塞其他的操作

# 加锁
lock tables {table_name} read

# 释放锁
1.unlock tables
2.客户端断开连接
```

![截屏2022-04-19 下午3.41.38](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-19 下午3.41.38.png)

```mysql
# 表独占写锁(write lock)
# 仅当前客户端可以读写

# 加锁
lock tables {table_name} write

# 释放锁
1.unlock tables
2.客户端断开连接
```

### 2.元数据锁(meta data lock, MDL)

```mysql
# MDL加锁过程是系统自动控制，无需显示使用，在访问一张表时会自动加上。
# MDL锁主要作用是维护表元数据的一致性，在表上有活动事务的时候，不可以对元数据进行写操作。
# 为避免DML与DDL冲突，保证读写的正确性。

# 当对一张表进行增删改查的时候，加MDL读锁(共享)
# 当对表结构进行变更操作的时候，加MDL写锁(排他)
```

![截屏2022-04-19 下午4.50.06](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-19 下午4.50.06.png)

### 3.意向锁(Intention Locks)

避免DML在执行时，加的行锁与表级锁冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查。

```mysql
# *** 不与行级锁冲突的表级锁
# 意向锁之间不互斥

# 意向共享锁(IS): 与表锁共享锁(read)兼容，与表锁排他锁(write)互斥。
select ... lock in share mode

# 意向排他锁(IX): 与表锁共享锁(read)及排他锁(write)都互斥。意向锁之间不会互斥。
select ... for update

# 意向锁是有数据引擎自己维护的，用户无法手动操作意向锁。
```

![截屏2022-04-19 下午8.38.48](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-19 下午8.38.48.png)

![截屏2022-04-19 下午8.39.08](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-19 下午8.39.08.png)



## 行级锁

```mysql
# InnoDB的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加锁。
```

### 1.行锁(Recode Lock)

```mysql
# 锁定单个行记录的锁，防止其他事务对此进行update和delete。在RC,RR隔离级别下都支持。
# 即共享锁之间不互斥，共享锁与排他锁互斥。

# 共享锁(S)
允许一个事务区读一行，阻止其他事务获得相同数据的排他锁。

# 排他锁(X)
允许获取排他锁的事务更新数据，阻止其他事务获得相同数据的共享锁和排他锁。
```

![截屏2022-04-20 下午3.02.17](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-20 下午3.02.17.png)

|              SQL              |  行锁类型  |   说明   |
| :---------------------------: | :--------: | :------: |
|            insert             |   排他锁   | 自动加锁 |
|            update             |   排他锁   | 自动加锁 |
|            delete             |   排他锁   | 自动加锁 |
|            select             | 不加任何锁 |          |
| select ... lock in share mode |   共享锁   |          |
|     select... for update      |   排他锁   |          |

```mysql
1.针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。
2.InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据，那么InnoDB将表中所有的记录加锁，此时就会升级为表锁。
```

### 2.间隙锁(Gap Lock)

```mysql
# 锁定索引记录间隙(不含该记录)，确保索引记录间隙不变，防止其他事务再这个间隙进行insert，产生幻读。在RR隔离级别下都支持。
```

```mysql
# 1.索引上的等值查询(唯一索引),给不存在的记录加锁时，优化为间隙锁
begin;
update stu set name = '5' where id = 6; => 在id=4和id=8之间加了间隙锁
insert into stu values(5, 10, '5') => 因间隙锁而阻塞
```

![截屏2022-04-20 下午4.59.37](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-20 下午4.59.37.png)

```mysql
# 2.索引上的等值查询(普通索引),向右遍历时最后一个值不满足查询需求时，next-key lock退化为间隙锁
select * from stu where age = '13' lock in share mode;

select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;

# 表中共有2个age=13的行
???
```

![截屏2022-04-20 下午8.17.31](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-20 下午8.17.31.png)

![截屏2022-04-20 下午8.18.30](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-20 下午8.18.30.png)

```mysql
3.索引上的范围查询(唯一索引),会访问到不满足条件的第一个值为止

# 间隙锁的唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务再同一个间隙上采用间隙锁。

???
```

### 3.临键锁(Next-Key Lock)

```mysql
# 默认情况下 InnoDB会在RR隔离级别下运行，InnoDB使用next-key进行搜索和扫描，防止幻读。
# 行锁和间隙锁的组合，同时锁住数据，并锁住数据前面的间隙Gap。在RR隔离级别下支持。
```



# 二进制日志

- 二进制日志记录整个事务的sql语句，而事务日志只记录数据修改的操作
- 二进制日志记录的是逻辑信息，事务日志记录的是物理信息
- 二进制日志可以用于数据库复制和恢复，而事务日志只能用于恢复
- 二进制日志中的日志条目可以在不同的Mysql版本间共享

## 1.查看日志

```mysql
# 获取binlog文件列表
show binary logs

# 获取当前正在写入的binlog文件
show master status\G

# 恢复数据的操作也会写入日志，因此最好切换新的日志
flush logs

# 需要在/usr/local/mysql/data下查看
# 根据时间查看
mysqlbinlog -v --base64-output=DECODE-ROWs 'binlog.000209'

-d 指定数据库名称
-o 忽略掉日志中前n行命令
-v 将行事件重构为sql语句
-w 将行事件重构为sql语句，并输出注释信息

# 根据节点查看
show binlog events in 'binlog.000209' (from {pos})

# Query事件 负责开始一个事务
# Table_map事件 负责映射需要的标
# Update_rows事件 负责写入数据
# Xid事件 负责结束事务
```

## 2.数据恢复

### 1.根据节点恢复

```mysql
# --start-position=开始节点
# --stop-position=结束节点
# -f 忽略error
# -v 使得二进制可读为row
sudo mysqlbinlog --start-position=1396 --stop-position=1299856 --database=holy_relic  binlog.000209 | mysql -uroot -phmz990203 -v holy_relic -f
```

### 2.根据时间恢复

```
sudo mysqlbinlog --start-datetime="2022-05-16 22:00:00" --stop-datetime="2022-05-16 23:59:00" --database=holy_relic binlog.000209 | mysql -uroot -phmz990203 -v holy_relic -f
```



# $\textcolor{orange}{MVCC}$

## 1.当前读(<font color=red>悲观锁</font>)

- 读取最新版本的记录
  - select ... lock in share mode;(共享锁)
  - select ... for update(排他锁)
  - update,insert,delete(排他锁)

```mysql
# eg:
在RR隔离级别下，事务A，B同时开始。
A普通查询返回salary = 1000
B修改了salary = 2000但未提交
A普通查询返回salary = 1000
B提交
A普通查询返回salary = 1000
A进行当前读 select ... lock in share mode; => 返回salary = 2000
A提交
```

## 2.快照读(<font color=red>乐观锁</font>)

- 简单的select(不加锁)就是快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。
  - Read committed: 每次select，都生成一个快照读
  - Repeatable Read: <font color=red>开启事务后第一个select语句才是快照读的地方</font>
  - Serializable: 快照读会退化为当前读



## 3.MVCC

**数据库并发场景有三种，分别为：**

- `读-读`：不存在任何问题，也不需要并发控制
- `读-写`：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，幻读，不可重复读
- `写-写`：有线程安全问题，可能会存在更新丢失问题。

MVCC是一种用来解决<font color=red>读-写</font>冲突的**无锁并发控制**



#### 实现原理

具体实现依赖于数据库记录中的`三个隐式字段、undolog、readView`

#### 1.隐藏字段

| 隐藏字段    | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| DB_TRX_JD   | 最近修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID |
| DB_ROLL_PTR | 回滚指针，指向这条记录的上一个版本，配合undo log             |
| DB_ROW_ID   | 隐藏主键，如果表结构没有指定逐渐，将会生成该隐藏字段         |

#### 2.undo log

记录数据库事务操作的逆操作

回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志。

insert时，产生的undo log日志只在回滚时需要，在事务提交后，可立即被删除。

而update、delete时，产生的undo log日志不仅在回滚时需要，在快照读里也需要，不会被立即删除。

#### 3.readView(读视图)

是快照度SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务。

| 字段           | 含义                                           |
| -------------- | ---------------------------------------------- |
| m_ids          | 当前活跃的事务ID集合                           |
| min_trx_id     | 最小活跃事务ID                                 |
| max_trx_id     | 预分配事务ID，当前最大事务ID+1(事务id是自增的) |
| creator_trx_id | ReadView创建者的事务IDMM                       |



# 权限管理

```mysql
-uroot 				用户名
-phmz990203   密码
-h{hostname}  指定服务器ip或域名
-p{port}	    指定端口
-e{}					指定sql语句并退出

# -e选项可以在Mysql客户端执行sql语句，方便一些批处理脚本
mysql -uroot -phmz990203 -e 'select * froms stu';

## 管理用户
# 只能够在当前主机localhost装备
create user 'hmz'@'localhost' identified by '123456'

# 可以在任意主机访问该数据库
create user 'hmz'@'%' identified by '123456'

# 修改用户密码
alter user 'hmz'@'localhost' identified with mysql_native_password by '654321'

# 删除用户
drop user 'hmz'@'localhost'

# 查询权限
show grants for 'hmz'@'localhost'

# 授予权限
grant all on {table}.* to 'hmz'@'localhost'

# 撤销权限
revoke all on {table}.* from 'hmz'@'localhost'
```



# 主从复制

```mysql
# 特点
1.主库出现问题，可以快速切换到从库提供服务
2.实现读写分离，降低主库的访问压力
3.可以在从库中执行备份，避免备份期间影响主库服务
```

```mysql
# 原理
1.Master主库在事务提交时，会吧数据变更记录在二进制日志文件Binlog中
2.从库读取主库的二进制日志文件Binlog,写入到从库的中继日志Relay Log
3.slave重做中继日志中的事件，将改变反应它自己的数据
```

## 主库配置

```mysql
cd /etc/mysql/mysql.conf.d

vim mysqld.cnf
```

```mysql
# mysql服务ID，保证整个集群环境中唯一，取值范围:1-2^32-1,默认为1
server-id = 1

# 是否只读，1代表只读，0代表读写
read-only = 0

# 忽略的数据，指不需要同步的数据库
# binlog-ignore-db = mysql

# 指定同步的数据库
# binlog-do-db = {db_name}
```

```mysql
# 重启mysql(可能是别的名称,如mysqld)
systemctl restart mysql
```

```mysql
# % 允许任意主机访问该数据库
create user 'hmz'@'%' identified by 'hmz990203'

# 分配主从复制权限
grant replication slave on *.* to 'hmz'@'%'
```

```mysql
# 通过指令，查看二进制日志坐标
show master status
```

![截屏2022-04-26 下午4.01.03](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-04-26 下午4.01.03.png)

|       File       | 从哪个日志文件开始推送日志 |
| :--------------: | :------------------------: |
|     position     |   从哪个位置开始推送日志   |
| binlog_ignore_db |   指定不需要同步的数据库   |

## 从库配置

```
cd /etc/mysql/mysql.conf.d

vim mysqld.cnf
```

```mysql
# mysql服务ID，保证整个集群环境中唯一，取值范围:1-2^32-1,默认为1
server-id = 2

# 是否只读，1代表只读，0代表读写
read-only = 1
```

```mysql
# 重启mysql(可能是别的名称,如mysqld)
systemctl restart mysql
```

```mysql
# 从库配置 mysql 8.0.23及以后的语法
mysql > change replication source to source_host = '10.22.191.78', source_user = 'hmz', source_password = 'hmz990203', source_log_file='mysql-bin.000003', source_log_pos = 944;
```

|     参数名      |        含义        |
| :-------------: | :----------------: |
|   source_host   |     主库ip地址     |
|   source_user   |  连接主库的用户名  |
| source_password |   连接主库的密码   |
| source_log_file |  binlog日志文件名  |
| source_log_pos  | binlog日志文件位置 |

```mysql
# 开启同步操作
start replica (8.0.22之后)

start slave (8.0.22之前)
```

```mysql
# 查询情况
show replica status\G;
```

```
openssl s_client -connect 10.22.191.138:3306 --ssl-cert=server-cert.pem --ssl-key=server-key.pem

```

```
mysql -u username -p --ssl-ca=server-cert.pem --ssl-cert=client-cert.pem --ssl-key=client-key.pem --ssl-mode=VERIFY_CA

```

```
CHANGE MASTER TO MASTER_SSL=1,
MASTER_SSL_CA='/etc/mysql/ssl/server-cert.pem',
MASTER_SSL_CERT='/etc/mysql/ssl/server-cert.pem',
MASTER_SSL_KEY='/etc/mysql/ssl/server-key.pem',
MASTER_HOST='10.22.191.78',
MASTER_USER='hmz',
MASTER_PASSWORD='hmz990203',
MASTER_LOG_FILE='mysql-bin.000003',
MASTER_LOG_POS=944;
```


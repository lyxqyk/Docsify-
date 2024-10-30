> # select的查询语句

### 简单查询

select 字段 from 表名

查询一个表中某字段的所以值

### 条件查询

select 字段 from 表名 where 条件

查询满足条件的所有记录

### 模糊查询

注意！！！   使用模糊查询的时候使用like不用‘  =  ’；

select 字段  from 表名 where 字段 like ‘_数据%’；

### 算数运算符

常见的有 > 	< 	= 	!= 	<=  >=

### 逻辑运算符

可以将查询的单个条件改为多个条件或者满足多个条件其中的一个

and   or  not

这个not 配like使用有奇效

例如找出女人表中所有不姓王的人

select  name from  women where name not like "王%";

in与not in运算符

select 字段 from 表名 where 字段 in (列名)

### 高级查询

#### 范围运算

between...and...,

select 字段 from 表名 where 字段 between 范围1 and 范围2;

#### 限制查询

limit

select 字段 from 表明 limit n,m;

limit可以强制指定查询结果的记录条数.

n是开始记录行,0表示第一条记录

m表示显示行,从n开始,总共显示几行记录.

#### 嵌套查询

select name  from women where sno=(select sno from...);

#### 多表连查

全称多表连接查询,和嵌套查询一样,都需要有一个共同的字段,然后将多个表连接在一起查询,将符合条件的组成一个合集.

#### 内连接

inner join ..on..通常位于表名后面

select  字段  from 表1 inner join 表2 on 表1.字段=表2.字段;

on后面是连接条件,也就是表1和表2共有的字段

举个栗子

```
INSERT INTO students (student_name, sex, age) VALUES
('张三', '男', 18),
('李四', '女', 19),
('王五', '男', 20);
INSERT INTO teachers (teacher_name, department) VALUES
('张老师', '数学组'),
('李老师', '语文组');
INSERT INTO courses (course_name, teacher_id) VALUES
('数学', 1),
('语文', 2);
INSERT INTO scores (student_id, course_id, score) VALUES
(1, 1, 85.50),
(2, 2, 90.00),
(3, 1, 78.20);
select student_name,course_name,score from students
inner join scores on students.student_id=scores.student_id
inner join courses on courses.course_id=scores.course_id;
```

#### 左连接

left join ..on 位于表名后面

select 字段 from 表1 left join 表2 on 连接条件

左连接是左边为主表,指定字段都显示,右边为从表,没内容显示null.

使用左连接将两个表连接到一起

举个栗子

```
select student_name,course_name,score from students
left join scores on students.student_id=scores.student_id
left join courses on courses.course_id=scores.course_id;
```



#### 右连接

right join...on位于表后面

select 字段 from 表1 right join 表2 on 连接条件



内连接和左右连接的区别是内连接值显示两者匹配的 左右就是按一个主表匹配匹配不到就显示为空

### 聚合函数

#### 最小值 min（）

elect min(字段) from 表名；

#### 最大值max（）

select max(字段) from 表名；

#### 求和sum（）

select sum(字段) from 表名；

#### 平均值avg()

select avg(字段) from 表名；

#### 统计个数count()

select count(字段) from 表名

#### as 聚合别名

常用在聚合函数后面给字段起别名

select 运算函数(字段) as 别名 from 表名；

#### 大小写转换

upper（）/lower（）

select upper(字段) from 表名；

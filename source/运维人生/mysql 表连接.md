# mysql 表连接

## 创建表并插入测试数据

```mysql

CREATE TABLE Person (
    id int primary key auto_increment,
    name varchar(32) unique not null default '',
    sex bool not null default 1,
    age int not null DEFAULT 0
) ENGINE=InnoDB DEFAULT CHARSET=utf8


CREATE TABLE Address  (
    id int primary key auto_increment,
    p_id int not null,
    province varchar(32) not null default '',
    city varchar(32)  not null default ''
) ENGINE=InnoDB DEFAULT CHARSET=utf8

insert into Person(name,age,sex) value 
("张三",1,28),
("李四",1,21),
("王五",1,33),
("赵六",0,11),
("元七",1,64),
("冯八",1,56)


insert into Adress(p_id,province,city) value 
(1,'北京','北京'),
(2,'上海','上海'),
(3,'江苏','南京'),
(4,'安徽','蚌埠'),
(5,'江苏','徐州'),
(5,'河北','雄安'),
(8,'广州','惠州')


```

## 左连接 右连接 内连接

内连接: 组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分。

左连接: left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。

右连接: right join是right outer join的简写，它的全称是右外连接，是外连接中的一种。 左表(a_table)只会显示符合搜索条件的记录，而右表(b_table)的记录将会全部表示出来。左表记录不足的地方均为NULL。

```mysql

#内连接
SELECT name,sex,age, Address.province, Address.city FROM Person INNER JOIN Address on Person.id = Address.p_id

#左连接
SELECT name,sex,age, Address.province, Address.city FROM Person LEFT JOIN Address on Person.id = Address.p_id

#右连接
SELECT name,sex,age, Address.province, Address.city FROM Person RIGHT JOIN Address on Person.id = Address.p_id

#使用AS示例(效果同左连接)
SELECT name,sex,age, A.province, A.city FROM Person as P INNER JOIN Address as A on P.id = A.p_id

```

显示效果

内连接

![](https://blog-1251258764.cos.ap-shanghai.myqcloud.com/mysql%20%E8%A1%A8%E8%BF%9E%E6%8E%A51.jpg)

左连接

![](https://blog-1251258764.cos.ap-shanghai.myqcloud.com/mysql%20%E8%A1%A8%E8%BF%9E%E6%8E%A52.jpg)

右连接

![](https://blog-1251258764.cos.ap-shanghai.myqcloud.com/mysql%20%E8%A1%A8%E8%BF%9E%E6%8E%A53.jpg)

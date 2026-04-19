---
title: "sql语句学习"
date: 2026-04-06
#image: "cover.jpg"   # 封面图（可选）
math: true            # 如果有公式，记得开启
hidden: true
categories:
    - "数据库"      # 这里填你想划分的板块名
tags:
    - "sql语句"
    - "MySql"
---


```sql
//console0
show databases;

use itcast;

show tables;


show create table employee;

alter table employee add workpl varchar(50) comment '工作地址';

//console0


-- 插入数据
insert into employee(id, workno, name, gender, age, idcard, entrydate,workpl) values(1,'1','idcast','男',20,'123456789','20000101','天津');

insert into employee values(2,'2','idcast2','男',20,'423456789','20000101','北京');
insert into employee values(3,'3','idcast3','男',30,'523456789','20000101','天津'),
                           (4,'4','idcast6','男',18,'623456789','20010101','上海');


-- 修改id为1的数据，将name修改为itheima
update employee set name = 'itheima' where id = 1;

-- 修改两个字段，中间加逗号即可
update  employee set name = 'zw' ,gender = '女' where id = 4;

-- 修改表中某个字段的所有值,不加where
update employee set entrydate = '2019-09-01';

update employee set idcard = null where id = 4;



-- 删除
-- delete语句不加条件会删除整个表中的数据
delete from employee where gender = '女';
-- delete from employee;



-- 基础以及条件查询
-- distinct去重
select distinct name as '名字',age as '年龄' from employee where age Between 18 and 20 and idcard like '%X';



-- 聚合函数
-- 统计员工数量
select count(*) from employee;
select count(idcard) from employee; -- null 值不参与聚合函数的统计

-- 统计员工平均年龄
select avg(age) from employee;
-- 统计最大年龄
select max(age) from employee;

-- 统计特定字段下员工年龄之和
select sum(age) from employee where gender = '男';



-- 分组查询
-- where 在分组前过滤，having 在分组后对结果进行过滤
-- where 不能对聚合函数判断， having可以

-- 根据性别进行查询，统计男女员工数量
select gender,count(*) from employee group by gender;
-- 根据性别 。。。，统计平均年龄
select gender,avg(age) from employee group by gender;
-- 查询年龄小于等于45，，按工作地址分组，获取员工数大于等于2的工作地址
select workpl as '工作地址',count(*) as address_count from employee where age <= 45 group by workpl having count(*) >= 2;

-- 分组之后，查询的字段一般为聚合函数和分组的字段



-- 排序查询
-- 根据年龄，升序
select * from employee order by age asc;

select * from employee order by id desc;

select * from employee order by age asc,id desc;



-- 分页查询              起始索引，索引数
select * from employee limit 0,2;
select * from employee limit 2; -- 效果同第一条
select * from employee limit 2,2;

-- homework
select * from employee where gender = '女' and age in(18,20,22,24);
select * from employee where gender = '男' and age between 20 and 25 and name like '_______';
select gender,count(*) from employee where age <= 20 group by gender;
select name,age from employee where age <= 25 order by age asc,workpl desc;
select * from employee where gender = '男' and age between 20 and 40 order by age asc,idcard asc limit 0,2 ;


/////
-- DCL语句，管理用户

-- 创建用户itcast，只能在当前主机localhost访问，密码123456；
create user 'itcast'@'localhost' identified by '123456';

-- 创建用户heimaa，可以在任意主机访问该数据库
create user 'heimaa'@'%' identified by '123456';

-- 修改用户密码
alter user 'heimaa'@'%' identified with mysql_native_password by '1234';

-- 删除用户
drop user 'itcast'@'localhost';



-- 权限控制
-- 查询权限
show grants for 'root'@'localhost';
show grants for 'heimaa'@'%';

-- 授予权限
-- grant 权限列表 on 数据库名.表名 to '用户名'@‘主机名’;
grant all on itcast.* to 'heimaa'@'%';

-- 撤销权限
revoke all on itcast.* from 'heimaa'@'%';


///////

-- 常用的字符串函数
select concat('Hello',' world');

select lower('HELLo');

select upper('hello');

select lpad('01',5,'-');

select rpad('01',5,'-');

select trim('  hello   world   '); -- trim()删除字符串首尾的空格

select substring('Hello world! ',1,7); -- 从1号位截取长度为7的



-- 数值函数
select ceil(1.2); -- 向上取整
select floor(1.2); -- 向下取整
select mod(12,5); -- 12/5的模
select rand();  -- 0-1的随机数
select round(1.235,2); -- 返回1.235四舍五入的值，保留2位小数

select lpad(round(rand()*1000000,0),6,'0'); -- 案例随机生成六位数码




-- 日期函数
select curdate(); -- 当前日期
select curtime(); -- 当前时间
select now();

select year(now());
select month(now());
select day(now());

select date_add(now(),INTERVAL 70 day);  -- 指定时间间隔后的时间
select date_add(now(),interval 70 month);
select date_add(now(),interval 70 year);

select datediff('20211201','20240301'); -- 两时间间隔天数（前减去后）

-- 返回员工名字以及入职天数
select name,datediff(curdate(),entrydate) as entrydays from employee order by entrydays desc;



-- 流程函数
-- 一号位为true则返回二号位，否则返回三号位
select if(true, 'ok','error');
-- 一号位为空则返回二号位，否则返回一号位
select ifnull('ok','default');
select ifnull('','default');
select ifnull(null,'hello');
select ifnull(null,null);

-- case when then else end
select
    name,
    (case workpl when '北京' then '一线城市' when '天津' then '新一线城市' else '其他城市' end) as '工作地址'
from employee;



//////

create table user(
    id int primary key auto_increment comment '主键',
    name varchar(10) not null unique comment '姓名',
    age int check ( age > 0 and age <= 120 ) comment '年龄',
    status char(1) default  '1' comment '状态',
    gender char(1) comment '性别'
)comment '用户表';



insert into user(name,age,status,gender) values('tom1',19,'1','男'),('jack2',23,'0','男');



-- 外键相关
-- ......

-- 输入我们要的数据
insert into dept(id,name) values(1,'研发部'),(2,'市场部'),(3,'财务部'),(4,'销售部'),(5,'总经部');
insert into emp(id, name, age, job, salary, entrydate, managerid, dept_id) values
    (1,'金庸',66,'总裁',20000,'20200101',null,5),(2,'张三',20,'项目经理',12500,'20201203',1,1),
    (3,'李四',33,'开发',10500,'20211203',2,1),(4,'阿伟',40,'开发',12500,'20220903',2,1),
    (5,'小马',34,'开发',10000,'20221217',3,1),(6,'小刘',18,'心理咨询师',9000,'20210703',2,1);

-- 生成外键的格式 （在子表中进行外键的添加）
-- alter table 子表名 add constraint 外键名 foreign key (需要添加外键的字段) references 主表名(主表中的字段名);
alter table emp add constraint fk_emp_dept_id foreign key (dept_id) references dept(id) on update cascade on delete cascade;
-- 删除外键
-- alter table 子表名 drop foreign key 外键名;

-- 删除更新行为: no action, restrict 都不允许删除
-- cascade 如果存在外键则要把外表相关的也删除
-- set null 设置子表中的外键值为null
-- set default 设置为一个默认值 （innodb不支持）
-- 直接在生成外键的语句后加: on update 行为 on delete 行为


///////
-- 多表查询 -- 笛卡尔积（多表查询的时候需要删除无效的笛卡尔积）
select *
from emp,
     dept
where emp.dept_id = dept.id;
-- 用where进行条件约束来删除无效数据

-- 多表查询分类
-- 连接查询
-- 1，内连接（两张表交集部分）
-- 隐式内连接 (如果有置空的数据无法查询)
select emp.name, dept.name
from emp,
     dept
where emp.dept_id = dept.id;

select e.name, d.name
from emp e,
     dept d
where e.dept_id = d.id;
-- 取了别名，就不能用原来的名字

-- 显示内连接
select e.name, d.name
from emp e
         join dept d on d.id = e.dept_id;

-- 2,外连接
-- 左外连接(查询左表所有数据 )
select e.*, d.name
from emp e
         left outer join itheima.dept d on d.id = e.dept_id;
-- 右外连接（查询右表......）
select e.*, d.*
from emp e
         right outer join itheima.dept d on d.id = e.dept_id;


-- 自连接 (记得要两个别名)
-- 员工及其所属领导
select e.name, e1.name as leadername
from emp e,
     emp e1
where e.managerid = e1.id; -- 内连接

select e.name, e1.name
from emp e
         left join emp e1 on e.managerid = e1.id;
-- 外连接


-- 联合查询（多次查询的结果合并)
-- 查询的字段类型，个数必须相同
select *
from emp
where salary < 10000
union all
select *
from emp
where age > 40;

-- 删除all后会去重
select *
from emp
where salary < 10000
union
select *
from emp
where age > 40;



-- 嵌套查询（子查询）

-- 标量子查询（子查询返回的是单个值（数字，字符串，日期））
select *
from emp
where dept_id = (select id from dept where name = '研发部');

select *
from emp
where entrydate > (select entrydate from emp where name = '金庸');

-- 列子查询（返回的值是一行）
select *
from emp
where dept_id in (select id from dept where name = '研发部' or name = '总经部');
--  查询比研发部所有人工资都高的人的信息
select *
from emp
where salary > all (select salary from emp where dept_id = (select id from dept where name = '研发部'));
-- 同上，但是比一个人高就行（用some，或any）
select *
from emp
where salary > some (select salary from emp where dept_id = (select id from dept where name = '研发部'));


-- 行子查询
-- 查询张三的薪资及其直属领导都相同的员工的信息
select *
from emp
where (salary, managerid) = (select salary, managerid from emp where name = '张三');


-- 表子查询
-- 嵌套的查询结果是子表


-- 练习

create table salgrade
(
    grade int,
    losal int,
    hisal int
) comment '薪资等级表';

insert into salgrade(grade, losal, hisal)
VALUES (1, 0, 3000),
       (2, 3001, 5000),
       (3, 5001, 8000),
       (4, 8001, 10000),
       (5, 10001, 15000),
       (6, 15001, 20000),
       (7, 20001, 25000),
       (8, 25001, 30000);

-- practice
select emp.name, emp.age, emp.job, dept.name
from emp,
     dept
where dept.id = emp.dept_id;

select e.name, e.age, e.job, d.name
from emp e
         join dept d on e.dept_id = d.id
where e.age < 30;

select distinct d.id, d.name
from emp e,
     dept d
where d.id = e.dept_id;

select e.*, d.name
from emp e
         left outer join itheima.dept d on d.id = e.dept_id
where e.age > 30;

select e.name, s.grade
from emp e,
     salgrade s
where e.salary between s.losal and s.hisal; -- 连接条件


select e.*, s.grade
from emp e
         left join salgrade s on e.salary between s.losal and s.hisal
where e.dept_id = (select id from dept where dept.name = '研发部');
-- 或者三表一起查
select e.*, s.grade
from emp e,
     salgrade s,
     dept d
where d.id = e.dept_id
  and e.salary between s.losal and s.hisal
  and d.name = '研发部';

select avg(e.salary)
from emp e,
     dept d
where e.dept_id = d.id
  and d.name = '研发部';


select d.name , count(*) as '人数' from emp e,dept d where e.dept_id = d.id group by d.name ;
-- 或者
select d.id,d.name,(select count(*) from emp e where e.dept_id = d.id) '人数' from dept d;


-- 一个查询要求可以由子查询或者多表联查实现



///////
-- 事务是一系列操作的集合，事务会把所有操作作为一个整体一起向系统提交或撤销操作请求
-- 要么同时成功，要么同时失败

create table account(
    id int auto_increment primary key comment  '生成ID',
    name varchar(10) comment '姓名',
    money int comment '余额'
)comment '账户表';
insert into account(id, name,money) values (null,'张三',2000),(null,'李四',2000);

update account set money = 2000 where name = '张三' or name = '李四'; -- 恢复数据


-- mysql 中的事务是默认直接自动提交的

-- 事务操作
-- 查看/设置事务提交方式
select @@autocommit; -- 1 为自动提交，0为手动提交
set @@autocommit = 1;

-- 开启事务
-- start transaction 或 begin

-- 手动提交事务
-- commit;

-- 回滚事务
-- rollback;


start transaction; -- 开启事务
select money from account where name = '张三';
update account set money = money - 1000 where name = '张三';

update account set money = money + 1000 where name = '李四';
commit; -- 没异常就提交
rollback ; -- 有异常回滚




-- 事务的四大特性 （ACID）
-- 原子性：事务是最小操作单元，要么全成功，要么全失败
-- 一致性：完成时所有数据保持一致状态
-- 隔离性：数据库系统提供隔离机制，保证事务不受外部并发操作影响的独立环境下运行
-- 持久性：事务一旦提交或者回滚，对数据库的数据改变就是永久的

-- 并发事务问题
-- 脏读：一个事务读取到另一个事务还没提交的数据
-- 不可重复读：一个事务先后读取同一条记录，但两次读取得到的数据不同（在两次之间另一个事务提交了修改）
-- 幻读：一个事务按条件查询数据时。，没有对应的数据，但在插入数据时，又发现这行数据被其他事务插入了，已存在

-- 事务的隔离级别就是解决并发事务问题的
-- 隔离级别见表level
insert into level(name, zangdu, bukechongfu, huandu) values('Read uncommitted','t','t','t'),
            ('Read committed','f','t','t'),('Repeatable Read(默认)','f','f','t'),
            ('Serializable','f','f','f');
-- t 表示会出现

-- 查看事务隔离级别
select @@transaction_isolation;
-- 设置事务隔离级别
set session transaction isolation level repeatable read ;



///////
-- 创建索引
create index idx_emp on emp(name);

-- 查看索引
show index from emp;

-- 删除索引
drop index idx_emp on emp;
-- 查看索引
show index from emp;

-- 主键字段会自动创建主键索引
-- 添加唯一约束，数据库会添加上唯一索引


# use itheima;
# alter table emp add index newIndex(name, age) 创建索引

explain select * from emp where id = 5




///////

use xiaozhaovip;
CREATE TABLE `xiaozhaoVIP__taker_detail` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `UserId` int(11) NOT NULL,
  `CollegeId` int(11) NOT NULL,
  `CollegeName` varchar(20) NOT NULL,
  `XueliType` tinyint(2) NOT NULL COMMENT '1-本科 ; 2-硕士 ；3 专科',
  `Status` tinyint(1) NOT NULL COMMENT '0-未认证 / 认证失效; 1-认证成功',
  `AdmissionDate` date DEFAULT NULL,
  `GraduationDate` date DEFAULT NULL,
  `DepositLevelType` tinyint(2) NOT NULL COMMENT '押金等级 0 1 2 3',
  `RamainAuthTimes` int(11) NOT NULL,
  `AddTime` datetime NOT NULL,
  `UpdateTime` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='接单人详情表';

INSERT INTO `xiaozhaoVIP__taker_detail` (
  `UserId`,
  `CollegeId`,
  `CollegeName`,
  `XueliType`,
  `Status`,
  `AdmissionDate`,
  `GraduationDate`,
  `DepositLevelType`,
  `RamainAuthTimes`,
  `AddTime`,
  `UpdateTime`
) VALUES (
  123456,
  8, -- 假设学校Id为8
  '南京航天航空大学', -- 假设学校名称为'南京航天航空大学'
  1, -- 假设学历类型为本科
  1, -- 假设认证状态为认证成功
  '2018-09-01', -- 假设入学时间为2018-09-01
  '2022-06-30', -- 假设毕业时间为2022-06-30
  1, -- 假设押金等级为1
  3, -- 假设年内剩余认证修改次数为3
  NOW(), -- 添加时间设置为当前时间
  NOW() -- 更新时间设置为当前时间
);


CREATE TABLE `xiaozhaoVIP__paotui_colllege` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `CollegeName` varchar(20) NOT NULL COMMENT '学校名称',
  `Status` tinyint(1) NOT NULL COMMENT '0-失效 删除; 1-正常',
  `EmailSuffix` varchar(20) DEFAULT NULL COMMENT '学校邮箱后缀',
  `AddTime` datetime NOT NULL,
  `UpdateTime` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='跑腿开通学校表';

INSERT INTO `xiaozhaoVIP__paotui_colllege` (
  `CollegeName`,
  `Status`,
  `EmailSuffix`,
  `AddTime`,
  `UpdateTime`
) VALUES (
  '南京航天航空大学',
  1, -- 正常状态
  'nuaa.edu.cn', -- 学校邮箱后缀
  NOW(), -- 添加时间设置为当前时间
  NOW() -- 更新时间设置为当前时间
);

UPDATE `xiaozhaoVIP__taker_detail`
SET CollegeId = 1
WHERE id = 1;

CREATE TABLE `xiaozhaoVIP__mail_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `Email` int(11) NOT NULL COMMENT '邮箱',
  `Status` tinyint(1) NOT NULL COMMENT '0-未校验; 1-已校验 【redis方案不走这个逻辑】',
  `Code` int(11) NOT NULL COMMENT '6位验证码',
  `AddTime` datetime NOT NULL,
  `UpdateTime` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='短信表（不区别业务）';

CREATE TABLE `xiaozhaoVIP__taker_auth_audit` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `UserId` int(11) NOT NULL COMMENT '主表的用户id',
  `CollegeId` int(11) NOT NULL COMMENT '学校Id',
  `CollegeName` varchar(20) NOT NULL COMMENT '学校名称',
  `XueliType` tinyint(2) NOT NULL COMMENT '1-本科 ; 2-硕士 ; 3-专科',
  `AdmissionDate` date NOT NULL COMMENT '入学时间',
  `GraduationDate` date NOT NULL COMMENT '毕业时间',
  `AddTime` datetime NOT NULL COMMENT '添加时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='接单人认证流水表';
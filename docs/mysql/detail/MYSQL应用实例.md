一般大型的sql会放在一个脚本文件里写，然后进入mysql命令行通过source test.sql的方式运行脚本命令或`D:\mysql\bin\mysql –uroot –p123456 -Dtest<d:\test\ss.sql`运行

## 根据一批id，查询这些id所对应的所有数据

> 一般查询in(xxx,xx)等语句会对in的数据有长度限制，所以这时候就要创建临时表，从文件中导入所有id，然后与其他表做join查询，如下所示：

```sql
use test;
-- 创建临时表存储文件中的一批userid
drop table if exists `tmp_userid`;
create table `tmp_userid`(id int(11));
load data infile '/Users/zhangbingbing/Work/mysql/script/userids.txt' into table `tmp_userid`;
-- -- 从文件导入数据到user表,数据如下：
-- -- aa,bb,cc
-- -- 11,22,33
-- load data infile '/Users/zhangbingbing/Work/mysql/script/data.txt' into table `user` fields terminated BY ',' lines terminated by '\n';
-- 创建临时表tmp_user,并选择指定条件的数据填充
drop table if exists `tmp_user`;
create table `tmp_user` 
    select a.id,a.name,a.email from `user` a join `tmp_userid` b on (a.id=b.id);

-- insert into `tmp_user`
--     select * from `user` where id not in(1,4);
-- 
-- 导出临时表`tmp_user`表里的数据到tmp_user.txt文件里
select * from `tmp_user` into outfile '/Users/zhangbingbing/Work/mysql/script/tmp_user.txt' fields terminated BY ',' optionally enclosed by '"' lines terminated by '\n';
```

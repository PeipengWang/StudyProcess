# Sybase死锁问题查询与解决
sp_who  查看锁表情况
sp_lock  查看被锁的表的id号
查看数据库lock配置
sp_config ‘number of lock’
数据库锁资源使用情况
sp_lock
检查锁资源使用情况
select fid, spid, loid, locktype = v1.name,
page, row,
objectName = rtrim(db_name(dbid)) + ‘…’ +
rtrim(object_name(id,dbid)),id,
class, context=v2.name
from master…syslocks l,
master…spt_values v1,
master…spt_values v2
where l.type = v1.number
and v1.type = “L”
and (l.context+2049) = v2.number
and v2.type = “L2”
order by fid, spid, loid, objectName, page,
row, locktype
检查死锁情况
用sp_who获取关于被阻碍进程的信息。
检查死锁情况
用sp_who获取关于被阻碍进程的信息。
sp_who的显示结果中：
status列显示“lock sleep”。
blk列显示保持该锁或这些锁的进程标识，即被谁锁定了。
loginame列显示登录操作员。结合相应的操作员信息表，便可知道操作员是谁。
locktype列显示加锁的类型和封锁的粒度，有些锁的后缀还带有blk表明锁的状态。前缀表明锁的类型：Sh—共享锁，Ex—排它锁或更新锁，中间表明锁死在表上（”table”或’intent’）还是在页上（page）.。
后缀“blk”表明该进程正在障碍另一个需要请求锁的进程。 一旦正在障碍的进程一结束，其他进程就向前移动。“demand”后缀表明当前共享锁一释放， 该进程就申请互斥锁。
table_id列显示表的id号，结合sysobjects即可查出被封锁的表名
备注：有时候死锁有可能是数据库中某些进程被未完成的事务阻塞资源而导致的。可以用select * from master.syslogshold查看未完成的任务。

查看执行计划是否走索引
1、查看进程执行计划是否走索引
sp_showplan 进程号
2、分析语句执行计划是否走索引
SET NOCOUNT ON
SET SHOWPLAN ON
SET NOEXEC ON
GO
具体代码
GO
SET NOCOUNT OFF
SET SHOWPLAN OFF
SET NOEXEC OFF
GO

你可以使用 set showplan 命令来检查删除语句是否使用了索引。该命令的输出将显示语句的执行计划，包括是否使用了索引。以下是一个示例：
set showplan on
go
delete from mytable where mycolumn = 'myvalue'
go
然后，您可以通过运行以下命令关闭 showplan 选项：
set showplan off
go
使用（!= ,<,>）无法使用索引，导致全表扫描。
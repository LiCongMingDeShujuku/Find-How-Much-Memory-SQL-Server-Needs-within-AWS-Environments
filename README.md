![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 查找SQL Server在AWS环境下需要多少内存
#### Find How Much Memory SQL Server Needs within AWS Environments
**发布-日期: 2015年01月06日 (评论)**

![#](images/##############?raw=true "#")

## Contents

- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


This is alittle write-up I did on determining the average memory, and io for each database instance. This is a global solution, and can be deployed across any SQL Server 2012 environment. Presently this is running on our AWS Cluster environment.

这是我在确定每个数据库实例的平均内存和io时写的小代码，这是一个全局解决方案，可以跨任何SQL Server 2012环境进行部署。目前，这正在我们的AWS Cluster环境中运行。

Getting the correct trending metrics for SQL Server Memory & IO Consumption.

获取SQL Server内存和IO消耗的正确趋势指标。

Question:
问题：

What can you do to configure the database environment in such a way that you can get a basic metric for how much is being used?
你可以采用哪种方式配置数据库环境，以便获得使用量的基本指标？

Well… You can do this directly with just a handful of objects. It’s basically a slim down SQL monitoring solution. You create a single database, and collect the daily performance information with a few queries.

通过几个对象你就可以实现它。它大致是一种简单的SQL监控解决方案。你可以创建单个数据库，并通过查询（queries）收集每日性能信息。

In my case I created a database called DBSYSMON meaning ‘Database System Monitor’ because I am only collecting performance information that is associated with the Database system it’s self.

在这个例子中，我创建了一个名为DBSYSMON的数据库，意思是“数据库系统监视器”，因为我只用它收集与数据库系统相关的性能信息。

The juicy part of this post is at the very bottom ( after you have created all the corresponding objects ).

这篇文章的精华部分在最后面（在你创建了所有相应的对象之后）。

The SQL logic below creates the following objects to help you get things going.

下面的SQL逻辑（logic）创建了以下对象，以帮助你实现目标。

Database: DBSYSMON
Table: SDIVFS_IO_STATS
Table: SDOPC_MAIN_TABLE
Agent Job: DBSYSMON_COLLECTION

Note: This logic was created for SQL Server 2012 edition and later so I would only recommend running this on newer environments.

注意：此逻辑是为SQL Server 2012及更高版本创建的，因此我建议在较新的环境中运行。

The logic first goes to the registry to gather the default Data, and Log file paths so it will create the database according to your existing structure. Additionally; the database is set to simply recovery so you don’t have to manage any log growth as a result of your table queries, or general table population.

此逻辑（logic）中首先用注册表（registry）收集默认数据和日志文件路径，根据现有的结构创建数据库。另外，数据库被设置为简单恢复（simply recovery），因此你就不需要管理因为表查询（table queries）或常规表填充（table population）导致的任何日志增长。

Ok so I hear you ask; What’s the deal with the tables?
Let’s take this one table at a time.

我估计你会问，“这和表（table）有什么关系？”
我们一个表一个表来看。

Table: SDIVFS_IO_STATS
Notice the prefix SDIVFS. This is basically the acronym for Sys.Dm_Io_Virtual_File_Stats so the table will store all the information from the DMV and has a Primary Key ( int identity ), and a timestamp so you can see when the performance information was collected. As you may already know the sys.dm_io_virtual_file_stats is cumulative, and only good since the last reboot, but… if you’re collecting them regularly then you can trend out the data and “since last restart ” is no longer the case. Some of the information will not be useful, but some of it will. You’ll have to run your own queries against it to see what you can use.

请注意前缀SDIVFS。这是Sys.Dm_Io_Virtual_File_Stats的首字母缩写，所以该表将用来存储来自DMV的所有信息，它有一个主键（int标识）和时间戳，可以查看收集性能信息的时间。你应该知道sys.dm_io_virtual_file_stats是累积的，并且只从上一次重启之后才有效，但是......如果你定期收集它们，你可以输出数据，那么“自上一次重启以来”就不存在了。有些信息没用，但有些信息会有用。 你必须通过查询才能看到有用的内容。

Table: SDOPC_MAIN_TABLE
Again; the prefix is SDOPC. This is for Sys.Dm_Os_Performance_Counters. Having this information stored into a table with a timestamp can be extremely valuable. As usual; the table has a Primary Key ( int identity ).

这个的前缀是SDOPC。 是Sys.Dm_Os_Performance_Counters的缩写。将此信息存储到带有时间戳的表中可能会非常有价值。 照理这个表该有一个主键（int标识）。

Agent Job: DBSYSMON_COLLECTION
No mystery here. The DBSYSMON_COLLECTION job will simply gather up the virtual file stats, and the performance counters, and populate the tables. The SQL logic below will automatically populate the tables upon first run anyway so you can start running queries against those tables right away ( I will provide a cursory set of queries at the bottom of this post to get you started ). The SQL Agent Job is designed to run every 15 minutes, and is created under the SQL Service Account MyDomain\MySQLServiceAccount

一点也不神秘。DBSYSMON_COLLECTION作业将会简单地收集虚拟文件统计信息和性能计数器，并且填表。下面的SQL逻辑（logic）会在首次运行时自动填表，方便你立即对这些表进行查询（我将在本文的最后提供一些粗略的查询来帮助你入门）。这个SQL代理作业我建在名为MyDomain \ MyServiceAccount的示例SQL服务帐户下，设为每15分钟运行一次。

The Job has two steps.
1. Gather All Stats From SDOPC
2. Gather Stats From SDIVFS

此作业分为两步：
1.从SDOPC收集所有统计数据
2.从SDIVFS收集所有统计数据

I’ve taken the liberty of adding the ‘table create’ logic in the SQL Job Steps in case you feel like dropping the database and recreating… the tables will automatically be created, and populated.

我自作主张在SQL作业步骤中添加了“table create”逻辑（logic），以防你想要删除数据库并重新创建，这些表将会自动创建并自动填表。

The good news the logic can be run across any SQL Server 2012 instance, and it will create the database in the correct location regardless of the instance name.

好消息是这个逻辑（logic）可以在任何SQL Server 2012实例上运行，不管实例名称是什么，它都能在正确的位置创建数据库。

Unless we feel the need to keep data beyond 6 months or so I would say it would be a good idea to create a 3rd Job step which includes some house cleaning. Basically removing rows that are older than 6 months, or whatever the limit works for us.

除非6个月以上的数据你还需要，否则我觉得创建第三个工作步骤是个不错的主意，其中包括一些house cleaning。 能删除超过6个月的行（rows），或你可以设置任何你认为最好的时间限制。

---
## Logic
```SQL
--To run this logic you will need SQL 'sysadmin' rights.
--要运行此逻辑，你需要有SQL“sysadmin”权限。
--The following SQL logic creates the database:  DBSYSMON
要运行此逻辑，你需要有SQL“sysadmin”权限。
--DBSYSMON is where database system performance information is stored for trending, reporting etc.
DBSYSMON存储数据库系统性能信息用于趋势分析，报告等。
--DBSYSMON is home for the following tables
 DBSYSMON是下列表格的基地。

--Note
注意
--The database DBSYSMON will become quite large.  Aggressive table management, and data file
--management is warranted.
数据库DBSYSMON将会变得非常大。需要积极的表格和数据文件管理。
--sdopc_main_table
--sdivfs_io_stats
--This logic will also create the tables, and the SQL  that populate them.
此逻辑还将创建表格以及填表的SQL Agent 作业。

--This code can be run on any SQL 2012 environment or later.
这些代码可以在SQL 2012或更新环境下运行。
--Note:
注意：
--When Jobs are created they will use the rights of the SQL Service account.
创建作业时，需要使用SQL Service帐户的权限。

--IF this account does not exist; the Job will need to be altered to include the standing 
--SQL Service account on whatever server this code has been deployed.
如果此帐户不存在，需要对作业进行修改，使其包含已部署此代码的任何服务器上的常设SQL服务帐户。
 
 
PRINT '  '
PRINT '  ***********************************************************************'
PRINT '  '
PRINT '   RUNNING SQL LOGIC FOR DBSYSMON'
PRINT '  '
为DBSYSMON运行SQL逻辑（logic）
PRINT '  ***********************************************************************'
PRINT '  '
PRINT '  GATHERING DEFAULT DATA LOG FILE LOCATIONS FOR DATABASE CREATION'
PRINT '  '
为创建数据库收集默认数据和日志文件位置信息

 
	gather default data and log file location information from the registry use master; set nocount on
使用master从注册表（registry）中收集默认数据和日志文件位置信息，设置nocount
 
declare @defaultdata nvarchar(512)
exec master.dbo.xp_instance_regread
                'hkey_local_machine'
,               'software\microsoft\mssqlserver\mssqlserver'
,               'defaultdata'
,               @defaultdata output
 
declare @defaultlog nvarchar(512)
exec master.dbo.xp_instance_regread 
                'hkey_local_machine'
,               'software\microsoft\mssqlserver\mssqlserver'
,               'defaultlog'
,               @defaultlog output
 
declare @masterdata nvarchar(512)
exec master.dbo.xp_instance_regread 'hkey_local_machine'
,               'software\microsoft\mssqlserver\mssqlserver\parameters'
,               'sqlarg0'
,               @masterdata output
 
select @masterdata=substring(@masterdata, 3, 255) select @masterdata=substring(@masterdata, 1, len(@masterdata) - charindex('\', reverse(@masterdata)))
 
declare @masterlog nvarchar(512)
exec master.dbo.xp_instance_regread 'hkey_local_machine'
,               'software\microsoft\mssqlserver\mssqlserver\parameters'
,               'sqlarg2'
,               @masterlog output
 
select @masterlog=substring(@masterlog, 3, 255) select @masterlog=substring(@masterlog, 1, len(@masterlog) - charindex('\', reverse(@masterlog)))
 
	create the DBSYSMON database
创建DBSYSMON数据库
 
PRINT '  '
PRINT '  CREATING DATABASE DBSYSMON'
PRINT '  '
创建DBSYSMON数据库
 
declare    @path_data_file                    varchar(255)
declare    @path_log_file                       varchar(255)
declare    @create_database varchar(max)
declare    @database_name                  varchar(255)
set                           @database_name                  = 'DBSYSMON'
set                           @path_data_file                    = ( select isnull(@defaultdata,              @masterdata)        defaultdata ) + '\DBSYSMON.mdf'
set                           @path_log_file                       = ( select isnull(@defaultlog, @masterlog)                          defaultlog               ) + '\DBSYSMON_log.ldf'
 
set                           @create_database = 
'create database [' + @database_name + ']  containment = none  on  primary ( name = ''' + @database_name + '_data''' + ', filename = ''' + @path_data_file + ''', size = 4096kb , filegrowth = 1024kb )  log on ( name = ''' + @database_name + '_log''' +  ', filename = ''' + @path_log_file  + ''', size = 2048kb , filegrowth = 10%); alter database ' + @database_name + ' set recovery simple'
 
if not exists ( select name from sys.databases where name = 'DBSYSMON' )
exec        (@create_database)
go
 
	create the table:  sdivfs_io_stats
	创建表格：sdivfs_io_stats
 
PRINT '  '
PRINT '  CREATING THE TABLE  SDIVFS_IO_STATS'
PRINT '  '
创建表格sdivfs_io_stats
 
if exists (select name from sys.databases where name = 'DBSYSMON')
                begin
                                if not exists (select name from sys.tables where name = 'sdivfs_io_stats')
                                begin
                                                create table dbsysmon.dbo.sdivfs_io_stats
(
rowid                                                                       int identity(1,1) primary key clustered
,               timestamp                                                              datetime
,               database_id                                                            smallint
,               num_of_reads                                        bigint
,               num_of_writes                                       bigint
,               size_on_disk_bytes                                bigint
,               io_stall                                                    bigint
,               io_stall_read_ms                    bigint
,               io_stall_write_ms                  bigint
,               data_file_name                                      varchar(255)
,               fileid                                                                        smallint
,               physicalname                                         varchar(255)
,               data_file_type                                        varchar(50)
)

-- populate the table:  sdivfs_io_stats
填充表格sdivfs_io_stats
PRINT '  '
PRINT '  POPULATING TABLE ON FIRST RUN'
PRINT '  '
 
                                                insert into dbsysmon.dbo.sdivfs_io_stats
                                                (
                                                                timestamp
                                                ,               database_id
                                                ,               num_of_reads
                                                ,               num_of_writes
                                                ,               size_on_disk_bytes
                                                ,               io_stall
                                                ,               io_stall_read_ms
                                                ,               io_stall_write_ms
                                                ,               data_file_name
                                                ,               fileid
                                                ,               physicalname
                                                ,               data_file_type
                                                )
                                                select
                                                                getdate()
                                                ,               sdivfs.database_id
                                                ,               sdivfs.num_of_reads
                                                ,               sdivfs.num_of_writes
                                                ,               ( ( sdivfs.size_on_disk_bytes / 1024 ) / 1024 / 1024 )
                                                ,               sdivfs.io_stall
                                                ,               sdivfs.io_stall_read_ms
                                                ,               sdivfs.io_stall_write_ms
                                                /*
                                                ,               sdivfs.sample_ms
                                                ,               sdivfs.num_of_bytes_read
                                                ,               sdivfs.num_of_bytes_written
                                                ,               sdivfs.io_stall_write_ms
                                                */ 
                                                ,               smf.name
                                                ,               sdivfs.file_id
                                                ,               smf.physical_name
                                                ,               db_file_type =        case 
                                                                                                                                                when sdivfs.file_id = 2 then 'log' 
                                                                                                                                                else 'data' 
                                                                                                                                end
                                                from
                                                                sys.dm_io_virtual_file_stats (null, null) sdivfs join sys.master_files smf on sdivfs.file_id = smf.file_id 
                                                                and sdivfs.database_id = smf.database_id
                                end
                end
 
	creating table sdopc_main_table
创建表格sdopc_main_table
PRINT '  '
PRINT '  CREATING TABLE SDOPC_MAIN_TABLE'
PRINT '  '
创建表格SDOPC_MAIN_TABLE
 
if exists(select name from sys.databases where name = 'DBSYSMON')
                begin
                                if not exists(select name from sys.tables where name = 'dbsysmon.dbo.sdopc_main_table')
                                create table dbsysmon.dbo.sdopc_main_table
                                (
                                                [rowid]                                    int identity(1,1) primary key clustered
                                ,               [servername]          varchar(255)
                                ,               [timestamp]                           datetime
                                ,               [object_name]        nchar(256)
                                ,               [counter_name]      nchar(256)
                                ,               [instance_name]    nchar(256)
                                ,               [cntr_value]            bigint
                                ,               [cntr_type]                              bigint
                                );
 -- populating table sdopc_main_table
 填充表格sdopc_main_table
PRINT '  '
PRINT '  POPULATING TABLE SDOPC_MAIN_TABLE ON FIRST RUN'
                                PRINT '  '
 
                                insert into dbsysmon.dbo.sdopc_main_table
                                (
                                                [servername]
                                ,               [timestamp]
                                ,               [object_name]
                                ,               [counter_name]
                                ,               [instance_name]
                                ,               [cntr_value]
                                ,               [cntr_type]
                                )
 
                                select
                                                cast(serverproperty('servername') as varchar(250))
                                ,               getdate()
                                ,               [object_name]
                                ,               [counter_name]
                                ,               [instance_name]
                                ,               [cntr_value]
                                ,               [cntr_type]
                                from
                                                sys.[dm_os_performance_counters]
                end
 
	CREATE  FOR DBSYSMON COLLECTION  
为DBSYSMON系列创建代理作业（）
 
PRINT '  '
PRINT '  CREATING  FOR DBYSMON COLLECTION'
PRINT '  '
为DBSYSMON系列创建代理作业（）
 
USE [msdb]
GO
 
if exists ( select name from msdb..sysjobs where name = 'DBSYSMON_COLLECTION' ) exec msdb..sp_delete_job @job_name = 'DBSYSMON_COLLECTION'
 
 
BEGIN TRANSACTION
DECLARE @ReturnCode INT
SELECT @ReturnCode = 0
 
IF NOT EXISTS (SELECT name FROM msdb.dbo.syscategories WHERE name=N'[Uncategorized (Local)]' AND category_class=1) BEGIN EXEC @ReturnCode = msdb.dbo.sp_add_category @class=N'JOB', @type=N'LOCAL', @name=N'[Uncategorized (Local)]'
IF (@@ERROR &lt;&gt; 0 OR @ReturnCode &lt;&gt; 0) GOTO QuitWithRollback
 
END
 
DECLARE @jobId BINARY(16)
EXEC @ReturnCode =  msdb.dbo.sp_add_job @job_name=N'DBSYSMON_COLLECTION', 
                                @enabled=1, 
                                @notify_level_eventlog=0, 
                                @notify_level_email=0, 
                                @notify_level_netsend=0, 
                                @notify_level_page=0, 
                                @delete_level=0, 
                                @description=N'No description available.', 
                                @category_name=N'[Uncategorized (Local)]', 
                                @owner_login_name=N'MyDomain\MyServiceAccount', @job_id = @jobId OUTPUT IF (@@ERROR &lt;&gt; 0 OR @ReturnCode &lt;&gt; 0) GOTO QuitWithRollback
 
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'Gather All Stats From SDOPC', 
                                @step_id=1, 
                                @cmdexec_success_code=0, 
                                @on_success_action=3, 
                                @on_success_step_id=0, 
                                @on_fail_action=3, 
                                @on_fail_step_id=0, 
                                @retry_attempts=0, 
                                @retry_interval=0, 
                                @os_run_priority=0, @subsystem=N'TSQL', 
                                @command=N'use DBSYSMON;
set nocount on
 
if not exists(select name from sys.tables where name = ''sdopc_main_table'')
 
create table sdopc_main_table
(
                [rowid]                                    int identity(1,1) primary key clustered
,               [servername]          varchar(255)
,               [timestamp]                           datetime
,               [object_name]        nchar(256)
,               [counter_name]      nchar(256)
,               [instance_name]    nchar(256)
,               [cntr_value]            bigint
,               [cntr_type]                              bigint
);
 
insert into sdopc_main_table
(
                [servername]
,               [timestamp]
,               [object_name]
,               [counter_name]
,               [instance_name]
,               [cntr_value]
,               [cntr_type]
)
select
                cast(serverproperty(''servername'') as varchar(250))
,               getdate()
,               [object_name]
,               [counter_name]
,               [instance_name]
,               [cntr_value]
,               [cntr_type]
from
                sys.[dm_os_performance_counters]', 
                                @database_name=N'DBSYSMON', 
                                @flags=0
IF (@@ERROR &lt;&gt; 0 OR @ReturnCode &lt;&gt; 0) GOTO QuitWithRollback
 
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'Gather Stats From SDIVFS', 
                                @step_id=2, 
                                @cmdexec_success_code=0, 
                                @on_success_action=1, 
                                @on_success_step_id=0, 
                                @on_fail_action=2, 
                                @on_fail_step_id=0, 
                                @retry_attempts=0, 
                                @retry_interval=0, 
                                @os_run_priority=0, @subsystem=N'TSQL', 
                                @command=N'use DBSYSMON;
set nocount on
 
if not exists (select name from sys.tables where name = ''sdivfs_io_stats'')
 
create table sdivfs_io_stats
(
                rowid                                       int identity(1,1) primary key clustered
,               timestamp                                              datetime
,               database_id                            smallint
,               num_of_reads                        bigint
,               num_of_writes                       bigint
,               size_on_disk_bytes                                bigint
,               io_stall                                    bigint
,               io_stall_read_ms                    bigint
,               io_stall_write_ms                  bigint
,               data_file_name                      varchar(255)
,               fileid                                        smallint
,               physicalname                         varchar(255)
,               data_file_type                        varchar(50)                             
)
 
insert into sdivfs_io_stats
(
                timestamp
,               database_id
,               num_of_reads
,               num_of_writes
,               size_on_disk_bytes
,               io_stall
,               io_stall_read_ms
,               io_stall_write_ms
,               data_file_name
,               fileid
,               physicalname
,               data_file_type
)
select
                getdate()
,               sdivfs.database_id
,               sdivfs.num_of_reads
,               sdivfs.num_of_writes
,               ( ( sdivfs.size_on_disk_bytes / 1024 ) / 1024 / 1024 )
,               sdivfs.io_stall
,               sdivfs.io_stall_read_ms
,               sdivfs.io_stall_write_ms
                --sdivfs.sample_ms, sdivfs.num_of_bytes_read, sdivfs.num_of_bytes_written, sdivfs.io_stall_write_ms, 
,               smf.name
,               sdivfs.file_id
,               smf.physical_name
,               db_file_type =        case
                                                when sdivfs.file_id = 2 then ''log''
                                                else ''data''
                                                end
from
                sys.dm_io_virtual_file_stats (null, null) sdivfs join sys.master_files smf on sdivfs.file_id = smf.file_id 
                and sdivfs.database_id = smf.database_id', 
                                @database_name=N'DBSYSMON', 
                                @flags=0
IF (@@ERROR &lt;&gt; 0 OR @ReturnCode &lt;&gt; 0) GOTO QuitWithRollback EXEC @ReturnCode = msdb.dbo.sp_update_job @job_id = @jobId, @start_step_id = 1 IF (@@ERROR &lt;&gt; 0 OR @ReturnCode &lt;&gt; 0) GOTO QuitWithRollback EXEC @ReturnCode = msdb.dbo.sp_add_jobschedule @job_id=@jobId, @name=N'Every 15 minutes', 
                                @enabled=1, 
                                @freq_type=4, 
                                @freq_interval=1, 
                                @freq_subday_type=4, 
                                @freq_subday_interval=15, 
                                @freq_relative_interval=0, 
                                @freq_recurrence_factor=0, 
                                @active_start_date=20141116, 
                                @active_end_date=99991231, 
                                @active_start_time=0, 
                                @active_end_time=235959, 
                                @schedule_uid=N'92a0ceb0-9ef9-4414-a097-0a4d2b953ebf'
IF (@@ERROR &lt;&gt; 0 OR @ReturnCode &lt;&gt; 0) GOTO QuitWithRollback EXEC @ReturnCode = msdb.dbo.sp_add_jobserver @job_id = @jobId, @server_name = N'(local)'
IF (@@ERROR &lt;&gt; 0 OR @ReturnCode &lt;&gt; 0) GOTO QuitWithRollback COMMIT TRANSACTION GOTO EndSave
QuitWithRollback:
    IF (@@TRANCOUNT &gt; 0) ROLLBACK TRANSACTION
EndSave:
 
GO
 
PRINT '  '
PRINT '  ***********************************************************************'
PRINT '  '
PRINT '   THE DBSYSMON DATABASE, TABLES, AND ASSOCIATED JOBS HAVE BEEN CREATED.'
PRINT '  '
DBSYSMON数据库，表格和相关作业已经创建
PRINT '  ***********************************************************************'
PRINT '  '


```

Here are a couple queries you can use to get some useful information out of the tables.

以下是一些可以从表格中获取有用信息的查询。

Here’s some SQL query logic to determine what the recommended amount of memory is needed for SQL Server according to the performance hit that is being carried out by any applications, or other queries. Keep in mind that are natural swings in the performance of any database system and you shouldn’t haphazardly add memory, or reconfigure memory for your environment based on a few results from this query. The good news is because the information is stored in the new DBSYSMON database, and we have a timestamp associated with the performance information you can begin to trend out the average swings in performance, and provision the necessary AWS Instance Types with actual guaranteed performance data to prove the Instance Type configuration that you need.

下面是一些SQL查询逻辑，用于根据任何应用程序或其他查询执行的性能命中来确定SQL Server所需的建议内存量。请记住，这是任何数据库系统性能的自然波动，不应随意添加内存，或根据此查询的一些结果为您的环境重新配置内存。好消息是因为信息存储在新的DBSYSMON数据库中，我们有一个时间戳与性能信息相关联，有助于你趋向于平均性能波动，还能提供必要的AWS实例类型和实际保证的性能数据来检验你需要的实例类型配置。

Whats better is that if the ‘recommended memory value’ is below that of existing configuration you can remove the Memory from AWS accordingly for the correct ‘Intance Type’ or ( memory packages ) so the Database systems that are less performance intensive can be properly managed with lower AWS ‘Instance Type’

如果“建议的内存值”低于现有配置，则可以相应地从AWS中删除内存以获取正确的“Intance Type”或（内存包），以便可以使用较低的AWS'Instance Type'恰当管理性能较低的数据库系统 。

```SQL
elect
                'rsm_timestamp'                    = datename(dw, rsm.timestamp) + ' ' + convert(char, rsm.timestamp, 9)
,               'rsm_cntr_value'                    = rsm.cntr_value
,               'ssm_cntr_value'                    = ssm.cntr_value
,               'recommended_memory_value'           = cast (rsm.cntr_value + ssm.cntr_value / 100 as varchar(10)) + ' mb'
from
                (select timestamp, cntr_value from sdopc_main_table where counter_name = 'Reserved Server Memory (KB)' ) rsm
                join
                (select timestamp, cntr_value from sdopc_main_table where counter_name = 'Stolen Server Memory (KB)' ) ssm on rsm.timestamp = ssm.[timestamp] where
                rsm.cntr_value &gt; 0
order by
                rsm.timestamp desc

```
This SQL query logic will basically return all the access times to the data. Stalls will show you how long queries are waiting etc. You can use this to determine the AWS Instance Type for storage. I will try to include some queries later on with conversions from MS to something human readable.

此SQL查询逻辑（logic）将会基本上返回对数据的所有访问时间。停顿将显示查询等待的时间等。你可以使用它来确定用于存储的AWS实例类型。 之后我会尝试将一些查询包含在从MS到人类可读的内容的转换中。

`select
                *
from
                sdivfs_io_stats
`


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")


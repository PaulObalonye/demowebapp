Pre-Migration Checklist

Analyze the disk space of the target server for the new database, if the disk space is not enough add more space on the target server
Confirm the data and log file location for the target server
Collect the information about the Database properties (Auto Stats, DB Owner, Recovery Model, Compatibility level, Trustworthy option etc)
Collect the information of dependent applications, make sure application services will be stopped during the database migration
Collect the information of database logins, users and their permissions. (Optional)
Check the database for the Orphan users if any
Check the SQL Server for any dependent objects (SQL Agent Jobs and Linked Servers)
Check, if the database is part of any maintenance plan
Below are various scripts you can run to collect data.

Script to Check the Disk and Database Size

-- Procedure to check disc space
exec master..xp_fixeddrives
-- To Check database size
exec sp_helpdb [dbName]
or
use [dbName]
select str(sum(convert(dec(17,2),size)) / 128,10,2)  + 'MB'
from dbo.sysfiles
GO
Script to Check Database Properties

select 
 sysDB.database_id,
 sysDB.Name as 'Database Name',
 syslogin.Name as 'DB Owner',
 sysDB.state_desc,
 sysDB.recovery_model_desc,
 sysDB.collation_name, 
 sysDB.user_access_desc,
 sysDB.compatibility_level, 
 sysDB.is_read_only,
 sysDB.is_auto_close_on,
 sysDB.is_auto_shrink_on,
 sysDB.is_auto_create_stats_on,
 sysDB.is_auto_update_stats_on,
 sysDB.is_fulltext_enabled,
 sysDB.is_trustworthy_on
from sys.databases sysDB
INNER JOIN sys.syslogins syslogin ON sysDB.owner_sid = syslogin.sid
Another Script to Check Database Properties

declare @dbdesc varchar(max)
declare @name varchar(10)
set @name='Master'
SELECT @dbdesc = 'Status=' + convert(sysname,DatabasePropertyEx(@name,'Status'))  
SELECT @dbdesc = @dbdesc + ', Updateability=' + convert(sysname,DatabasePropertyEx(@name,'Updateability'))  
SELECT @dbdesc = @dbdesc + ', UserAccess=' + convert(sysname,DatabasePropertyEx(@name,'UserAccess'))  
SELECT @dbdesc = @dbdesc + ', Recovery=' + convert(sysname,DatabasePropertyEx(@name,'Recovery'))  
SELECT @dbdesc = @dbdesc + ', Version=' + convert(sysname,DatabasePropertyEx(@name,'Version'))  
  
 -- These props only available if db not shutdown  
 IF DatabaseProperty(@name, 'IsShutdown') = 0  
 BEGIN  
  SELECT @dbdesc = @dbdesc + ', Collation=' + convert(sysname,DatabasePropertyEx(@name,'Collation'))  
  SELECT @dbdesc = @dbdesc + ', SQLSortOrder=' + convert(sysname,DatabasePropertyEx(@name,'SQLSortOrder'))  
 END  
  
 -- These are the boolean properties  
 IF DatabasePropertyEx(@name,'IsAutoClose') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAutoClose'  
 IF DatabasePropertyEx(@name,'IsAutoShrink') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAutoShrink'  
 IF DatabasePropertyEx(@name,'IsInStandby') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsInStandby'  
 IF DatabasePropertyEx(@name,'IsTornPageDetectionEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsTornPageDetectionEnabled'  
 IF DatabasePropertyEx(@name,'IsAnsiNullDefault') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAnsiNullDefault'  
 IF DatabasePropertyEx(@name,'IsAnsiNullsEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAnsiNullsEnabled'  
 IF DatabasePropertyEx(@name,'IsAnsiPaddingEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAnsiPaddingEnabled'  
 IF DatabasePropertyEx(@name,'IsAnsiWarningsEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAnsiWarningsEnabled'  
 IF DatabasePropertyEx(@name,'IsArithmeticAbortEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsArithmeticAbortEnabled'  
 IF DatabasePropertyEx(@name,'IsAutoCreateStatistics') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAutoCreateStatistics'  
 IF DatabasePropertyEx(@name,'IsAutoUpdateStatistics') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsAutoUpdateStatistics'  
 IF DatabasePropertyEx(@name,'IsCloseCursorsOnCommitEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsCloseCursorsOnCommitEnabled'  
 IF DatabasePropertyEx(@name,'IsFullTextEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsFullTextEnabled'  
 IF DatabasePropertyEx(@name,'IsLocalCursorsDefault') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsLocalCursorsDefault'  
 IF DatabasePropertyEx(@name,'IsNullConcat') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsNullConcat'  
 IF DatabasePropertyEx(@name,'IsNumericRoundAbortEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsNumericRoundAbortEnabled'  
 IF DatabasePropertyEx(@name,'IsQuotedIdentifiersEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsQuotedIdentifiersEnabled'  
 IF DatabasePropertyEx(@name,'IsRecursiveTriggersEnabled') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsRecursiveTriggersEnabled'  
 IF DatabasePropertyEx(@name,'IsMergePublished') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsMergePublished'  
 IF DatabasePropertyEx(@name,'IsPublished') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsPublished'  
 IF DatabasePropertyEx(@name,'IsSubscribed') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsSubscribed'  
 IF DatabasePropertyEx(@name,'IsSyncWithBackup') = 1  
  SELECT @dbdesc = @dbdesc + ', ' + 'IsSyncWithBackup'  
SELECT @dbdesc
Script to List Orphan Users

sp_change_users_login 'report'
GO
Script to List Linked Servers

select  *
from sys.sysservers
Script to List Database Dependent Jobs

select 
 distinct 
 name,
 database_name
from sysjobs sj
INNER JOIN sysjobsteps sjt on sj.job_id = sjt.job_id
Database Migration Checklist

These are the steps you would go through to make the change.

1. Stop the application services
2. Change the database to read-only mode (Optional) 

-- Script to make the database readonly
USE [master]
GO
ALTER DATABASE [DBName] SET  READ_ONLY WITH NO_WAIT
GO
ALTER DATABASE [DBName] SET  READ_ONLY 
GO
3. Take the latest backup of all the databases involved in migration

4. Restore the databases on the target server on the appropriate drives

5. Cross check the database properties as per the database property script output, change the database properties as per the pre migration- checklist 

Script to Change DB Owner

This will change the database owner to "sa".  This can be used to change to any owner you would like.

 USE databaseName
EXEC sp_changedbowner 'sa'
Script to Turn on Trustworthy Option

If trustworthy option was set, this will turn it on for the database.

 ALTER DATABASE database_name SET TRUSTWORTHY ON
Script to Change the Database Compatibility Level

When you upgrade to a new version, the old compatibility level will remain.  This script shows how to change the compatibility level to SQL Server 2005 compatibility .

ALTER DATABASE DatabaseName
SET SINGLE_USER
GO
EXEC sp_dbcmptlevel DatabaseName, 90;
GO
ALTER DATABASE DatabaseName
SET MULTI_USER
GO
6. Execute the output of Login transfer script on the target server, to create logins on the target server you can get the code from this technet article: http://support.microsoft.com/kb/246133.

7. Check for Orphan Users and Fix Orphan Users 

Script to Check and Fix Orphan Users

-- Script to check the orphan user
EXEC sp_change_users_login 'Report'
--Use below code to fix the Orphan User issue
DECLARE @username varchar(25)
DECLARE fixusers CURSOR 
FOR
SELECT UserName = name FROM sysusers
WHERE issqluser = 1 and (sid is not null and sid <> 0x0)
and suser_sname(sid) is null
ORDER BY name
OPEN fixusers
FETCH NEXT FROM fixusers
INTO @username
WHILE @@FETCH_STATUS = 0
BEGIN
EXEC sp_change_users_login 'update_one', @username, @username
FETCH NEXT FROM fixusers
INTO @username
END
CLOSE fixusers
DEALLOCATE fixusers
8. Execute DBCC UPDATEUSAGE on the restored database.

Run the DBCC UPDATEUSAGE command against the migrated database when upgrading to a newer version of SQL Server.

DBCC UPDATEUSAGE('database_name') WITH COUNT_ROWS
DBCC CHECKDB 
OR
DBCC CHECKDB('database_name') WITH ALL_ERRORMSGS
9. Rebuild Indexes (Optional) As per the requirement and time window you can execute this option.

Take a look at this tip to rebuild all indexes.

This will rebuild or reorganize all indexes for a particular table.

Index Rebuild :- This process drops the existing Index and Recreates the index.
Index Reorganize :- This process physically reorganizes the leaf nodes of the index.

-- Script for Index Rebuild
USE [DBName];
GO
ALTER INDEX ALL ON [ObjectName] REBUILD
GO
-- Script for Index Reorganize
USE AdventureWorks;
GO
ALTER INDEX ALL ON [ObjectName] REORGANIZE
GO
10. Update index statistics

sp_updatestats
11. Recompile procedures

Take a look at this tip to recompile all objects.

This will recompile a particular stored procedure.

sp_recompile 'procedureName'
12. Start the application services, check the application functionality and check the Windows event logs. 

13. Check the SQL Server Error Log for login failures and other errors

Take a look at this tip on how to read SQL Server error logs.

EXEC xp_readerrorlog 0,1,"Error",Null
14. Once the application team confirms that application is running fine take the databases offline on the source server or make them read only

-- Script to make the database readonly
USE [master]
GO
ALTER DATABASE [DBName] SET  READ_ONLY WITH NO_WAIT
GO
ALTER DATABASE [DBName] SET  READ_ONLY 
GO
-- Script to take the database offline
EXEC sp_dboption N'DBName', N'offline', N'true'
OR
ALTER DATABASE [DBName] SET OFFLINE WITH
ROLLBACK IMMEDIATE
Next Steps
Test the process to determine how much time and disk space would be needed by using the backup and recovery process.
Meet with your technical and business teams to find out how much time is available for the migration and plan the activity
Design the rollback plan, if the application is not working fine
Add more migration cases in your checklist, for example check if the database requires any server level change (For example CLR, XP_Cmdshell etc)
Some of these scripts give you the base command to update a portion of the data, enhance the process to hit each object in your database.

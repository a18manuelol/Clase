# Restoring a Backup to MSSQL in Kubernetes

## Verify MSSQL Version

```
SELECT @@VERSION
```

## Download Backup

```
./download-bkp.sh
POD=$(kubectl get pods | grep mssql-deployment | awk {'print $1'})
kubectl cp ./WideWorldImporters-Full.bak $POD:/var/opt/mssql/WideWorldImporters-Full.bak
```

## Restore Backup

```
restore database WideWorldImporters from disk = '/var/opt/mssql/WideWorldImporters-Full.bak' with
move 'WWI_Primary' to '/var/opt/mssql/data/WideWorldImporters.mdf',
move 'WWI_UserData' to '/var/opt/mssql/data/WideWorldImporters_UserData.ndf',
move 'WWI_Log' to '/var/opt/mssql/data/WideWorldImporters.ldf',
move 'WWI_InMemory_Data_1' to '/var/opt/mssql/data/WideWorldImporters_InMemory_Data_1'
go
```

## Run Queries (Have Fun!)

Get the tables
```
USE WideWorldImporters;
SELECT name, SCHEMA_NAME(schema_id) as schema_name FROM sys.objects WHERE type = 'U' ORDER BY name;
```

Get data from the **People** table
```
USE WideWorldImporters;
SELECT TOP 10 FullName, PhoneNumber, EmailAddress FROM [Application].[People] ORDER BY FullName;
```

Get server information
```
SELECT session_id, login_time, host_name, program_name, reads, writes, cpu_time
FROM sys.dm_exec_sessions WHERE is_user_process = 1
GO
SELECT dr.session_id, dr.start_time, dr.status, dr.command
FROM sys.dm_exec_requests dr
JOIN sys.dm_exec_sessions de
ON dr.session_id = de.session_id
AND de.is_user_process = 1
GO
SELECT cpu_count, committed_kb from sys.dm_os_sys_info
GO
```
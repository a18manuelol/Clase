# MSSQL Always-On on Kubernetes

## Deploy the Operator
```
kubectl apply -f operator.yaml
kubectl get pods -n ag1
```

## Create the Secret for the DB password
```
kubectl create secret generic sql-secrets --from-literal=sapassword="MyC0m9l&xP@ssw0rd" --from-literal=masterkeypassword="MyC0m9l&xP@ssw0rd" --namespace ag1
```

## Deploy the MSSQL Cluster
```
kubectl apply -f mssql.yaml -n ag1
watch kubectl get pods -o wide -n ag1
kubectl get services -n ag1
```

## Deploy the Primary and Secondary Endpoints

```
kubectl apply -f services.yaml -n ag1
```

## Check that Always On is Working

```
SELECT ar.replica_server_name, hars.role_desc, hars.operational_state_desc
FROM sys.dm_hadr_availability_replica_states hars
JOIN sys.availability_replicas ar
ON hars.replica_id = ar.replica_id
GO
```

## Create a Test DB

```
USE master
GO
CREATE DATABASE testag
GO
USE MASTER
GO
BACKUP DATABASE [testag] 
TO DISK = N'/var/opt/mssql/data/testag.bak'
GO
ALTER AVAILABILITY GROUP [ag1] ADD DATABASE [testag]
GO
```

```
USE testag
GO
CREATE TABLE ilovesql (col1 INT, col2 VARCHAR(500) NULL)
GO
INSERT INTO ilovesql VALUES (1, 'SQL Server 2019 is fast, secure, and highly available')
GO
```

## Connect to the Replica (Secondary)
```
SELECT 'Connected to Primary = '+@@SERVERNAME;USE testag;
SELECT * FROM ilovesql;
```

## Recovery (HA)

```
kubectl get pods -o wide --watch
```
```
POD=$(kubectl get pods | grep mssql-deployment | awk {'print $1'})
kubectl delete pod $POD
```

```
SELECT @@VERSION
```

## Failover (Chaos Engineering)

```
kubectl apply -f failover.yaml
```

```
SELECT ar.replica_server_name, hars.role_desc, hars.operational_state_desc
FROM sys.dm_hadr_availability_replica_states hars
JOIN sys.availability_replicas ar
ON hars.replica_id = ar.replica_id
GO
```
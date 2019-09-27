# Changing MSSQL Version of a Pod in Kubernetes

## Upgrade MSSQL Version
```
kubectl get pods -o wide --watch
```
```
kubectl apply -f mssql.yaml
POD=$(kubectl get pods | grep mssql-deployment | awk {'print $1'})
kubectl logs $POD -f
```

```
SELECT @@VERSION
```
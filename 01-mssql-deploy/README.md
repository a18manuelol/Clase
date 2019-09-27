# 01 - MSSQL Deployment in Kubernetes

## Deploy MSSQL

```
kubectl create secret generic mssql --from-literal=SA_PASSWORD="MyC0m9l&xP@ssw0rd"

kubectl apply -f pvc.yaml
kubectl describe pvc mssql-data

kubectl apply -f mssql.yaml
POD=$(kubectl get pods | grep mssql-deployment | awk {'print $1'})
kubectl get pods -o wide --watch
kubectl logs $POD -f

kubectl get services
```

```
SELECT @@VERSION
```

## Recovery

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

## Cleanup
```
kubectl delete -f .
```
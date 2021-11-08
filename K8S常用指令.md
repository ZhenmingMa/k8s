强制删除pod

```
kubectl delete pods <pod> --grace-period=0 --force
```
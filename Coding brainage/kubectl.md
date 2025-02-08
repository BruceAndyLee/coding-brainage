---
Status: Completed
tags:
 - devops
---
[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

Посмотреть namespace-ы

```Bash
kubectl get ns
```

Посмотреть поды внутри namespace-а

```Bash
kubectl get pods -n <namespace>
```

Посмотреть список сервисов внутри namespace-а

```Bash
kubectl get services -n <namespace>
```

Посмотреть логи пода внутри namespace

```Bash
kubectl logs <pod-id> -n <namespace> -f
```
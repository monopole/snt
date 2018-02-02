# The base app

> _Confirm that these resources are functional._
>
> _Time: 1m_

The resources fully define a runnable app.

To optionally confirm this, apply them to your cluster
directly and try a query.

<!-- @runKinflate @demo -->
```
kubectl apply -f $TUT_DAM
```

<!-- @query @demo -->
```
tut_query tut-service peach
```

Delete them, to clear the cluster for the next example.

<!-- @query @demo -->
```
kubectl delete --all deployment
kubectl delete --all service
kubectl delete --all configmap
```

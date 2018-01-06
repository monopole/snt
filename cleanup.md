# Clean up

For optional entertainment, wipe the entire namespace:

<!-- @deleteNamespace @test -->
```
kubectl delete namespace ns-beansprout
```

Stop the cluster.  On GKE, this stops billing.

<!-- @stopCluster @test -->
```
if tut_isMinikube; then
  $MINIKUBE_HOME/minikube stop
else
  gcloud --quiet container clusters \
      delete $TUT_CLUSTER_NAME
fi
```

Recover diskspace - or just wait, the system
will reclaim it eventually:

> ```
> if [ -n "$TUT_DIR" ]; then
>  rm -rf $TUT_DIR
> fi
> ```

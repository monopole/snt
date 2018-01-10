# Clean up

Optionally, wipe the entire namespace (merely to show
that this is possible):

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

Optionally, delete the tutorial work directory.

<!-- @wipeDirectory -->
```
if [ -d "$TUT_DIR" ]; then
  rm -rf $TUT_DIR
fi
```

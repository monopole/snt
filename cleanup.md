# Clean up

One can individually delete things, e.g.

```
kubectl delete deployment dep-kale
kubectl delete service svc-eggplant
```

or wipe the entire namespace:

<!-- @deleteNamespace -->
```
kubectl delete namespace ns-beansprout
```

Stop the cluster.

<!-- @stopCluster -->
```
if isMinikube; then
  minikube stop
else
  # This will stop billing.
  gcloud --quiet container clusters delete $TUT_CLUSTER_NAME
fi
```

Recover diskspace (or just wait - the system
will reclaim it eventually):

<!-- @rmTutDir -->
```
if [ -n "$TUT_DIR" ]; then
  rm -rf $TUT_DIR
fi
```

# Clean up

One can individually delete things, e.g.

```
kubectl delete namespace ns-beansprout
kubectl delete deployment dep-kale
```

or leverage the namespace:

<!-- @deleteTheNamespaceAndEverythingInIt -->
```
kubectl delete namespace ns-beansprout
```

If using GKE, delete the cluster to stop billing.

<!-- @deleteCluster -->
```
if ! isMinikube; then
  gcloud --quiet container clusters delete $TUT_CLUSTER_NAME
fi
```

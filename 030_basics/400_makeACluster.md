# Build a Cluster

Do the following once - it launches three nodes.

<!-- @createCluster -->
```
# Allocates three nodes by default.
gcloud container clusters create $TUT_CLUSTER_NAME
```

Confirm you have some nodes, but no pods.

<!-- @getNodes -->
```
kubectl get nodes
```

<!-- @getPods -->
```
kubectl get pods
```

That's it.

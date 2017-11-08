## Confirm kubectl can talk to your cluster

Regardless of which cluster you've set up,
you should now be able to talk to it via kubectl.

Confirm you have at least one node, but no pods.

<!-- @getNodes -->
```
kubectl get nodes
```

<!-- @getPods -->
```
kubectl get pods
```

Get the details:
```
kubectl describe nodes
```

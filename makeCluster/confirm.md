## Confirm kubectl can talk to your cluster

Regardless of which cluster you've set up,
you should now be able to talk to it via kubectl.

Confirm you have at least one node...

<!-- @getNodes -->
```
kubectl get nodes
```

...but no pods.
<!-- @getPods -->
```
kubectl get pods
```

Get node details:
```
kubectl describe nodes
```

Note the cpu and memory numbers in the capacity section.

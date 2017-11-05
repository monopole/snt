## Confirm kubectl can talk to your cluster

Regardless of which cluster you've set up,
you should now be able to talk to it via kubectl.

If you didn't install kubectl as a consequence of
installing gcloud, you can try


<!-- @optionallyInstallIndependentKubectl -->
```
apis=https://storage.googleapis.com
version=$(curl -s $apis/kubernetes-release/release/stable.txt)
curl -Lo $TUT_DIR/kubectl \
  $apis/kubernetes-release/release/$version/bin/linux/amd64/kubectl
chmod +x $TUT_DIR/kubectl
alias kubectl=$TUT_DIR/kubectl
```

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

# Define a namespace

[namespace]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces

The command blocks that follow create pods, services,
replicasets, ingress points, etc..  These resources
exist in a [namespace].

Namespaces divide the set of all resources into
subsets.  A command like `kubectl delete pod
shoeShopServer` or the more dramatic `kubectl delete
pods --all` implicitly operates only on the currently
active namespace, leaving pods in other namespaces
alone.

<!-- @getNamespaces -->
```
kubectl get namespaces
```

<!-- @getPodsInDifferentNamespaces -->
```
function showPods {
  echo "---- pods in namespace $1"
  kubectl --namespace=$1 get pods
  echo "---- "
  echo " "
}
showPods default
showPods kube-system
showPods kube-public
unset -f showPods
```

Labels offer another way to arrange resources into
sets, and it may be useful to imagine a namespace as an
implicitly declared label.

Conceptual differences:

* A default namespace exists (called
  `default`). There's no such thing as a default label.

* Kubectl commands operate in an implicit namespace via
  kubectl configuration.  There's no analogous
  functionality for a label.

* A resource has exactly one namespace, but can have
  any number of labels.

* The namespace (but no labels) appears in DNS entries
  for resources that have DNS entries (pods, services).

* A node can have labels, but does not have an
  associated namespace.


Your current namespace is:
<!-- @viewNamespace -->
```
kubectl config view | grep namespace:
```

Create a new namespace, but first delete it and every resource
in it:

<!-- @deleteNamespace -->
```
kubectl delete namespace $TUT_NAMESPACE_NAME
```

Create and switch to a new namespace
<!-- @createNamespace -->
```
kubectl create namespace $TUT_NAMESPACE_NAME
```

<!-- @changeDefaultNamespace -->
```
kubectl --namespace=$TUT_NAMESPACE_NAME \
    config set-context $(kubectl config current-context)
```

<!-- @viewNamespace -->
```
kubectl config view | grep namespace:
```

The rest of the commands will operate in this namespace.

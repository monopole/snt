# Isolate Apps with Namespaces

> _Isolate the tutorial work in your cluster._
>
> _Time: 2min_


[namespace]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces

Command blocks that follow create the pods, services,
replicasets, ingress points, etc, that comprise a
cluster app.  These resources exist in a [namespace].

A command like `kubectl delete pod shoeShopServer` or
the more dramatic `kubectl delete pods --all`
implicitly operates only on the currently active
namespace, leaving pods in other namespaces alone.

Namespaces isolate one app from another, one tenant
from another, etc.


## Review current status

<!-- @getNamespaces -->
```
kubectl get namespaces
```

<!-- @getPodsByNamespace -->
```
function tut_showPods {
  echo "---- pods in namespace $1"
  kubectl --namespace=$1 get pods
  echo "---- "
  echo " "
}
tut_showPods default
tut_showPods kube-system
tut_showPods kube-public
unset -f tut_showPods
```

Labels offer another way to partition resources.
Conceptual differences between labels and namespace:

* A default namespace exists (called
  `default`). There's no such thing as a default label.

* kubectl commands operate in an implicit namespace via
  kubectl configuration.  There's no analogous
  functionality for a label.

* A resource has one namespace, but any number of labels.

* The namespace (but no labels) appears in DNS entries
  for resources that have DNS entries (pods, services).

* A node can have labels, but does not have an
  associated namespace.


Your current namespace is:
<!-- @viewNamespace -->
```
kubectl config view | grep namespace:
```

## Make a Namespace for your App

First, delete anything created in a previous
pass through this tutorial:

<!-- @deleteNamespace -->
```
kubectl delete namespace ns-beansprout
```

Create and switch to a new namespace:
<!-- @createNamespace -->
```
kubectl create namespace ns-beansprout
```

<!-- @changeDefault -->
```
kubectl --namespace=ns-beansprout \
    config set-context $(kubectl config current-context)
```

<!-- @viewNamespace -->
```
kubectl config view | grep namespace:
```

The rest of the commands will operate in this
namespace, rather than in the `default` namespace.

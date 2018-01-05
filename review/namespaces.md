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

<!-- @getNamespaces @test -->
```
kubectl get namespaces
```

<!-- @funcShowPods @env @test -->
```
function tut_showPods {
  echo "---- pods in namespace $1"
  kubectl --namespace=$1 get pods
  echo "---- "
  echo " "
}
```

<!-- @showPodsByNs @test -->
```
tut_showPods default
tut_showPods kube-system
tut_showPods kube-public
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
<!-- @viewNamespace @test -->
```
kubectl config view | grep namespace: || true
```
There should be no output, implying you're in the default namespace.

## Make a Namespace for your App

Before making the namespace to use in this tutorial,
remove it if it already exists from a previous pass:

<!-- @deleteNamespace @test -->
```
set +e
exists=$(kubectl get namespace ns-beansprout 2> /dev/null)
if [ ! -z "$exists" ]; then
  kubectl delete --ignore-not-found namespace ns-beansprout
  exists=$(kubectl get namespace ns-beansprout 2> /dev/null)
  while [ ! -z "$exists" ]; do
     sleep 2
     exists=$(kubectl get namespace ns-beansprout 2> /dev/null)
  done
fi
tut_restoreErrorOnExit
```

Create and switch to a new namespace:
<!-- @createNamespace @test -->
```
kubectl create namespace ns-beansprout
```

<!-- @changeDefault @test -->
```
kubectl --namespace=ns-beansprout \
    config set-context $(kubectl config current-context)
```

<!-- @viewNamespace @test -->
```
kubectl config view | grep namespace: || true
```

The rest of the commands will operate in this
namespace, rather than in the `default` namespace.

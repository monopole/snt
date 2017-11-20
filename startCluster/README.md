# Starting a bare cluster

[minikube]: https://github.com/kubernetes/minikube/releases
[GKE]: https://cloud.google.com/container-engine

A k8s tutorial requires a working, albeit bare,
cluster.

This section has instructions for starting either

 * a locally hosted cluster on [minikube],
 * or a remote cluster hosted on [GKE].

Here, _starting_ a cluster means doing the minimal
setup necessary to bring up a k8s API server and nodes
on some hardware somewhere, with no actual services
running save those native to k8s.

_Configuring_ a cluster is bringing up and evolving
your own apps on that cluster.

### Which one?

Using a local cluster (minikube) will get you through
the material faster, as it avoids orthogonal detours
into cloud authentication and billing issues.

Try GKE to practice ingress configuration and launch
cluster-backed services accessible world-wide.

Short of ingress, the commands that configure the
cluster are the same.  k8s is a portable cloud.

### kubectl

Regardless of cluster (minikube, GKE, AWS, etc.)  one
needs `kubectl`, a command line client that facilitates
controlling a k8s cluster.

You'll install `kubectl` below, via means that
differ slightly depending on the cluster choice.

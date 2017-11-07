# Starting a bare cluster

[minikube]: https://github.com/kubernetes/minikube/releases
[GKE]: https://cloud.google.com/container-engine

A k8s tutorial requires a working, albeit bare,
cluster.  This section has instructions for starting

 * a locally hosted cluster on [minikube].
 * a remote cluster hosted on [GKE].

Here, _starting_ a cluster means doing the minimal
setup necessary to bring up a k8s API server and nodes
on some hardware somewhere, with no actual services
running save those native to k8s. _Configuring_ a
cluster is bringing up and evolving your own apps on
that cluster.

### which one?

Use a local minikube first, then try GKE when ready to
try ingress to a remote cloud.

A local cluster is much faster for what follows, as it
avoids orthogonal detours into cloud authentication and
billing issues.

### kubectl

Regardless of cluster (minikube, GKE, AWS, etc.)  one
needs `kubectl`, a command line client that facilitates
controlling a k8s cluster.

Instructions for installing `kubectl` are covered in
the platform-specific instructions that follows.

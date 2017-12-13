# Which Cluster?

> _Time: 1min_

[minikube]: https://github.com/kubernetes/minikube/releases
[GKE]: https://cloud.google.com/container-engine

Instructions follow to start either

 * a locally hosted cluster on [minikube],
 * or a remote cluster hosted on [GKE].

You'll do the minimal setup necessary to
be able to contact a k8s API server.

### Which one?

k8s is a portable cloud.  Short of
ingress (how one makes the cluster accessible to the
world), the commands that configure the cluster are
the same regardless of the platform underneath.

Using __minikube__ takes one through the material
faster, as it avoids detours into cloud authentication
and billing issues.  But it first requires installing
software necessary to run VMs on your laptop.

Using __GKE__ lets one practice ingress configuration
and launch cluster-backed services accessible
world-wide, even if for only a few minutes.  But it
requires a credit card and (possibly) billing that will
amount to less than a dollar if you complete the
instructions to turn down the cluster you create.

### kubectl

Regardless of cluster (minikube, GKE, AWS, azure, etc.)
one needs `kubectl`, a command line client that
facilitates controlling a k8s cluster.

You'll install `kubectl` below, via means that
differ slightly depending on the cluster choice.

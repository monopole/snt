# Starting a bare cluster

[minikube]: https://github.com/kubernetes/minikube/releases
[GKE]: https://cloud.google.com/container-engine

To _configure_ a cluster - bring up and evolve your
services on a cluster, one must first _start_ a cluster.

Starting a cluster is doing the minimal setup necessary
to bring up a k8s API server and nodes on some
hardware, with no actual services running save those
that comprise k8s itself.

This section has instructions for starting either

 * a locally hosted cluster on [minikube],
 * or a remote cluster hosted on [GKE].



### Which one?

Short of ingress (how one makes the cluster accessible
to the world), the commands that configure the cluster
are the same.  k8s is a portable cloud.

That said, using minikube will get you through
the material faster, as it avoids orthogonal detours
into cloud authentication and billing issues.

In a second pass, one could try GKE to practice ingress
configuration and launch cluster-backed services
accessible world-wide, even if for only a few minutes.

### kubectl

Regardless of cluster (minikube, GKE, AWS, etc.)  one
needs `kubectl`, a command line client that facilitates
controlling a k8s cluster.

You'll install `kubectl` below, via means that
differ slightly depending on the cluster choice.

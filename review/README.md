# Kubernetes Review

> _An overview of the concepts needed to discuss configuration._
>
> _Total Time: 30min_

You should now have a functional but otherwise empty
k8s cluster and two binary versions of `tuthello`
stored in a container registry.

The next sections deploy your server to the cluster
using manual, declarative configuration.  You'll
write raw YAML, and _apply_ that YAML to your cluster.

This defines a reference point for higher level
configuration techniques discussed later.

Even if you know k8s, please run through it to define
a namespace and some helper functions used later.

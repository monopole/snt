# Kubernetes Review

> _An overview of the concepts needed to discuss configuration._
>
> _Total Time: 5-30min_

At this point, you have a raw k8s cluster, and two
binary versions of `tuthello` stored in a container
registry.

The next sections deploy your server to the cluster
using manual, declarative configuration.  I.e.  you'll
write raw YAML files for everything, and use `kubectl`
to _apply_ that YAML to your cluster.

This sets the stage for higher level configuration
techniques discussed later.

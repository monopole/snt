# Kubernetes Review

> _An overview of the concepts needed to discuss configuration._
>
> _Total Time: 5-30min_

The next sections deploy the server built above to a
k8s cluster using manual, declarative configuration.

I.e.  you write raw YAML files for everything, and use
`kubectl` to _apply_ the YAML to your cluster.

This is necessary to understand and appreciate
higher level abstractions for cluster configuration.

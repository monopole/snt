# Declarative Application Management

> _Configure apps using recognizable
> k8s API types._
>
> _Time: 5m_

[here]: https://goo.gl/T66ZcD
[baseline]: /apps/examples/baseline
[helm]: /apps/examples/helm

The set of k8s API types (deployments, services,
configmaps, etc.), and the notion of _applying_
instantiations of these types to a cluster, have been
the two central themes in the previous examples of app
configuration ([baseline] and [helm]).

Declarative Application Management (DAM), described in
detail [here], seeks to add value to configuration
management without _masking or modifying_ these central
concepts.

DAM wants one to use literal resources, employing new
declarations to modify resources, rather than achieving
modification via templates.  It facilitates coupling
cluster state changes to version control commits.  It
encourages forking a configuration, customizing it,
rebasing to capture upgrades from the original
configuration.

## kinflate

`kinflate` is an implementation of DAM.

It reads an app definition from a directory of YAML
files, and creates new YAML suitable for directly
piping to `kubectl apply`.

Installation:

<!-- @installKinflate @test -->
```
GOBIN=$TUT_BIN go install k8s.io/kubectl/cmd/kinflate
```

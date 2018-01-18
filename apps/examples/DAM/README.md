# Declarative Application Management

> _Configure apps using recognizable
> k8s API types, eschewing complexity._
>
> _Time: 5m_

[declarative]: /apps/instances

The preferred approach to managing Kubernetes
applications is through [declarative] configuration.

Doing so, however, introduces new challenges, such as
how to reuse common pieces of configuration across
multiple apps and how to publish partially complete
configuration for other users to consume.

[predecessors]: https://research.google.com/pubs/pub44843.html

Attempts to address this (both in k8s, and its
[predecessors]) have used notions of templating,
inheritance, even Turing-complete configuration
languages. This adds orthogonal complexity, and
obscures the native API (if there is one).

[here]: https://goo.gl/T66ZcD

Declarative Application Management (DAM), described in
detail [here], seeks to manage app instances solely
through declarative techniques that directly use,
rather than attempt to wrap, the k8s API.

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

# kinflate

> _An implementation of
> [Declarative Application Management](https://goo.gl/T66ZcD)._
>
> _Time: 5m_


kinflate manages app instances solely through
declarative techniques that use the kubernetes API as
is, rather than hiding it behind an abstraction of
templates and code.

## Install to disk

<!-- @installKinflate @test -->
```
GOBIN=$TUT_BIN go install k8s.io/kubectl/cmd/kinflate
```

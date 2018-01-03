# Kubernetes Cluster Configuration

_A tutorial on installing customized cluster-level apps._

> __status: under development Nov 2017__

This tutorial requires no knowledge of kubernetes,
also known as _k8s_.

It focusses on command execution, eschewing deeper
discussion that can be found at https://kubernetes.io
and many other sites.

To establish context for the main topic, this tutorial
starts with a walk-through of manual configuration
(individual commands to create a pod, create a service,
etc.).  Context established, it compares different
approaches to app-level configuration.

## Prerequisites

[Go]: https://golang.org/doc/install
[curl]: https://github.com/curl/curl
[docker]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu

[Go], [curl], and [docker].

The bash shell is implicitly required, as the
command blocks use bash syntax.

Further requirements arise from the choice of where you
run your cluster (discussed next).

<!-- @checkPrerequisites @env @test -->
```
function tut_checkProgram {
  if ! type -P "$1" >/dev/null 2>&1; then
    echo Please install $1
  fi
}
tut_checkProgram go
tut_checkProgram curl
tut_checkProgram docker
```

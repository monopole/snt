# Kubernetes Cluster Configuration

_A tutorial on installing customized cluster-level apps._

> _Total Time: ~1hour_

> __status: under development Nov 2017__

This tutorial requires no knowledge of kubernetes,
also known as _k8s_.

It focusses on command execution, eschewing deeper
discussion that can be found at https://kubernetes.io
and many other sites.

To establish context for the main topic, this tutorial
begins with the creation of a service, then does a
walk-through of manual configuration of a cluster
offering the service.

Context established, the tutorial compares different
approaches to configuring _cluster apps_.

## Prerequisites

[Go]: https://golang.org/doc/install
[curl]: https://github.com/curl/curl
[docker]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu
[ubuntu]: https://www.ubuntu.com

[Go], [curl], and [docker].

The bash shell is implicitly required;
command blocks use bash syntax.

The tutorial is tested on [ubuntu], but
should work on any linux distro, or on OSX with
some command modification.

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

Further requirements (discussed shortly) arise from
your choice of cluster host.

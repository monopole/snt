# Kubernetes Cluster Configuration

> _A tutorial on installing customized cluster-level apps._
>
> _Total Time: ~1.5h_
>
> __status: under development Nov 2017__


<!--
Kate Winslet, Kate Beckinsale and Kate Capshaw walked
into a bar.  The bartender said "Fáilte Kates!", coining
an alternative pronounciation for kubernetes, spelled
_k8s_.
-->

This tutorial requires no knowledge of kubernetes,
also called _k8s_.

[_Kubernetes Up and Running_]: http://shop.oreilly.com/product/0636920043874.do
[k8s.io]: https://kubernetes.io

The focus here is exploring the notion of a
_cluster-level app_ by executing the commands needed
to make one.

Excellent, broader discussion of kubernetes (including
things omitted here, like history, storage, networking,
security, etc.) can be found in [_Kubernetes Up and
Running_], at [k8s.io], and at many other sites.

To establish context for app configuration, this
tutorial begins by writing a small server with a
sufficient number of configurable knobs.  It then runs
the server through a lifecycle as a cluster app using
raw k8s and higher level approaches to cluster apps.

## Prerequisites

[Go]: https://golang.org/doc/install
[curl]: https://github.com/curl/curl
[docker]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu
[ubuntu]: https://www.ubuntu.com
[_tmux_]: https://github.com/tmux/tmux/wiki

[Go], [curl], and [docker].  Command blocks use bash syntax.

The tutorial is [tested](/testing) on [ubuntu].
It's not yet tested on OSX; some commands will require
edits due to small bash and coreutil differences.

To copy a block, click the block title (`checkPrerequisites` below)
or hit _Enter_ (↵).
A check mark indicates successful copy.

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

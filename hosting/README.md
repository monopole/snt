# Where to Host Your k8s Cluster

> _Total Time: ~5min_

One needs a hosted cluster - raw k8s components running
on some set of real or virtual hardware - before one
can practice configuring it.

The process to configure the cluster is the same
regardless of where it is hosted, because k8s is a
portable cloud.

[hard way]: https://github.com/kelseyhightower/kubernetes-the-hard-way

You can obtain machines and do this the [hard way],
or you can use one of several existing hosted clusters.
Two choices are offered here.

## Host on your laptop

[Minikube](https://github.com/kubernetes/minikube/releases)
is an open source project that brings up a k8s cluster
on your laptop.

It avoids detours into cloud authentication and billing
issues, and is good investment as it lets you
experiment locally even after you start using corporate
clouds.

It requires installation of the software necessary to
run VMs on your laptop.

-> Choose __[minikube](/hosting/minikube)__.

## Host on Google Kubernetes Engine

Using [GKE](https://cloud.google.com/container-engine)
lets you practice ingress configuration and make your
tutorial app accessible world-wide, even if only for a
few minutes.  You can easily build from there.

It requires the installation of some Google tools, a
Google account and credit card. New users are given
free cloud credit, many times more than is necessary to
complete this tutorial.

-> Choose __[GKE](/appendix/GKE)__.

A sensible strategy is first try the tutorial locally
with minikube, then try it on GKE.

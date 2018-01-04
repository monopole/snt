# Where to Host Your k8s Cluster

> _Total Time: ~5min_

_Starting_ a cluster means marshalling the underlying
machines (real or virtual) needed to run raw k8s.

_Configuring_ a k8s cluster means talking to the k8s
API server to start and maintain your apps.

Short of ingress (how one makes the cluster accessible
to the world), the commands that configure the cluster
are the same regardless of where it is hosted, because
kubernetes is a portable cloud.

## Host on your laptop

[Minikube](https://github.com/kubernetes/minikube/releases)
allows one to run a k8s cluser on your laptop.

This avoids detours into cloud authentication and
billing issues, but requires installation of the
software necessary to run VMs on your laptop.

-> Choose __[minikube](/startCluster/minikube)__.

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

A reasonable strategy is try it locally, then try it on GKE.

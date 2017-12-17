# Where to Host Your k8s Cluster

> _Time: 1min_

_Starting_ a cluster means marshalling the underlying machines (real
or virtual) needed to run k8s.

_Configuring_ a k8s cluster means talking to the k8s API server
to start and maintain your apps.

Short of ingress (how one makes the cluster accessible to the world),
the commands that configure the cluster are the same regardless of
where it is hosted, because kubernetes is a portable cloud.

## Host on your laptop

[Minikube](https://github.com/kubernetes/minikube/releases) allows one
to run a k8s cluser on your laptop.

This avoids detours into cloud authentication and billing issues, but
does require installation of the software necessary to run VMs on your
laptop.

-> Choose __[minikube](/startCluster/minikube)__.

## Host on Google Kubernetes Engine

Using [GKE](https://cloud.google.com/container-engine) lets you
practice ingress configuration and make your tutorial app accessible
world-wide, even if only for a few minutes.  You can easily build from
there.

It requires a Google account and credit card, and the (well-tested)
installation of some Google programs.  New users are given more than
enough free credit to complete this tutorial.

-> Choose __[GKE](/startCluster/GKE)__.

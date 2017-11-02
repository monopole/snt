# Establish a cluster

Kubernetes runs on a variety of platforms.

Instructions follow for setting up

 * a locally hosted cluster (minikube),
 * a remote cluster hosted on GKE.

Regardless of platform, one needs `kubectl`
to talk to the cluster, and we're going to
use GCR for storing containers in the cloud
(pods grab their containers from the cloud),
so it's necessary to

[gcloud downloads]: https://cloud.google.com/sdk/downloads#versioned
[gcloud sdk]: https://cloud.google.com/sdk/
[billing]: https://support.google.com/cloud/answer/6158867?hl=en

 * Install [gcloud sdk].  See also [gcloud downloads] site.
 * Install `kubectl` via `gcloud components install kubectl`.
 * Enable [billing] for a GCP project.

Assign the following variable to a billable
GCP project ID so we can use Google's
container repo API (and GKE generally if
using that instead of minikube).

<!-- @initializeProjectId -->
```
TUT_PROJECT_ID=lyrical-gantry-618
```

One can install `kubectl` without gcloud using

<!-- @optionallyInstallIndependentKubectl -->
```
apis=https://storage.googleapis.com
version=$(curl -s $apis/kubernetes-release/release/stable.txt)
curl -Lo $TUT_DIR/kubectl \
  $apis/kubernetes-release/release/$version/bin/linux/amd64/kubectl
chmod +x $TUT_DIR/kubectl
alias kubectl=$TUT_DIR/kubectl
```

but it's not clear that's useful given one needs gcloud anyway for GCR.

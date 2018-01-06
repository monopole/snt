# Start a GKE Cluster

> _Run this tutorial on Google's cloud instead of minikube._
>
> _Time: 5min (assuming prerequisites)_

[minikube]: /hosting/minikube

__This is a drop-in replacement for the
[minikube] section, to let one run the tutorial
on Google Kubernetes Engine rather than minikube.__

__Go to [confirmation](/hosting/confirm) if you
just set up a minikube cluster and came here by mistake.__

__Warning: Not under CI/CD testing (due to auth and billing issues).__

## Prerequisites

[gcloud downloads]: https://cloud.google.com/sdk/downloads#versioned
[gcloud]: https://cloud.google.com/sdk/
[Enable billing]: https://support.google.com/cloud/answer/6158867?hl=en

 * Install [gcloud].
 * [Enable billing].

Pick any name for the cluster made below.
The name is used in project billing.
<!-- @nameTheCluster -->
```
TUT_CLUSTER_NAME=cluster-spinach
```

### Install open-source `kubectl`

<!-- @initKubeConfig -->
```
# Don't want to stomp on your normal config
export KUBECONFIG=$HOME/.kube/tut-gke-config
rm -f $KUBECONFIG
```

<!-- @initKubeCtl -->
```
gcloud components install kubectl
```

Confirm that `gcloud` and `kubectl` are on your path
and are the version you want (i.e. not some local
version you were hacking on):

<!-- @whichPrograms -->
```
which kubectl
which gcloud
```

Failing that, try something like this to
get them on your `PATH`:

<!-- @setGCloudEnv -->
```
tmp=$HOME/google-cloud-sdk
if [ -d "$tmp" ]; then
  source $tmp/path.bash.inc
  source $tmp/completion.bash.inc
fi
unset tmp
```

Choose your cloud identity and project ID.
It should be a billable project ID.
<!-- @cloudIdentity -->
```
gcloud config set account <your google account email address>
TUT_PROJECT_ID=<your google project ID, e.g. colorful-boat-918>
```

Complete setup, login and confirm your settings:

<!-- @completeSetup -->
```
gcloud config set project $TUT_PROJECT_ID
gcloud config set compute/region us-west1
gcloud config set compute/zone us-west1-a
```

<!-- @login -->
```
gcloud auth application-default login
```

<!-- @confirmConfig -->
```
gcloud config list
```

Do the following once - it launches three nodes by default.

<!-- @createCluster -->
```
gcloud container clusters create $TUT_CLUSTER_NAME
```

Now proceed to [confirmation](/hosting/confirm)
and continue the tutorial.

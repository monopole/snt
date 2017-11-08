# GKE cluster

[gcloud downloads]: https://cloud.google.com/sdk/downloads#versioned
[Install gcloud]: https://cloud.google.com/sdk/
[Enabled billing]: https://support.google.com/cloud/answer/6158867?hl=en

Prerequisites:

 * [Install gcloud].  See also [gcloud downloads] site.
 * [Enabled billing] for a GCP project.

Set this variable to match one of your actual,
billable projects.

<!-- @initializeKubeConfig -->
```
TUT_PROJECT_ID=lyrical-gantry-618

# Arbitrary name for the cluster made below.
TUT_CLUSTER_NAME=cluster-spinach
```
### Install `kubectl`



<!-- @initializeKubeConfig -->
```
# Don't want to stomp on your normal config
export KUBECONFIG=$HOME/.kube/tut-gke-config
rm -f $KUBECONFIG
```

<!-- @initializeKubeCtl -->
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

<!-- @useConsumerCloudEnv -->
```
tmp=$HOME/google-cloud-sdk
if [ -d "$tmp" ]; then
  source $tmp/path.bash.inc
  source $tmp/completion.bash.inc
fi
unset tmp
```

Choose your cloud identity and project.

<!-- @chooseCloudIdentity -->
```
gcloud config set account jregan@google.com
gcloud config set account jeff.regan@gmail.com
TUT_PROJECT_ID=svc-cat-tangle
TUT_PROJECT_ID=lyrical-gantry-618
```

Complete setup, login and confirm your settings:

<!-- @completeConfigSetup -->
```
gcloud config set project $TUT_PROJECT_ID
gcloud config set compute/region us-west1
gcloud config set compute/zone us-west1-a
```

<!-- @login -->
```
gcloud auth application-default login
```

<!-- @confirmCloudConfig -->
```
gcloud config list
```

Do the following once - it launches three nodes by default.

<!-- @createCluster -->
```
gcloud container clusters create $TUT_CLUSTER_NAME
```

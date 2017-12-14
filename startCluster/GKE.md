# ...or Start a GKE Cluster

> _Do the prep work to run this tutorial on Google's cloud,
> allowing one to adapt the work
> The setup can then be adapted to greater purpose._
>
> _Time: 5min (assuming prerequisites)_


[gcloud downloads]: https://cloud.google.com/sdk/downloads#versioned
[Install gcloud]: https://cloud.google.com/sdk/
[Enabled billing]: https://support.google.com/cloud/answer/6158867?hl=en

Prerequisites:

 * [Install gcloud].
 * [Enabled billing] for a GCP project.

Set this variable to match one of your actual, billable projects.

<!-- @existingProjectId -->
```
TUT_PROJECT_ID=lyrical-gantry-618
```

Pick any name for the cluster made below.
The name is used in project billing.
<!-- @nameTheCluster -->
```
TUT_CLUSTER_NAME=cluster-spinach
```

### Install `kubectl`



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

Choose your cloud identity and project.

<!-- @cloudIdentity -->
```
gcloud config set account jregan@google.com
gcloud config set account jeff.regan@gmail.com
TUT_PROJECT_ID=svc-cat-tangle
TUT_PROJECT_ID=lyrical-gantry-618
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

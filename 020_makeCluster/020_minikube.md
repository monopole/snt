# Minikube cluster

This cluster uses cores on your local machine.

Follow official install instructions at
https://github.com/kubernetes/minikube, or try what follows.

The tutorial stores docker images in Google's
cloud, so until that changes you'll still
need to install the [gcloud] program to upload the
container images.

[gcloud]: https://cloud.google.com/sdk/


<!-- @removeOldMinikube -->
```
/bin/rm -rf  $HOME/.minikube/
mv -f $HOME/bin/minikube
```

<!-- @installLatest -->
```
apis=https://storage.googleapis.com
curl -Lo $HOME/bin/minikube \
    $apis/minikube/releases/latest/minikube-linux-amd64
chmod +x $HOME/bin/minikube
```

<!-- @confirmVersionAndPath -->
```
$HOME/bin/minikube version
minikube version
```

<!-- @initializeKubeConfig -->
```
# Don't want to stomp on your normal config
export KUBECONFIG=$HOME/.kube/tut-minikube-config
rm -f $KUBECONFIG
```

<!-- @defineOtherMiniKubeEnvVars -->
```
# Where .minikube directory will live
export MINIKUBE_HOME=$HOME

# See comments around use of the following variable at
# https://github.com/kubernetes/minikube/blob/master/cmd/minikube/cmd/start.go#L315
export CHANGE_MINIKUBE_NONE_USER=true
```

<!-- @defineProjectId -->
```
# Used for making container images
export TUT_PROJECT_ID=minikube
```

<!-- @startTheCluster -->
```
sudo -E $HOME/bin/minikube start --vm-driver=none
```

Wait for it if need be:
```
function awaitMinikube {
  for i in {1..150}; do # timeout for 5 minutes
    kubectl get pods &> /dev/null
    if [ $? -ne 1 ]; then
      break
    fi
    sleep 2
  done
}
awaitMinikube

```

<!-- @confirmMinikubeRunning -->
```
minikube status
```

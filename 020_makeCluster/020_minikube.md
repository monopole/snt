# Minikube cluster

This cluster uses cores on your local machine.

Follow official install instructions at
https://github.com/kubernetes/minikube, or try what follows.

The tutorial (at the moment) stores docker images in
Google's cloud.  Until that changes you'll still need
to install the [gcloud] program to upload the container
images.

[gcloud]: https://cloud.google.com/sdk/


<!-- @possiblyCleanUpPreviousVmUsage -->
```
vboxmanage list vms
# vboxmanage startvm minikube --type emergencystop
# vboxmanage unregistervm {idFromPreviousCommand}  -delete
```

<!-- @removeOldMinikubeState -->
```
# Where .minikube directory will live
export MINIKUBE_HOME=$TUT_DIR
rm -rf $MINIKUBE_HOME/.minikube
```

<!-- @overrideKubeConfigAndWipeIt -->
```
# Don't stomp on your existing kube config (if any)
export KUBECONFIG=$TUT_DIR/.kube/tut-minikube-config
rm -f $KUBECONFIG
```

```
cat $KUBECONFIG
```


<!-- @installLatest -->
```
apis=https://storage.googleapis.com
curl -Lo $MINIKUBE_HOME/minikube \
    $apis/minikube/releases/latest/minikube-linux-amd64
chmod +x $MINIKUBE_HOME/minikube
alias minikube=$MINIKUBE_HOME/minikube
```

<!-- @confirmVersionAndPath -->
```
which minikube
minikube version
```

<!-- @defineOtherMiniKubeEnvVars -->
```
# Suppress prompts to report error messages.
export MINIKUBE_WANTREPORTERRORPROMPT=false

# See comments around use of the following variable at
# https://github.com/kubernetes/minikube/blob/master/cmd/minikube/cmd/start.go#L315
export CHANGE_MINIKUBE_NONE_USER=true
```

<!-- @startTheClusterOnVirtualBox -->
```
# sudo -E $HOME/bin/minikube start --vm-driver=none
minikube start --vm-driver=virtualbox
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

<!-- @examineKubeConfig -->
```
cat $KUBECONFIG
```

<!-- @checkFirewall -->
```
sudo iptables -L INPUT
```

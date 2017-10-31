# Minikube cluster

This cluster uses cores on your local machine.

Follow official install instructions at
https://github.com/kubernetes/minikube, or try what follows.

<!-- @removeOldMinikube -->
```
/bin/rm -rf  $HOME/.minikube/
mv -f $HOME/bin/minikube
```

<!-- @installLatest -->
```
curl -Lo $HOME/bin/minikube \
    https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x $HOME/bin/minikube
```

<!-- @confirmVersionAndPath -->
```
$HOME/bin/minikube version
minikube version
```

<!-- @defineClusterEnvVars -->
```
export KUBECONFIG=$HOME/.kube/minikube-config
rm -f $KUBECONFIG
export MINIKUBE_HOME=$HOME
export CHANGE_MINIKUBE_NONE_USER=true
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

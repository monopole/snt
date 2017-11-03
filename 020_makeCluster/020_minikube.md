# Minikube cluster

[installation instructions]: https://github.com/kubernetes/minikube
[virtualbox]: https://www.virtualbox.org/

This cluster uses cores on your local machine.

What follows is a condensed and tested version of
official minikube [installation instructions], with some initial
cleanup operations.

These instructions work on ubuntu using minikube with
the [virtualbox] VM.  minikube's local option
(`--vm-driver=none`) is awesome but requires `sudo`, so
not using it here as it would complicate testing.


<!-- @possiblyCleanUpPreviousVmUsage -->
```
for line in $(vboxmanage list vms); do
  if [[ $line =~ .*\{[0-9a-f\-]+\}.* ]]; then
    id=$(echo $line | sed 's/.*{\(.*\)}.*/\1/')
    vboxmanage unregistervm $id --delete
  fi
done
# vboxmanage startvm minikube --type emergencystop
```

<!-- @removeOldMinikubeState -->
```
# Where .minikube directory will live
export MINIKUBE_HOME=$TUT_DIR/mk
rm -rf $MINIKUBE_HOME/.minikube
mkdir -p $MINIKUBE_HOME
```

<!-- @overrideKubeConfigAndWipeIt -->
```
# Don't stomp on your existing kube config (if any)
export KUBECONFIG=$TUT_DIR/tut-minikube-config
rm -f $KUBECONFIG
```

<!-- @installLatest -->
```
apis=https://storage.googleapis.com
curl -Lo $MINIKUBE_HOME/minikube \
    $apis/minikube/releases/latest/minikube-linux-amd64
chmod +x $MINIKUBE_HOME/minikube
alias minikube=$MINIKUBE_HOME/minikube
find $MINIKUBE_HOME
```

<!-- @confirmVersionAndPath -->
```
which minikube  # expect nothing, since aliasing
minikube version
```

<!-- @defineOtherMiniKubeEnvVars -->
```
# Suppress prompts to report error messages.
export MINIKUBE_WANTREPORTERRORPROMPT=false

# See
# https://github.com/kubernetes/minikube/blob/master/cmd/minikube/cmd/start.go#L315
export CHANGE_MINIKUBE_NONE_USER=true
```

<!-- @startTheClusterOnVirtualBox -->
```
# sudo -E minikube start --vm-driver=none
minikube start --vm-driver=virtualbox
```

<!-- @optionallyWaitForFullMinikubeStartup -->
```
function awaitMinikube {
  for i in {1..150}; do # timeout for 5 minutes
    kubectl get pods &> /dev/null
    local foo=$?
    if [ $foo -eq 0 ]; then
      break
    fi
    echo "sleep 2"
    sleep 2
  done
}
awaitMinikube
```

Confirm expectations

<!-- @confirmMinikubeRunning -->
```
minikube status
find $MINIKUBE_HOME
```

<!-- @examineKubeConfig -->
```
cat $KUBECONFIG
```

<!-- @optionallyObserveFirewallChanges -->
```
sudo iptables -L INPUT
```

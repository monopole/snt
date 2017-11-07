# Minikube cluster

Prerequisites:

 * linux (tested on ubuntu).  Should work on OSX with
   some modification.
 * Install [virtualbox]

[here]: https://github.com/kubernetes/minikube
[virtualbox]: https://www.virtualbox.org/

These instructions (see more [here]) are tested to work
on ubuntu.

minikube's `--vm-driver=none` provides access to all
your local true cores, but it requires `sudo`, so not
using it here to avoid password prompts in testing.

### Define a project ID

The GKE instructions require a project ID in which to
create cloud containers and clusters for billing.  When
using minikube, a project ID isn't needed, but the env
variable is assigned regardless so the same tutorial
code blocks (for naming container images and such) can
be used regardless of cluster.

<!-- @initializeProjectId -->
```
TUT_PROJECT_ID=mk-project
```

### Clean up from last minikube run (if any)

<!-- @possiblyCleanUpPreviousVmUsage -->
```
for line in "$(vboxmanage list vms)"; do
  echo $line
  if [[ $line =~ .*minikube.*\{[0-9a-f\-]+\}.* ]]; then
    id=$(echo $line | sed 's/.*{\(.*\)}.*/\1/')
    echo "powering down $id"
    vboxmanage controlvm $id poweroff
    sleep 4
    vboxmanage unregistervm $id --delete
  fi
done
vboxmanage list vms
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

### Install minikube

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


### Install kubectl

Manually install the latest version of `kubectl`.

<!-- @installKubectl -->
```
apis=https://storage.googleapis.com
version=$(curl -s $apis/kubernetes-release/release/stable.txt)
mkdir -p $TUT_DIR/bin
curl -Lo $TUT_DIR/bin/kubectl \
  $apis/kubernetes-release/release/$version/bin/linux/amd64/kubectl
chmod +x $TUT_DIR/bin/kubectl
alias kubectl=$TUT_DIR/bin/kubectl
```


### Start the cluster

This can take a few minutes.

<!-- @startTheClusterOnVirtualBox -->
```
# sudo -E minikube start --vm-driver=none
minikube start --memory 8192 --cpus 6 --vm-driver=virtualbox
```

<!-- @optionallyWaitForFullMinikubeStartup -->
```
function awaitMinikube {
  for i in {1..150}; do # timeout for 5 minutes
    kubectl get nodes &> /dev/null
    local foo="$?"
    if [ $foo -eq 0 ]; then
      return
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
```

<!-- @examineKubeConfig -->
```
cat $KUBECONFIG
```

<!-- @optionallyObserveFirewallChanges -->
```
sudo iptables -L INPUT
```

# Start a Minikube cluster

Time required: ~5 mins

Prerequisites:

 * linux. Should work on OSX with some modification.
 * Install [virtualbox] for use as minikube's vmdriver.

minikube's `--vm-driver=none` provides access to all
your local true cores, but it requires `sudo`.  That
complicates testing these scripts, so using the
virtualbox hypervisor instead.

[here]: https://github.com/kubernetes/minikube
[virtualbox]: https://www.virtualbox.org/


### Define a project ID

The "GKE path" through these instructions require a
project ID in which to create cloud containers and
clusters for billing.

When using minikube, a project ID isn't needed, but the
env variable is assigned regardless so the same
tutorial code blocks (for naming container images and
such) can be used regardless of cluster.

<!-- @initializeProjectId -->
```
TUT_PROJECT_ID=mk-project
```

### Clean up from previous runs (if any)

<!-- @purgePreviousMinikubeVmUsage -->
```
function purgePreviousMinikubeVmUsage {
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
}
purgePreviousMinikubeVmUsage
# confirm no minikube vms
vboxmanage list vms
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
export KUBECONFIG=$MINIKUBE_HOME/tut-minikube-config
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

Install `kubectl` before starting `minikube` to be
ready to talk to it.

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
  for i in {1..100}; do
    kubectl get nodes &> /dev/null
    local foo="$?"
    if [ $foo -eq 0 ]; then
      echo "k8s API appears to be up."
      return
    fi
    echo "sleeping"
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

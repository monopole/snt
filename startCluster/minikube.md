# Start a Minikube Cluster

> _Create a multi-pod cluster on your laptop._
>
> _Time: 5min_

Prerequisites:

 * Linux.  It will also work on OSX with some modification
   (TODO: OS detection/branching)
 * Install [virtualbox] for use as minikube's vmdriver.
 * Install [docker].  Google for latest advice for your OS.

minikube's flag `--vm-driver=none` provides access to
all your local true cores, but it requires `sudo`.
Root access complicates testing this tutorial, so the
following uses the virtualbox hypervisor instead.

[here]: https://github.com/kubernetes/minikube
[virtualbox]: https://www.virtualbox.org/
[docker]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/

### Define environment

<!-- @defineEnv @test @debug -->
```
# Where .minikube directory will live
export MINIKUBE_HOME=$TUT_DIR/mk

# k8s/minikube stores its config here.
export KUBECONFIG=$MINIKUBE_HOME/tut-minikube-config

# Suppress prompts to report error messages.
export MINIKUBE_WANTREPORTERRORPROMPT=false

# See
# https://github.com/kubernetes/minikube/
#    blob/master/cmd/minikube/cmd/start.go#L315
export CHANGE_MINIKUBE_NONE_USER=true

export TUT_BIN=$TUT_DIR/bin
PATH=$TUT_BIN:$PATH
```

### Clean up VMs

Optionally remove VMs (if any) from previous runs through the tutorial.

<!-- @purgePrevMk @test -->
```
function tut_purgePrevVmUsage {
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
tut_purgePrevVmUsage
```

Check your list of VMs:

<!-- @listVms @test @debug -->
```
vboxmanage list vms
```

<!-- @removeOldMkState @test -->
```
rm -f $KUBECONFIG
rm -rf $MINIKUBE_HOME/.minikube
mkdir -p $MINIKUBE_HOME
```

### Install minikube

<!-- @installMk @test -->
```
apis=https://storage.googleapis.com
curl --fail --location --silent \
    --output $MINIKUBE_HOME/minikube \
    $apis/minikube/releases/latest/minikube-linux-amd64
```

<!-- @mkMinikubeExecutable @test -->
```
chmod +x $MINIKUBE_HOME/minikube
```

<!-- @confirmVersion @test @debug -->
```
$MINIKUBE_HOME/minikube version
```

### Install open-source kubectl

Install `kubectl` before starting `minikube` to be
ready to talk to it.

<!-- @downloadKubectl @test -->
```
mkdir -p $TUT_BIN
apis=https://storage.googleapis.com
version=$(curl -s $apis/kubernetes-release/release/stable.txt)
curl --fail --location --silent \
  --output $TUT_BIN/kubectl \
  $apis/kubernetes-release/release/$version/bin/linux/amd64/kubectl
```

<!-- @mkKubectlExecutable @test -->
```
chmod +x $TUT_BIN/kubectl
```

### Start the cluster

This downloads an OS image, and can thus take many
minutes.

<!-- @startClusterOnMk @test @debug -->
```
# sudo -E minikube start --vm-driver=none
$MINIKUBE_HOME/minikube \
    start --memory 8192 --cpus 6 \
    --vm-driver=virtualbox >& /dev/null
```

Assure that minikube is up.

<!-- @funcToWaitForIt @test @debug -->
```
function tut_awaitMk {
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
```

<!-- @waitForIt @test @debug -->
```
tut_awaitMk
```

Confirm expectations

<!-- @confirmUp @test @debug -->
```
$MINIKUBE_HOME/minikube status
```

The above steps have stored some state
in your `KUBECONFIG` file:

<!-- @catKubeConfig @test -->
```
printf "=====\n %s \n======\n" "$KUBECONFIG"
cat $KUBECONFIG
```

Confirm versions of client and server:

<!-- @kubectlVersion @test @debug -->
```
$TUT_BIN/kubectl version
```

If this spits out a version for the client and server,
you've got a cluster.  Proceed to
[confirm](/startCluster/confirm) step.

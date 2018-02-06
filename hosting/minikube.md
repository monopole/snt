# Minikube

> _Create a multi-pod cluster on your laptop._
>
> _Time: 5m_

## Prerequisites

[virtualbox]: https://www.virtualbox.org/

* [virtualbox] for use as minikube's vmdriver.

minikube's flag `--vm-driver=none` provides access to
all your local true cores, but it requires `sudo`.
Root access complicates testing this tutorial, so the
following uses the virtualbox hypervisor instead.

<!-- @checkPrerequisites @test -->
```
tut_checkProgram vboxmanage
```

## Define environment

<!-- @env @test -->
```
# Where .minikube directory will live
export MINIKUBE_HOME=$TUT_DIR/mk

# k8s/minikube stores its config here.
export KUBECONFIG=$MINIKUBE_HOME/tut-minikube-config

# Suppress prompts to report error messages.
export MINIKUBE_WANTREPORTERRORPROMPT=false

# See https://github.com/kubernetes/minikube/
#        blob/master/cmd/minikube/cmd/start.go#L315
export CHANGE_MINIKUBE_NONE_USER=true
```

## Clean up VMs

If minikube installed, ask it to stop:
<!-- @stopPrevMk @test -->
```
if type -P $MINIKUBE_HOME/minikube >/dev/null 2>&1
then
  $MINIKUBE_HOME/minikube status
  $MINIKUBE_HOME/minikube stop
fi
```

To enter a known state, delete vestigial VMs from previous runs:
<!-- @funcToPurgePrevMk @env @test -->
```
function tut_purgePrevVmUsage {
  set +e  # ignore errors
  vboxmanage list vms |
  while IFS= read -r line; do
    local id=$(echo $line | sed 's/.*{\(.*\)}.*/\1/')
    if [[ $line =~ .*minikube.* ]]; then
      local id=$(echo $line | sed 's/.*{\(.*\)}.*/\1/')
      echo "Powering down minikube VM $id"
      vboxmanage controlvm $id poweroff >& /dev/null
      sleep 8
      vboxmanage unregistervm $id --delete >& /dev/null
    fi
    if [[ $line =~ .*\<inaccessible\>.* ]]; then
      echo "Deleting inaccessible VM $id"
      vboxmanage unregistervm $id --delete >& /dev/null
    fi
  done
  tut_restoreErrorOnExit
}
```

<!-- @doPurgePrevMk @test -->
```
tut_purgePrevVmUsage
```

Confirm no running VMs:

<!-- @listVms @test -->
```
vboxmanage list vms
```

<!-- @removeOldMkState @test -->
```
rm -f $KUBECONFIG
rm -rf $MINIKUBE_HOME
mkdir -p $MINIKUBE_HOME
```

## Install minikube

<!-- @funcInstallMk @env @test -->
```
function tut_installMk {
  local v=latest
  v=v0.24.1 # Pick one, rather than use latest.
  local apis=https://storage.googleapis.com
  local target=$apis/minikube/releases/$v/minikube-linux-amd64
  echo Downloading $target
  curl --fail --location --silent \
      --output $MINIKUBE_HOME/minikube $target
  chmod +x $MINIKUBE_HOME/minikube
}
```

<!-- @installMk @test -->
```
if ! type -P $MINIKUBE_HOME/minikube >/dev/null 2>&1
then
  tut_installMk
fi
```

<!-- @confirmVersion @test -->
```
$MINIKUBE_HOME/minikube version
```

## Install kubectl

Install `kubectl` before starting `minikube` to be
ready to talk to it.

<!-- @downloadKubectl @env @test -->
```
function tut_installKubectl {
  local apis=https://storage.googleapis.com
  # local v=$(curl -s \
  #  $apis/kubernetes-release/release/stable.txt)
  local v=v1.9.2 # Pick one, rather than use latest.
  local path=release/$v/bin/linux/amd64/kubectl
  local target=$apis/kubernetes-release/$path
  echo Downloading $target
  curl --fail --location --silent \
     --output $TUT_BIN/kubectl $target
  chmod +x $TUT_BIN/kubectl
}
```

<!-- @funcInstallKubectl @test -->
```
if ! type -P $TUT_BIN/kubectl >/dev/null 2>&1
then
  tut_installKubectl
fi
```


## Start the cluster


<!-- @stopIfRunning @test -->
```
if $MINIKUBE_HOME/minikube status  >/dev/null 2>&1
then
  echo Stopping minikube...
  $MINIKUBE_HOME/minikube stop
fi
```


Start minikube, piping download progress meter to
`/dev/null`.
This downloads an OS image; it's the slowest step in the tutorial.


<!-- @startMkCluster @test -->
```
$MINIKUBE_HOME/minikube \
    start --memory 8192 --cpus 6 \
    --vm-driver=virtualbox > /dev/null
```

Assure that minikube is up:

<!-- @waitForIt @test -->
```
tut_retry 5 kubectl get nodes
```

Print status:

<!-- @confirmUp @test @sleep-->
```
$MINIKUBE_HOME/minikube status
```

The steps above have stored state
in your `KUBECONFIG` file.
Optionally, take a look:

<!-- @catKubeConfig @test -->
```
printf "=========\n %s \n=========\n" "$KUBECONFIG"
cat $KUBECONFIG
```

Confirm versions of client and server:

<!-- @kubectlVersion @test -->
```
$TUT_BIN/kubectl version
```

If this spits out a version for the client and server,
you've got a cluster.  Proceed to
[confirm](/hosting/confirm) step.

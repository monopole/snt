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

Delete vestigial VMs:
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

Check your list of VMs:

<!-- @listVms @test -->
```
vboxmanage list vms
```

<!-- @removeOldMkState @test -->
```
rm -f $KUBECONFIG
rm -rf $MINIKUBE_HOME/.minikube
mkdir -p $MINIKUBE_HOME
```

## Install minikube

<!-- @funcInstallMk @env @test -->
```
function tut_installMk {
  local apis=https://storage.googleapis.com
  echo Downloading minikube...
  curl --fail --location --silent \
      --output $MINIKUBE_HOME/minikube \
      $apis/minikube/releases/latest/minikube-linux-amd64
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
  local version=$(curl -s \
      $apis/kubernetes-release/release/stable.txt)
  local path=release/$version/bin/linux/amd64/kubectl
  echo Downloading kubectl...
  curl --fail --location --silent \
     --output $TUT_BIN/kubectl \
     $apis/kubernetes-release/$path
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

This downloads an OS image; it's the slowest step in the tutorial.

<!-- @stopMkCluster @test -->
```
if $MINIKUBE_HOME/minikube status  >/dev/null 2>&1
then
  echo Stopping minikube...
  $MINIKUBE_HOME/minikube stop
fi
```

Start minikube, piping download progress meter to
`/dev/null`:

<!-- @startMkCluster @test -->
```
$MINIKUBE_HOME/minikube \
    start --memory 8192 --cpus 6 \
    --vm-driver=virtualbox > /dev/null
```

Assure that minikube is up.

<!-- @waitForIt @test -->
```
tut_retry 4 kubectl get nodes &> /dev/null
```

Print status:

<!-- @confirmUp @test @sleep-->
```
$MINIKUBE_HOME/minikube status
```

The above steps have stored some state
in your `KUBECONFIG` file:

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

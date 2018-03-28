# minikube on top of kvm2 (ubuntu/debian)

Install and configure kvm2 dependencies:
```
sudo apt-get install -y \
    libvirt-clients libvirt-daemon-system qemu-kvm
```

```
sudo usermod -a -G libvirt $USER
sudo virsh net-autostart default
sudo virsh net-start default
newgrp libvirt
```

Define the minikube environment:
```
# Where .minikube directory will live
export MINIKUBE_HOME=~/minikube

# k8s/minikube stores its config here.
export KUBECONFIG=$MINIKUBE_HOME/tut-minikube-config

# Suppress prompts to report error messages.
export MINIKUBE_WANTREPORTERRORPROMPT=false

# See https://github.com/kubernetes/minikube/
#        blob/master/cmd/minikube/cmd/start.go#L315
export CHANGE_MINIKUBE_NONE_USER=true
```

If minikube installed, ask it to stop:
<!-- @stopPrevMk @test -->
```
if type -P $MINIKUBE_HOME/minikube >/dev/null 2>&1
then
  $MINIKUBE_HOME/minikube status
  $MINIKUBE_HOME/minikube stop
fi
```

<!-- @removeOldMkState @test -->
```
rm -f $KUBECONFIG
rm -rf $MINIKUBE_HOME
mkdir -p $MINIKUBE_HOME
```

<!-- @funcInstallMk @env @test -->
```
function tut_installMk {
  v=v0.25.0 # Pick one, rather than use latest.
  local apis=https://storage.googleapis.com
  local target=$apis/minikube/releases/$v/minikube-linux-amd64

  local mkhost=https://storage.googleapis.com/minikube
  curl -LO ${mkhost}/releases/v0.25.0/docker-machine-driver-kvm2
  curl -Lo minikube ${mkhost}/v0.25.0/latest/minikube-linux-amd64

  echo Downloading $target



   curl --fail --location --silent \
      --output $MINIKUBE_HOME/minikube $target
  chmod +x $MINIKUBE_HOME/minikube
}
```

Install minikube and kvm2 driver:
```
local mkhost=https://storage.googleapis.com/minikube
curl -LO ${mkhost}/releases/v0.25.0/docker-machine-driver-kvm2
curl -Lo minikube ${mkhost}/v0.25.0/latest/minikube-linux-amd64
chmod +x docker-machine-driver-kvm2 minikube
sudo mv docker-machine-driver-kvm2 minikube /usr/local/bin
```


Configure minikube to use kvm2.

Must be rerun if ~/.minikube i

```
minikube config set vm-driver kvm2
minikube config set bootstrapper kubeadm
```


# test the setup

```
minikube start
```

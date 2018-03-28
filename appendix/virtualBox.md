# Install virtual box on ubuntu

Hey

```
more /etc/lsb-release
sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get autoremove
sudo apt-get -y install gcc make linux-headers-$(uname -r) dkms
```

```
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

```
sudo sh -c 'echo "deb http://download.virtualbox.org/virtualbox/debian $(lsb_release -sc) contrib" >> /etc/apt/sources.list'
```

# Virtual Box installation on ubuntu

For reference, note your distribution version:
```
more /etc/lsb-release
```

Get up to date on packages:
```
sudo apt-get update && \
    sudo apt-get dist-upgrade && \
    sudo apt-get autoremove
```

Get up to date on headers:
```
sudo apt-get -y install \
    gcc make linux-headers-$(uname -r) dkms
```

Download and install package keys:
```
vbox=https://www.virtualbox.org/download
wget -q ${vbox}/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q ${vbox}/oracle_vbox.asc -O- | sudo apt-key add -
```

Modify sources list:
```
sudo sh -c 'echo "deb https://download.virtualbox.org/virtualbox/debian $(lsb_release -sc) contrib" >> /etc/apt/sources.list'
```

Update against the new list:
```
sudo apt-get update
```

Install virtual box:
```
sudo apt-get install virtualbox-5.2
```

Confirm installation
```
VBoxManage -v
vboxmanage list vms
```

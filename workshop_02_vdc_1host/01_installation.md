# Exercise: Install Wakame-vdc in one host

## Goals

* Learn how to set up and use Wakame-vdc
* Get the simplest working Wakame-vdc setup

## Assignment

Use your ssh client to log into the machine labeled `Wakame1`.

Wakame-vdc is designed to be distributed running on multiple servers and communicating through the network but it can also run with all components installed on one server. That is the setup we are going to create right now.

![Wakame in 1 host](images/02_01_01_wakame_1host.png)

### Yum repository setup

We will use the Redhat package manager `yum` to install Wakame-vdc. We will have to set up some repositories so `yum` knows where to download its packages from.

**Remark:** These yum repositories are already set up in the image and the packages are pre-cached to avoid unnecessary download times. These steps are here for completeness but you can skip to Installation.

First add Axsh's Wakame-vdc yum repository.

```
sudo curl -o /etc/yum.repos.d/wakame-vdc-stable.repo -R https://raw.githubusercontent.com/axsh/wakame-vdc/master/rpmbuild/yum_repositories/wakame-vdc-stable.repo
```

Wakame-vdc can work with OpenVz, KVM and LXC. Today we'll be working with OpenVz. Set up OpenVz's yum repository as well and tell the OS to trust it by assing its GPG key.

```
sudo curl -o /etc/yum.repos.d/openvz.repo -R https://raw.githubusercontent.com/axsh/wakame-vdc/develop/rpmbuild/yum_repositories/openvz.repo

sudo rpm --import http://download.openvz.org/RPM-GPG-Key-OpenVZ
```

Finally install the epel repository. This contains several dependencies for Wakame-vdc.

```
sudo yum install -y epel-release
```

### Installation

First let's install the `wakame-vdc-dcmgr-vmapp-config` package. This contains both **DCMGR** and **Collector**.

**RabbitMQ** and **MySQL** will be installed along with it as dependencies.

```
sudo yum install -y wakame-vdc-dcmgr-vmapp-config
```

Next install HVA. There's 3 versions of it. One for OpenVz, one for LXC and one for KVM. The contents of the HVA package itself are exactly the same. Only the dependencies are different. We are going to install the OpenVz version.

```
sudo yum install -y wakame-vdc-hva-openvz-vmapp-config
```

Finally install the `mussel` client. This is a CLI that we will use to send commands to Wakame-vdc's Web API aka DCMGR.

```
sudo yum install -y wakame-vdc-client-mussel
```

### Reboot

OpenVz uses a custom Linux kernel. We need to reboot to load it.

```
sudo reboot
```

### Set up bridged networking

Now we are going to create a bridge just like we did in the previous workshop. Unlike the previous workshop, we are going to set up a network script that makes sure the bridge is created every time on boot.

Create the file `/etc/sysconfig/network-scripts/ifcfg-br0` with the following contents

```
DEVICE=br0
TYPE=Bridge
BOOTPROTO=static
ONBOOT=yes
NM_CONTROLLED=no
IPADDR=192.168.4.10
NETMASK=255.255.255.0
GATEWAY=192.168.4.1
DNS1=8.8.8.8
DELAY=0
```

Bring the bridge up with the following command.

```
sudo ifup br0
```

This has done essentially the same as the bridge setup exercises in the last workshop. Only this time the bridge will be created automatically on every boot.

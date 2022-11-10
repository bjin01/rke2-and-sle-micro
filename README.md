# Deploy SLE-Micro and rke2

## Prep ignition, combustion and disk with label
[ignition-readme](./sle-micro/ignition/README.md)

## Create the KVM VM and start SLE-Micro
[libvirt-readme](./sle-micro/README.md)

## Install and configure DNS and external load balancer prior rke installation
[dnsmasq-and-haproxy](./haproxy/README.md)

## Install RKE2 on SLE-Micro
RKE2 installation in my case was straightforward. 
I started with first node and after the "first node" was up and running I started to install other two master nodes.

[install rke2](./rke2/README.md)


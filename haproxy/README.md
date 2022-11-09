# DNS and external load balancer for rke2
We use dnsmasq as simple dns tool for our rke2 cluster nodes.
And we use haproxy as "external loadbalancer" for kubernetes API.

The haproxy is runningo on a sle-micro v5.3 with SLES15SP4, same as rke2 nodes.

As sle-micro is an atomic OS with read-only root filesystem I need to install haproxy and dnsmasq using transactional-update command:
```
transactional-update pkg install dnsmasq
transactional-update pkg install haproxy

```
Reboot of the node after package install is needed as it will boot into the snapshot that has the new pkgs installed in it.

## dnsmasq
```
systemctl enable dnsmasq.service
systemctl start dnsmasq.service
systemctl status dnsmasq.service

vi /etc/dnsmasq.conf
```
In the dnsmasq.conf I configured below settings specifically:
```
# forward unknow targets to upstream, external dns
server=8.8.8.8

listen-address=127.0.0.1,192.168.122.189

# my dnsmasq is to be used only for dns not for dhcp because libvirt already comes with dhcp function.
no-dhcp-interface=eth0

# Provides the domain part for "expand-hosts"
domain=bocap.cloud
```
Verify dnsmasq configuration & restart dnsmasq:
```
dnsmasq --test
systemctl restart dnsmasq.service
```

## haproxy configurations:
haproxy is here used as layer 4 tcp load balancer.
All rke2 nodes are in same subnet as well as haproxy host.

The haproxy load balance configuration used here is very basic. 
The kubernetes api endpoint is in my case "mykube.bocap.cloud".

The dnsmasq is answering dns lookup for "mykube.bocap.cloud" the IP address of haproxy host. 
The haproxy host then forwards the request to the configured backend nodes which are rke2 master nodes, we have 3 master nodes.

Below is the important part of haproxy.cfg for my test rke2 cluster.

```
frontend rkeapi
  bind *:6443
  mode tcp
  default_backend rkeapibackend

frontend rkeregister
  bind *:9345
  mode tcp
  default_backend rkeregisterbackend

backend rkeapibackend
  balance roundrobin
  server mynode1 192.168.122.14:6443 check
  server mynode2 192.168.122.113:6443 check
  server mynode3 192.168.122.51:6443 check


backend rkeregisterbackend
  balance roundrobin
  server mynode1 192.168.122.14:9345 check
```

The backend rkeregisterbackend is used for registering the other nodes as mynode1 is the "first node" in rke2 cluster.

After modifying haproxy.cfg use this command to check for syntax errors:
```
haproxy -c -f /etc/haproxy/haproxy.cfg
```

And do not forget to restart haproxy service after config change:
```
systemctl restart haproxy.service
```


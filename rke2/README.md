# Install RKE2 on SLE-Micro

RKE2 installation in my case was straightforward. 
I started with first node and after the "first node" was up and running I started to install other two master nodes.
```
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server.service
systemctl start rke2-server.service
```
Copy the ```/etc/rancher/rke2/rke2.yaml``` to your local system and change the 
```
server: https://127.0.0.1:6443

to

server: https://mykube.bocap.cloud:6443
```
so that we use external load balancer.

Important details:
For registering other new master nodes to the "first node" the config must include the server parameter as below.

```
cat /etc/rancher/rke2/config.yaml 
token: blabla-token
server: https://mykube.bocap.cloud:9345
tls-san:
  - mykube.bocap.cloud
```

https://docs.ranchermanager.rancher.io/v2.5/how-to-guides/new-user-guides/kubernetes-cluster-setup/rke2-for-rancher

crictl can be used to obtain pod and containers on local host. To use crictl we can create an alias to simplify the command call.
```
alias crictl="/var/lib/rancher/rke2/data/v1.25.3-rke2r1-f85906cfac77/bin/crictl -r unix:///run/k3s/containerd/containerd.sock"
crictl pod
crictl ps 
```
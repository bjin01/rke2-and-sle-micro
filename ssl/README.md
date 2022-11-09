# Configure haproxy ssl termination
When I was learning how to configure haproxy tls termination I found a useful bash script that generates self-signed CA certificates and signs server certificate signing request.

If someone needs a haproxy load balancing with tls termination below haproxy.cfg can be used:
```
defaults
  log     global
  mode    http
  option  log-health-checks
  option  log-separate-errors
  option  dontlog-normal
  option  dontlognull
  option  httplog
  option  socket-stats
  retries 3
.
.
.

frontend myfrontend
  bind *:6443 ssl crt /etc/haproxy/certs/mykube.pem ssl-min-ver TLSv1.2
  http-request redirect scheme https unless { ssl_fc }
  default_backend myrke2

backend myrke2
  balance roundrobin
  server mynode1 192.168.122.14:6443 check
  server mynode2 192.168.122.113:6443 check
  server mynode3 192.168.122.51:6443 check
```

In order to provide a "fullchain.pem" we need to combine rootCA cert, intermediate cert and server certificate + CA cert's private key in __one file__.
```
cat rootCA.crt server.crt rootCA.key > mykube.pem
```
> Note: if the fullchain.pem is not correct the systemd service haproxy.service start will fail.

To test the tls termination we can start a pseudo http listener using python on one of the backend nodes:
```
python3 -m http.server 6443 --bind 0.0.0.0
```
And now we test the connection:
```
curl -v -k https://192.168.122.189:6443
```
The output should show you some http output.
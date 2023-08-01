# Install ingress nginx with helm and haproxy.

<br/>

## If you cannot install helm.

<br/>

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
$ helm version
```

<br/>

## Install ingress nginx with helm.

<br/>
 
```
$ helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.nodePorts.http="32012" --set controller.service.nodePorts.https="32002" --set controller.ingressClassResource.default=true --set controller.service.type="NodePort" --set controller.metrics.enabled=true --set-string controller.podAnnotations."prometheus\.io/scrape"="true" --set-string controller.podAnnotations."prometheus\.io/port"="10254"

# Check list helm in namespace ingress-nginx

$ helm ls -n ingress-nginx
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
ingress-nginx   ingress-nginx   1               2023-07-27 10:37:04.089895333 +0700 +07 deployed        ingress-nginx-4.7.1     1.8.1 

# Check service of ingress-nginx

$ kubectl get service -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.233.17.182   <none>        80:32012/TCP,443:32002/TCP   5d
ingress-nginx-controller-admission   ClusterIP   10.233.38.76    <none>        443/TCP                      5d
ingress-nginx-controller-metrics     ClusterIP   10.233.33.184   <none>        10254/TCP                    5d
```

<br/>

## Install and setup Haproxy

<br/>

### Note
 
<br />

- Haproxy is an application Loadbalancer. In this project, we will be using it connect to ingress-nginx of kubernetes, currently using NodePort.
- As you can see above, the `ingress-nginx-controller` service has the type `NodePort` and it is exposing port `32012` and `32002`.

<br/>

### Install Haproxy

<br/>

#### CentOS or Red Hat Enterprise Linux

```
$ yum install haproxy
```

#### Ubuntu

```
$ apt-get install haproxy
```

#### Debian

```
$ zypper install haproxy
```

<br/>

### Config Haproxy

<br/>

- Paste this config to the file /etc/haproxy/haproxy.cfg

```
global
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    daemon

defaults
    log                     global
    option                  dontlognull
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen stats # Define a listen section called "stats"
  bind :8080 # Listen on port 9000
  mode http
  stats enable  # Enable stats page
  stats hide-version  # Hide HAProxy version
  stats realm Haproxy\ Statistics  # Title text for popup window
  stats uri /haproxy  # Stats URI

frontend http_frontend
    bind *:80
    default_backend http_backend

backend http_backend
    server lab-masternode01 172.16.68.150:32012 check # IPv4 addresses of the master and worker nodes
    server lab-masternode02 172.16.68.151:32012 check # IPv4 addresses of the master and worker nodes
    server lab-masternode03 172.16.68.152:32012 check # IPv4 addresses of the master and worker nodes
    server lab-workernode01 172.16.68.153:32012 check # IPv4 addresses of the master and worker nodes
    server lab-workernode02 172.16.68.154:32012 check # IPv4 addresses of the master and worker nodes
    server lab-workernode03 172.16.68.155:32012 check # IPv4 addresses of the master and worker nodes

frontend https_frontend
    bind *:443
    default_backend https_backend

backend https_backend
    server lab-masternode01 172.16.68.150:32002 check # IPv4 addresses of the master and worker nodes
    server lab-masternode02 172.16.68.151:32002 check # IPv4 addresses of the master and worker nodes
    server lab-masternode03 172.16.68.152:32002 check # IPv4 addresses of the master and worker nodes
    server lab-workernode01 172.16.68.153:32002 check # IPv4 addresses of the master and worker nodes
    server lab-workernode02 172.16.68.154:32002 check # IPv4 addresses of the master and worker nodes
    server lab-workernode03 172.16.68.155:32002 check # IPv4 addresses of the master and worker nodes
```

- After setup config, reboot haproxy to apply the new config.

```
$ systemctl restart haproxy
$ systemctl status haproxy # view status of haproxy
● haproxy.service - SYSV: HAProxy is a TCP/HTTP reverse proxy which is particularly suited for high availability environments.
   Loaded: loaded (/etc/rc.d/init.d/haproxy; bad; vendor preset: disabled)
   Active: active (running) since Fri 2023-07-28 11:51:21 +07; 4 days ago
     Docs: man:systemd-sysv-generator(8)
  Process: 1169 ExecStart=/etc/rc.d/init.d/haproxy start (code=exited, status=0/SUCCESS)
 Main PID: 1293 (haproxy)
    Tasks: 2
   Memory: 4.3M
   CGroup: /system.slice/haproxy.service
           └─1293 /usr/sbin/haproxy -D -f /etc/haproxy/haproxy.cfg -p /var/run/haproxy.pid

Jul 28 11:51:20 vdi-cicd systemd[1]: Starting SYSV: HAProxy is a TCP/HTTP reverse proxy which is particularly suited for high availability environments....
Jul 28 11:51:20 vdi-cicd haproxy[1169]: /etc/rc.d/init.d/haproxy: line 26: [: =: unary operator expected
Jul 28 11:51:21 vdi-cicd haproxy[1169]: Starting haproxy: [NOTICE]   (1285) : haproxy version is 2.6.0-a1efc04
Jul 28 11:51:21 vdi-cicd haproxy[1169]: [NOTICE]   (1285) : path to executable is /usr/sbin/haproxy
Jul 28 11:51:21 vdi-cicd haproxy[1169]: [ALERT]    (1285) : config : parsing [/etc/haproxy/haproxy.cfg:3] : 'pidfile' already specified. Continuing.
Jul 28 11:51:21 vdi-cicd haproxy[1169]: [  OK  ]
Jul 28 11:51:21 vdi-cicd systemd[1]: Started SYSV: HAProxy is a TCP/HTTP reverse proxy which is particularly suited for high availability environments..
``` 
<br/>

### In this docs, I only setup environment. With write file ingress i will talk later.

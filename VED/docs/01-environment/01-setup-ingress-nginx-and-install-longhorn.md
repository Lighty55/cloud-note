# Install ingress nginx with helm, haproxy and longhorn.

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

### In this document, I only setup the environment. I will talk about writing the 'ingress' file.

<br/>

## Install Longhorn.

<br/>

### Setup environment in all worker node

<br/>

```zsh
$ mkdir /data/longhorn-storage/
$ sudo yum -y install iscsi-initiator-utils
```

<br/>

### Create folder and download helm chart

<br/>

```zsh
$ cd
$ mkdir longhorn-storage
$ cd longhorn-storage
$ helm repo add longhorn https://charts.longhorn.io
$ helm repo update
$ helm search repo longhorn
$ helm pull longhorn/longhorn
```

<br/>

### Change file values-longhorn.yaml

<br/>

```zsh
$ cp longhorn/values.yaml values-longhorn.yaml
$ vim values-longhorn.yaml
```

```yaml
service:
  ui:
    type: ClusterIP
  manager:
    type: ClusterIP
    
defaultDataPath: /data/longhorn-storage/
replicaSoftAntiAffinity: true
storageMinimalAvailablePercentage: 15
upgradeChecker: false
defaultReplicaCount: 2
backupstorePollInterval: 500
nodeDownPodDeletionPolicy: do-nothing
guaranteedEngineManagerCPU: 15
guaranteedReplicaManagerCPU: 15

ingress:  
  enabled: true
  ingressClassName: nginx 
  host: longhorn-vedlab.com

namespaceOverride: "longhorn-lab"
```

<br/>

### Deploy longhorn

<br/>

```zsh
$ helm install longhorn-storage -f values-longhorn.yaml longhorn --namespace longhorn-lab --set defaultSettings.defaultDataPath="/data/longhorn-storage"
```

<br/>

- Affter deploy check process pod of longhorn

```zsh
$ kubectl get pod -n longhorn-lab
NAME                                                READY   STATUS    RESTARTS       AGE
csi-attacher-759f487c5-dftxc                        1/1     Running   4 (10d ago)    13d
csi-attacher-759f487c5-mnjgq                        1/1     Running   4 (10d ago)    13d
csi-attacher-759f487c5-zvzzw                        1/1     Running   3 (10d ago)    13d
csi-provisioner-6df8547696-cqflr                    1/1     Running   6 (6d5h ago)   13d
csi-provisioner-6df8547696-k7j6d                    1/1     Running   4 (7d1h ago)   13d
csi-provisioner-6df8547696-rwbsn                    1/1     Running   4 (10d ago)    13d
csi-resizer-6bf6dbcb4-2v4n5                         1/1     Running   4 (10d ago)    13d
csi-resizer-6bf6dbcb4-4lkvc                         1/1     Running   3 (10d ago)    13d
csi-resizer-6bf6dbcb4-kmfhl                         1/1     Running   5 (6d5h ago)   13d
csi-snapshotter-69d7b7b84-8n952                     1/1     Running   6 (6d5h ago)   13d
csi-snapshotter-69d7b7b84-frjnn                     1/1     Running   3 (10d ago)    13d
csi-snapshotter-69d7b7b84-hlghl                     1/1     Running   4 (10d ago)    13d
engine-image-ei-74783864-kl8jb                      1/1     Running   5 (10d ago)    13d
engine-image-ei-74783864-slghw                      1/1     Running   4 (10d ago)    13d
engine-image-ei-74783864-wpqkw                      1/1     Running   5 (10d ago)    13d
instance-manager-89c5daca95bc54c99eee61ce692cc789   1/1     Running   0              10d
instance-manager-8d812e130c9e58c479f8d318e6527ac0   1/1     Running   0              10d
instance-manager-8e9bed22362a3ac2f3487f49414915bd   1/1     Running   0              10d
longhorn-csi-plugin-cvx6s                           3/3     Running   11 (10d ago)   13d
longhorn-csi-plugin-nnkrc                           3/3     Running   15 (10d ago)   13d
longhorn-csi-plugin-xlw5s                           3/3     Running   13 (10d ago)   13d
longhorn-driver-deployer-666b786578-99xd4           1/1     Running   5 (10d ago)    13d
longhorn-manager-h2lmf                              1/1     Running   5 (10d ago)    13d
longhorn-manager-hk746                              1/1     Running   4 (10d ago)    13d
longhorn-manager-t6ct5                              1/1     Running   6 (10d ago)    13d
longhorn-ui-76d8759584-79xzx                        1/1     Running   6 (10d ago)    13d
longhorn-ui-76d8759584-lnjm4                        1/1     Running   4 (10d ago)    13d
```

<br/>

### Deploy storage class

<br/>

- The storage class resource has provisioner, parameter and reclaimPolicy. However, in this project, we will only using reclaimPolicy. If you need more information about storages class resource, please [click me](https://kubernetes.io/docs/concepts/storage/storage-classes/)

<br/>

- Deploy storage class with reclaimPolicy = delete
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: longhorn-storage-delete
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: driver.longhorn.io
allowVolumeExpansion: true
reclaimPolicy: Delete
volumeBindingMode: Immediate
parameters:
  numberOfReplicas: "2"                
  staleReplicaTimeout: "2880"
  fromBackup: ""
  fsType: "ext4"
```

<br/>

- Deploy storage class with reclaimPolicy = retain
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: longhorn-storage-retain
provisioner: driver.longhorn.io
allowVolumeExpansion: true
reclaimPolicy: Retain
volumeBindingMode: Immediate
parameters:
  numberOfReplicas: "2"
  staleReplicaTimeout: "2880"
  fromBackup: ""
  fsType: "ext4"
```

<br/>

- If you want to set defaults for defferent storage class name
```zsh
$ kubectl patch storageclass longhorn-storage-delete -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
$ kubectl patch storageclass longhorn-storage-retain -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

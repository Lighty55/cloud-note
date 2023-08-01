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
<br/ >

## Install ingress nginx with helm.

<br/ >
```
$ helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.nodePorts.http="32012" --set controller.service.nodePorts.https="32002" --set controller.ingressClassResource.default=true --set controller.service.type="NodePort" --set controller.metrics.enabled=true --set-string controller.podAnnotations."prometheus\.io/scrape"="true" --set-string controller.podAnnotations."prometheus\.io/port"="10254"
// $ helm ls -n ingress-nginx
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
ingress-nginx   ingress-nginx   1               2023-07-27 10:37:04.089895333 +0700 +07 deployed        ingress-nginx-4.7.1     1.8.1 
// $ kubectl get service -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.233.17.182   <none>        80:32012/TCP,443:32002/TCP   5d
ingress-nginx-controller-admission   ClusterIP   10.233.38.76    <none>        443/TCP                      5d
ingress-nginx-controller-metrics     ClusterIP   10.233.33.184   <none>        10254/TCP                    5d
```
<br/ >
## Install and setup Haproxy

<br/ >



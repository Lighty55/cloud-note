# Keycloak installation

<br/>

```
$ mkdir -p  ~/projects/dev/ml/
$ cd ~/projects/dev/ml/
$ git clone https://github.com/Lighty55/cloud-note.git --branch VED
$ cd ~/projects/dev/ml/VED
```

<br/>
### If you using longhorn.
- Add `spec.temlace.spec.securityContext.fsGroup=1001` to file keycloak.yaml (notice: add this text to the deployment not add pvc or service).
```
...
spec:
  templace:
    spec:
      securityContext:
        fsGroup: 1001
...
```

<br/>
```
$ kubectl create ns keycloak
$ kubectl create -f Chapter04/keycloak.yaml
$ kubectl get pod -n keycloak
```

<br/>

```
// $ watch kubectl get pods -n keycloak
NAME                        READY   STATUS    RESTARTS   AGE
keycloak-75799d947b-l9mkw   1/1     Running   0          56s

```

<br/>

```
$ export MINIKUBE_IP_ADDR=$(minikube ip --profile ${PROFILE})
$ echo ${MINIKUBE_IP_ADDR}
```

<br/>

```
// Check
// $ envsubst < Chapter04/keycloak/keycloak-ingress.yaml
```

<br/>

```
$ envsubst < Chapter04/keycloak/keycloak-ingress.yaml | kubectl create -f - -n keycloak
```

<br/>

```
// $ kubectl get ingress -n keycloak
NAME       CLASS   HOSTS                          ADDRESS        PORTS     AGE
keycloak   nginx   keycloak.192.168.49.2.nip.io   192.168.49.2   80, 443   4m8s
```

<br/>

```
// Administration Console
// admin / admin
$ echo https://keycloak.${MINIKUBE_IP_ADDR}.nip.io/auth/
```

<br/>

### Importing the Keycloak configuration for the ODH components

```
Keycloak WEB UI

import -> Chapter05/realm-export.json

- If a resource exists - Skip

Import
```

<br/>

### Creating a Keycloak user

<br/>

```
Users -> Add user

Username: mluser
Email: mluser@example.com
First Name: mluser
Last Name: mluser

User Enabled: ON
Email Verified: ON

Groups: ml-group

SAVE
```

<br/>

```
Credentials:

Password: mluser
Password Confirmation: mluser

Temporary: OFF

Set Password
```

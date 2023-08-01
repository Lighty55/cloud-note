# Keycloak installation

<br/>

```
$ mkdir -p  ~/projects/dev/ml/
$ cd ~/projects/dev/ml/
$ git clone https://github.com/Lighty55/cloud-note.git --branch VED
$ cd ~/projects/dev/ml/VED
```

<br/>

### If you using longhorn with Keycloak.

- Add `spec.temlace.spec.securityContext.fsGroup=1001` to file keycloak.yaml (notice: add this text to the deployment not add pvc or service).

```
...
spec:
  templace:
    spec:
      securityContext: <- This is you need to add in.
        fsGroup: 1001
...
```

<br/>

### Deploy keycloak (Including Deployment, service, pvc)

<br/>

```
$ kubectl create ns keycloak
$ kubectl create -f Chapter04/keycloak.yaml
$ kubectl get pod,serivce,pvc -n keycloak
```

<br/>

### Deploy ingress Keycloak

<br/>

```
$ kubectl create ingress keycloak-ingress.yaml -n keycloak

# After deploy, we will check state of keycloak-ingress

$ kubectl get ingress -n keycloak
NAME       CLASS   HOSTS                 ADDRESS         PORTS     AGE
keycloak   nginx   keycloak-vedlab.com   10.233.17.182   80, 443   10d
```

<br/>

```
// Administration Console
// admin / admin
$ echo https://keycloak-vedlab.com/auth/
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

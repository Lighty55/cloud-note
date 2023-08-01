# Open Data Hub (ODH) operator installation

https://github.com/opendatahub-io/opendatahub-operator

## Install community-operators-redhat

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: community-operators-redhat
  namespace: olm
spec:
  displayName: Community Operators Red Hat
  #  image: registry.access.redhat.com/redhat/community-operator-index:v4.13
  image: registry.access.redhat.com/redhat/community-operator-index:v4.9
  publisher: RedHat
  sourceType: grpc
  updateStrategy:
    registryPoll:
      interval: 60m
EOF
```

<br/>

```
// wait 2-4 minutes
$ kubectl get packagemanifests -o wide -n olm | grep -I opendatahub
opendatahub-operator                        Community Operators Red Hat   52s
```

<br/>

## Install OpendataHub-operator

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: opendatahub-operator
  namespace: operators
spec:
  channel: stable
  installPlanApproval: Automatic
  # installPlanApproval: Manual
  name: opendatahub-operator
  source: community-operators-redhat
  sourceNamespace: olm
  # startingCSV: opendatahub-operator.v1.6.0
  # startingCSV: opendatahub-operator.v1.4.2
  startingCSV: opendatahub-operator.v1.1.1
EOF
```

<br/>

- After finishing pasting the above section, please continue and paste the following section below this.

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: opendatahub-operator
  namespace: operators
spec:
  channel: stable
  installPlanApproval: Manual
  name: opendatahub-operator
  source: community-operators-redhat
  sourceNamespace: olm
  startingCSV: opendatahub-operator.v1.1.1
EOF
```
<br/>

### This is the end result.

<br/>

```
$ kubectl get pod -n operators
NAME                                   READY   STATUS    RESTARTS   AGE
opendatahub-operator-b5f4c5757-d9td2   1/1     Running   0          15s
```
<br/>

```
$ kubectl get installplan -A
$ kubectl get subscription -n operators
NAME                   PACKAGE                SOURCE                       CHANNEL
opendatahub-operator   opendatahub-operator   community-operators-redhat   stable
```

<br/>

```
// $ kubectl describe catalogsource community-operators-redhat -n olm
// $ kubectl describe catalogsource operatorhubio-catalog -n olm
```

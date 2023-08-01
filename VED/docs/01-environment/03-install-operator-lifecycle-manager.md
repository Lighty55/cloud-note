# Operator Lifecycle Manager (OLM) installation

https://github.com/operator-framework/operator-lifecycle-manager/releases

<!-- <br/>

```
$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.24.0/install.sh | bash -s v0.24.0
``` -->

<br/>

```
$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.21.1/install.sh | bash -s v0.21.1
```

<br/>

```
// 30 sec
$ kubectl get pods -n olm
NAME                               READY   STATUS    RESTARTS   AGE
catalog-operator-7994ff899-tw72s   1/1     Running   0          3m42s
olm-operator-59555bc789-qrr7z      1/1     Running   0          3m42s
operatorhubio-catalog-6qkc4        1/1     Running   0          3m32s
packageserver-c45f4979c-q5ldc      1/1     Running   0          3m32s
packageserver-c45f4979c-xsdmr      1/1     Running   0          3m32s
```

<br/>

```
// 45 sec
$ kubectl get catalogsource -n olm
NAME                    DISPLAY               TYPE   PUBLISHER        AGE
operatorhubio-catalog   Community Operators   grpc   OperatorHub.io   4m8s
```

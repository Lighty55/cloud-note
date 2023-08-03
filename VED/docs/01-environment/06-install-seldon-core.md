# Install seldon core latest

<br/>

## Notice: if you have already executed the kfdef manifest, please follow these step ([Or else, move on to the next step]()).

<br/>

- step 1: Find seldon old version

```
$ kubectl get deployment -n ml-workshop
NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
app-aflow-airflow-scheduler                   1/1     1            1           10d
app-aflow-airflow-web                         1/1     1            1           10d
jupyterhub                                    1/1     1            1           11d
logger                                        1/1     1            1           10d
minio-ml-workshop                             1/1     1            1           11d
mlflow                                        1/1     1            1           11d
model-1-mlflowdemo-predictor-0-predictor      1/1     1            1           10d
model-test-predictor-0-model-test-predictor   1/1     1            1           10d
pg-flights-data                               1/1     1            1           10d
prometheus-operator                           1/1     1            1           8d
seldon-controller-manager                     1/1     1            1           11d      # <- This is removed.
seldon-f0328da1dc42cee051dba0ffcf56e06f       1/1     1            1           10d
spark-operator                                1/1     1            1           11d
```

<br/>

- step 2: Delete deployment seldon old version

```
$ kubectl delete deployment -n ml-workshop seldon-controller-manager
```

<br/>

- step 3: Recheck deployment, if it has been removed, proceed to the next step. If it not has been removed, check step 2 again.

<br/>

## Install Ambassador with helm

<br/>

```shell
$ helm repo add datawire https://app.getambassador.io
$ helm repo update
$ kubectl apply -f https://app.getambassador.io/yaml/edge-stack/3.7.0/aes-crds.yaml
$ kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
$ helm install -n ambassador --create-namespace edge-stack datawire/edge-stack && kubectl rollout status  -n ambassador deployment/edge-stack -w
```

<br/>

## Install Seldon core

<br/>

```zsh
$ helm install seldon-core seldon-core-operator --repo https://storage.googleapis.com/seldon-charts --set usageMetrics.enabled=true --set ambassador.enabled=true --namespace ml-workshop
```

<br/>

- After running, check seldon-core new versions

```sh
$ kubectl get pod -n ml-workshop | grep seldon-controller-manager
seldon-controller-manager-67c564546c-7tk9n                     1/1     Running     0               10m

```

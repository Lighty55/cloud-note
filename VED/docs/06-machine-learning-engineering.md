# [OK!] Chapter 6. Machine Learning Engineering

<br/>

### Using a custom notebook image

```
$ cd Chapter06/
$ docker build -t scikit-notebook:v1.1.0 -f CustomNotebookDockerfile .
$ docker tag scikit-notebook:v1.1.0 webmakaka/scikit-notebook:v1.1.0
$ docker login
$ docker push webmakaka/scikit-notebook:v1.1.0
```

```
$ cd ../
$ vi manifests/jupyterhub-images/base/customnotebook-imagestream.yaml
$ vi manifests/kfdef/ml-platform.yaml
```

<!-- <br/>

```
$ kubectl apply -f ./customnotebook-imagestream.yaml
``` -->

<br/>

```
$ kubectl get pods -n ml-workshop | grep jupyterhub
jupyterhub-7848ccd4b7-5wn6q                    1/1     Running     0              153m
jupyterhub-db-0                                1/1     Running     0              153m
```

<br/>

```
$ kubectl delete pod jupyterhub-7848ccd4b7-5wn6q -n ml-workshop
```

<br/>

Nothing happened.

<br/>

### Introducing MLflow

<br/>

```
// minio / minio123
https://minio.192.168.49.2.nip.io
```

<br/>

```
// mluser / mluser
https://mlflow.192.168.49.2.nip.io
```

<br/>

```
// mluser / mluser
https://jupyterhub.192.168.49.2.nip.io/hub/spawn
```

<br/>

```
Image: SciKit v.1.10 - Elyra Notebook Image
Container size: Small

Variable name: AWS_SECRET_ACCESS_KEY
Variable value: minio123

Secret: no
```

<br/>

```
Jupyter >

Run -> Chapter06/hellomlflow.ipynb

RUN -> Chapter06/hellomlflow-custom.ipynb
```

<br/>

```
MiniIO > Buckets -> MlFlow
```

<br/>

```
MLFlow > Experiment > Artifacts > Model > Register Model > mlflowdemo

MLFlow > Models > mlflowdemo > Stage > Production
```

<br/>

```
// To Debug
print(os.environ)
```

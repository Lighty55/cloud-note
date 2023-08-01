# [OK!] Chapter 5. Data Engineering

<br/>

### Validating the JupyterHub installation

<br/>

```
$ kubectl get ingress -n ml-workshop
NAME                   CLASS   HOSTS                            ADDRESS        PORTS     AGE
ap-airflow2            nginx   airflow.192.168.49.2.nip.io      192.168.49.2   80, 443   3m36s
grafana                nginx   grafana.192.168.49.2.nip.io      192.168.49.2   80, 443   3m36s
jupyterhub             nginx   jupyterhub.192.168.49.2.nip.io   192.168.49.2   80, 443   3m39s
minio-ml-workshop-ui   nginx   minio.192.168.49.2.nip.io        192.168.49.2   80, 443   3m36s
mlflow                 nginx   mlflow.192.168.49.2.nip.io       192.168.49.2   80, 443   3m36s
```

<br/>

```
// mluser / mluser
https://jupyterhub.192.168.49.2.nip.io

Image: Base Elyra Notebook
Container size: Default
```

<br/>

```
// Check new container
$ kubectl get pods -n ml-workshop | grep mluser
jupyterhub-nb-mluser                           1/1     Running     0             111s
```

<br/>

Jupyter > git (left panel) > clone a repo > https://github.com/webmakaka/Machine-Learning-on-Kubernetes.git

<br/>

File > Hub Control Panel > Stop My Server

<br/>

### Creating a Spark cluster

<br/>

```
$ kubectl get pods -n ml-workshop | grep spark-operator
spark-operator-545676669f-nnz84                1/1     Running     0             80m
```

<br/>

```
$ kubectl create -f Chapter05/simple-spark-cluster.yaml -n ml-workshop
```

<br/>

```
$ kubectl get pods -n ml-workshop | grep simple-spark
simple-spark-cluster-m-vpvk2                   1/1     Running     0             85s
simple-spark-cluster-w-b7qgp                   1/1     Running     0             84s
```

<br/>

```
$ kubectl delete sparkcluster simple-spark-cluster -n ml-workshop
```

<br/>

```
// mluser / mluser
https://jupyterhub.192.168.49.2.nip.io

Image: Elyra Notebook Image with Spark
Container size: Small
```

<br/>

```
$ kubectl get pods -n ml-workshop | grep mluser
jupyterhub-nb-mluser                           1/1     Running     0             6m55s
spark-cluster-mluser-m-bckpw                   1/1     Running     0             6m55s
spark-cluster-mluser-w-2sc6h                   1/1     Running     0             6m55s
spark-cluster-mluser-w-jth79                   1/1     Running     0             6m55s
```

<br/>

```
// SPARK UI
https://spark-cluster-mluser.192.168.49.2.nip.io
```

Jupyter > Chapter05/hellospark.ipynb

Run > Run All cells

<br/>

File > Hub Control Panel > Stop My Server

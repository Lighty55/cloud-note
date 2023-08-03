# Open Data Hub (ODH) installation

```
$ mkdir -p  ~/projects/dev/ml/
$ cd ~/projects/dev/ml/
$ git clone https://github.com/Lighty55/cloud-note.git --branch VED
$ cd ~/projects/dev/ml/VED
```

<br/>

## Create empty github repo with branch name main

<br/>

```
https://github.com/vdilab/airflow-dags
```

<br/>

## Add new repo

<br/>

- vim manifests/kfdef/ml-platform.yaml
```yaml
- name: DAG_REPO
  value: 'https://github.com/vdilab/airflow-dags.git'
```

<br/>

## Change domain for all service

<br/>

<br/>

- vim manifests/kfdef/ml-platform.yaml
```
# This is Normal Mode in VIM
# eg: app keycloak:   keycloak-vedlab.com
#     app jupyterhub: jupyterhub-vedlab.com
Esc
:%s/.${MINIKUBE_IP_ADDR}.nip.io/-<your subdomain>
```

<br/>

## Notice: You should follow these steps to avoid errors during deployment

<br/>

**If your domain is only declared to run locally** 
[Add local domain to app](../02-Note-Issue/01-Add-Host-K8s.md)

**If you using longhorn. You should add securityContext to deployment or stateFulSet of apps**
> keycloak <br/>
> airflow-worker
- Or if there is any issue with the log message "permission deny"

**If you create a user with the same documentation as keycloak, you should add unverify to the mlflow configuration**
```yaml
spec:
  template:
    spec:
      containers:
        - name: oauth-proxy
          image: 'quay.io/oauth2-proxy/oauth2-proxy:v7.2.0'
          args:
            - '--provider=keycloak-oidc'
            - '--https-address='
            - '--http-address=:5000'
            - '--client-id=mflow'
            - '--client-secret=ad216993-cf3a-4742-ba17-b531d5c22046'
            - '--upstream=http://localhost:5500'
            - '--email-domain=*'
            - '--insecure-oidc-allow-unverified-email=true' <- add this line
```

**If you are using a Kubernetes cluster version v1.25.x or later**
[Install new version seldon core](./01-environment/06-install-seldon-core.md)

<br/>

## Deploy all application in namespace ml-workshop

<br/>

```
$ kubectl create ns ml-workshop
```

<br/>

```
kubectl create -f manifests/kfdef/ml-platform.yaml -n ml-workshop
```

<br/>

```
$ kubectl get pod -n ml-workshop (please wait 15 minutes)
NAME                                           READY   STATUS      RESTARTS        AGE
app-aflow-airflow-scheduler-6ccc679fc6-6k9kv   2/2     Running     0               17m
app-aflow-airflow-web-55767cd99d-x9kh7         2/2     Running     0               9m41s
app-aflow-airflow-worker-0                     2/2     Running     1 (8m38s ago)   17m
app-aflow-postgresql-0                         1/1     Running     0               17m
app-aflow-redis-master-0                       1/1     Running     0               17m
flightsdatadb-0                                1/1     Running     0               15m
grafana-56594fd448-c76qd                       1/1     Running     0               17m
jupyterhub-6cff7d6cc5-62snc                    1/1     Running     0               17m
jupyterhub-db-0                                1/1     Running     0               17m
minio-ml-workshop-7fcc5dfd8-m57lv              1/1     Running     0               17m
minio-ml-workshop-lbfh5                        0/1     Completed   2               17m
mlflow-5fb4cb5d5d-89v74                        2/2     Running     0               17m
mlflow-db-0                                    1/1     Running     1 (16m ago)     17m
prometheus-odh-monitoring-0                    2/2     Running     0               13m
prometheus-operator-7869664bcc-f42d2           1/1     Running     0               15m
seldon-controller-manager-c5f59c56d-2rcc6      1/1     Running     0               17m
spark-operator-6bc4f8f5f8-rs2mk                1/1     Running     0               17m
```

<br/>

```
$ kubectl get all -n ml-workshop
```

<br/>

```
***
NAME                                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
service/app-aflow-airflow                   ClusterIP   10.106.233.18    <none>        8080/TCP             11m
service/app-aflow-airflow-worker-headless   ClusterIP   None             <none>        8793/TCP             11m
service/app-aflow-postgresql                ClusterIP   10.104.154.83    <none>        5432/TCP             11m
service/app-aflow-postgresql-headless       ClusterIP   None             <none>        5432/TCP             11m
service/app-aflow-redis-headless            ClusterIP   None             <none>        6379/TCP             11m
service/app-aflow-redis-master              ClusterIP   10.99.0.186      <none>        6379/TCP             11m
service/flightsdatadb                       ClusterIP   10.103.152.170   <none>        5432/TCP             9m17s
service/grafana                             ClusterIP   10.105.49.130    <none>        3000/TCP             11m
service/jupyterhub                          ClusterIP   10.110.27.45     <none>        8080/TCP,8081/TCP    12m
service/jupyterhub-db                       ClusterIP   10.105.3.90      <none>        5432/TCP             12m
service/minio-ml-workshop                   ClusterIP   10.101.161.251   <none>        9000/TCP,33933/TCP   11m
service/mlflow                              ClusterIP   10.104.199.112   <none>        5000/TCP,5500/TCP    11m
service/mlflow-db                           ClusterIP   10.97.125.116    <none>        5432/TCP             11m
service/prometheus-operated                 ClusterIP   None             <none>        9090/TCP             7m23s
service/seldon-webhook-service              ClusterIP   10.99.208.129    <none>        443/TCP              10m

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-aflow-airflow-scheduler   1/1     1            1           11m
deployment.apps/app-aflow-airflow-web         1/1     1            1           11m
deployment.apps/grafana                       1/1     1            1           11m
deployment.apps/jupyterhub                    1/1     1            1           12m
deployment.apps/minio-ml-workshop             1/1     1            1           11m
deployment.apps/mlflow                        1/1     1            1           11m
deployment.apps/prometheus-operator           1/1     1            1           9m23s
deployment.apps/seldon-controller-manager     1/1     1            1           11m
deployment.apps/spark-operator                1/1     1            1           11m

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/app-aflow-airflow-scheduler-6f9c44866d   1         1         1       11m
replicaset.apps/app-aflow-airflow-web-686b66f886         1         1         1       11m
replicaset.apps/app-aflow-airflow-web-6d9587698          1         1         0       3m54s
replicaset.apps/grafana-56594fd448                       1         1         1       11m
replicaset.apps/jupyterhub-6cff7d6cc5                    1         1         1       12m
replicaset.apps/minio-ml-workshop-7fcc5dfd8              1         1         1       11m
replicaset.apps/mlflow-5fb4cb5d5d                        1         1         1       11m
replicaset.apps/prometheus-operator-7869664bcc           1         1         1       9m23s
replicaset.apps/seldon-controller-manager-c5f59c56d      1         1         1       11m
replicaset.apps/spark-operator-6bc4f8f5f8                1         1         1       11m

NAME                                         READY   AGE
statefulset.apps/app-aflow-airflow-worker    1/1     11m
statefulset.apps/app-aflow-postgresql        1/1     11m
statefulset.apps/app-aflow-redis-master      1/1     11m
statefulset.apps/flightsdatadb               1/1     9m17s
statefulset.apps/jupyterhub-db               1/1     12m
statefulset.apps/mlflow-db                   1/1     11m
statefulset.apps/prometheus-odh-monitoring   1/1     7m23s

NAME                          COMPLETIONS   DURATION   AGE
job.batch/minio-ml-workshop   1/1           94s        11m
```

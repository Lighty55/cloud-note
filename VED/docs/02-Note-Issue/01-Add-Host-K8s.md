#Add host for /etc/host in apps k8s

<br/>

## Adding entries to Pod /etc/hosts with HostAliases

<br/>

- Adding entries to a Pod's /etc/hosts file provides Pod-level override of hostname resolution when DNS and other options are not applicable. You can add these custom entries with the HostAliases field in PodSpec.

- Modification not using HostAliases is not suggested because the file is managed by the kubelet and can be overwritten on during Pod creation/restart.

<br/>

### eg: Add entries to pod mlflow with HostAliases

<br/>

```
$ kubectl get pod -n ml-workshop
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

- Execution pod mlflow
`$ kubectl edit pod -n ml-workshop mlflow-5fb4cb5d5d-89v74`
```yaml
spec:
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  hostAliases:                        # <- Add words from this line downwards. 
  - hostnames:
    - keycloak-vedlab.com
    ip: 172.16.68.180
```

<br/>

## Adding entries to Deployment /etc/hosts with postStart

### COMMING SOON

<br/>

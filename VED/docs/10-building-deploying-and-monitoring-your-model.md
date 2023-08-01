# [FAILED!] Chapter 10. Building, Deploying, and Monitoring Your Model

<br/>

На этом шаге строим модель ML.

<br/>

```
// mluser / mluser
https://jupyterhub.192.168.49.2.nip.io

Image: SciKit v.1.10 - Elyra Notebook Image
Container size: Medium

Variable name: AWS_SECRET_ACCESS_KEY
Variable value: minio123

Secret: no
```

<br/>

```
RUN -> Chapter10/visualize.ipynb
```

<br/>

### Building and tuning your model using JupyterHub

<br/>

```
RUN -> Chapter10/experiments.ipynb
```

<br/>

### Tracking model experiments

<br/>

```
// mluser / mluser
https://mlflow.192.168.49.2.nip.io
```

<br/>

FlightsDelay-mluser

<br/>

<br/>

```
MLFlow > Experiment > Artifacts > Model > Register Model > flights-ontime

MLFlow > Models > flights-ontime > Stage > Production
```

<br/>

### Deploying the model as a service

<br/>

Maebe need to copy FlgithsDelayOrdinalEncoder.pkl to model_build_push folder and add it in properties for build_push_image.py

<br/>

Chapter10/model_deploy_pipeline/flights_model.pipeline

<br/>

**build_push_image.py**

Runtime Image: Kaniko Container Builder

<br/>

```
Environment Variables

MODEL_NAME=flights-ontime
MODEL_VERSION=1
CONTAINER_REGISTRY=https://index.docker.io/v1/
CONTAINER_REGISTRY_USER=webmakaka
CONTAINER_REGISTRY_PASSWORD=mypassword
CONTAINER_DETAILS=webmakaka/flights-ontime:latest
```

<br/>

**deploy_model.py**

Runtime Image: Airflow Python Runner

<br/>

```
File Dependencies:

Ingress.yaml
SeldonDeploy.yaml
```

<br/>

```
Environment Variables

MODEL_NAME=flights-ontime
MODEL_VERSION=1
CONTAINER_DETAILS=webmakaka/flights-ontime:latest
CLUSTER_DOMAIN_NAME=192.168.49.2.nip.io
```

<br/>

RUN

<br/>

https://hub.docker.com/u/webmakaka/

<br/>

logs are in minio

<br/>

### Calling your model

```
$ kubectl get ingress -n ml-workshop
```

<br/>

```
$ cd Chapter10/inference
```

<br/>

```
$ curl -vvvk --header "content-type: application/json" -X POST -d @data.json https://flights-ontime.192.168.49.2.nip.io/api/v1.0/predictions | jq
```

<br/>

**response**

```
{
  "data": {
    "names": [
      "t:0",
      "t:1"
    ],
    "ndarray": [
      [
        0.7724004126048876,
        0.22759958739511232
      ]
    ]
  },
  "meta": {
    "requestPath": {
      "predictor": "webmakaka/flights-ontime:latest",
      "transformer": "webmakaka/flights-ontime:latest"
    }
  }
}

```

<br/>

```
$ exec grafana
// $ curl -X GET seldon-d4908acab506130ac85b756a333763aa-56dcf794b8-ggrkz:8000/prometheus
$ curl -X GET model-26fff37f17864ac489bd8f42b6095959-flights-ontime:8000/prometheus

***
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="model-26fff37f17864ac489bd8f42b6095959",method="post",model_image="webmakaka/flights-ontime",model_name="transformer",model_version="latest",predictor_name="flights-ontime",predictor_version="",service="/transform-input",le="1"} 1
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="model-26fff37f17864ac489bd8f42b6095959",method="post",model_image="webmakaka/flights-ontime",model_name="transformer",model_version="latest",predictor_name="flights-ontime",predictor_version="",service="/transform-input",le="2.5"} 1
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="model-26fff37f17864ac489bd8f42b6095959",method="post",model_image="webmakaka/flights-ontime",model_name="transformer",model_version="latest",predictor_name="flights-ontime",predictor_version="",service="/transform-input",le="5"} 1
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="model-26fff37f17864ac489bd8f42b6095959",method="post",model_image="webmakaka/flights-ontime",model_name="transformer",model_version="latest",predictor_name="flights-ontime",predictor_version="",service="/transform-input",le="7.5"} 1
***
```

<br/>

```
servicemonitor -> seldon-services
seldondeployments
```

<br/>

### Monitoring your model

[How to Fix Issue](https://github.com/PacktPublishing/Machine-Learning-on-Kubernetes/issues/15#issuecomment-1649108272)

```
// admin / admin
https://grafna.192.168.49.2.nip.io
```

<br/>

Configuration -> Data sources -> Prometheus

<br/>

```
Name: HPonbHsnk
URL: http://prometheus-operated:9090
```

<br/>

```
$ kubectl get service -n ml-workshop | grep prometheus-operated
```

<br/>

```
+ -> Import -> Chapter10/grafana-dashboard/sample-seldon-dashboard.json
```

<br/>

Grafana UI do not show anything.

Prometheus -> No metrics found

```
$ curl prometheus-operated:9090/metrics
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 5.8443e-05
go_gc_duration_seconds{quantile="0.25"} 0.000127308
go_gc_duration_seconds{quantile="0.5"} 0.000184111
go_gc_duration_seconds{quantile="0.75"} 0.000409082
go_gc_duration_seconds{quantile="1"} 0.006598965
go_gc_duration_seconds_sum 0.196233683
go_gc_duration_seconds_count 502
***
```

<br/>

```
$ kubectl port-forward svc/prometheus-operated 9090:9090 -n ml-workshop
```

<br/>

```
http://localhost:9090/metrics
http://localhost:9090/graph
```

<br/>

```
global:
  scrape_interval: 30s
  scrape_timeout: 10s
  evaluation_interval: 30s
  external_labels:
    prometheus: ml-workshop/odh-monitoring
    prometheus_replica: prometheus-odh-monitoring-0
rule_files:
- /etc/prometheus/rules/prometheus-odh-monitoring-rulefiles-0/*.yaml
scrape_configs:
- job_name: serviceMonitor/ml-workshop/seldon-services/0
  honor_timestamps: true
  scrape_interval: 10s
  scrape_timeout: 10s
  metrics_path: /prometheus
  scheme: http
  follow_redirects: true
  enable_http2: true
  relabel_configs:
  - source_labels: [job]
    separator: ;
    regex: (.*)
    target_label: __tmp_prometheus_job_name
    replacement: $1
    action: replace
  - source_labels: [__meta_kubernetes_service_label_app_kubernetes_io_managed_by,
      __meta_kubernetes_service_labelpresent_app_kubernetes_io_managed_by]
    separator: ;
    regex: (seldon-core);true
    replacement: $1
    action: keep
  - source_labels: [__meta_kubernetes_endpoint_port_name]
    separator: ;
    regex: "8000"
    replacement: $1
    action: keep
  - source_labels: [__meta_kubernetes_endpoint_address_target_kind, __meta_kubernetes_endpoint_address_target_name]
    separator: ;
    regex: Node;(.*)
    target_label: node
    replacement: ${1}
    action: replace
  - source_labels: [__meta_kubernetes_endpoint_address_target_kind, __meta_kubernetes_endpoint_address_target_name]
    separator: ;
    regex: Pod;(.*)
    target_label: pod
    replacement: ${1}
    action: replace
  - source_labels: [__meta_kubernetes_namespace]
    separator: ;
    regex: (.*)
    target_label: namespace
    replacement: $1
    action: replace
  - source_labels: [__meta_kubernetes_service_name]
    separator: ;
    regex: (.*)
    target_label: service
    replacement: $1
    action: replace
  - source_labels: [__meta_kubernetes_pod_name]
    separator: ;
    regex: (.*)
    target_label: pod
    replacement: $1
    action: replace
  - source_labels: [__meta_kubernetes_pod_container_name]
    separator: ;
    regex: (.*)
    target_label: container
    replacement: $1
    action: replace
  - source_labels: [__meta_kubernetes_service_name]
    separator: ;
    regex: (.*)
    target_label: job
    replacement: ${1}
    action: replace
  - separator: ;
    regex: (.*)
    target_label: endpoint
    replacement: "8000"
    action: replace
  - source_labels: [__address__]
    separator: ;
    regex: (.*)
    modulus: 1
    target_label: __tmp_hash
    replacement: $1
    action: hashmod
  - source_labels: [__tmp_hash]
    separator: ;
    regex: "0"
    replacement: $1
    action: keep
  kubernetes_sd_configs:
  - role: endpoints
    kubeconfig_file: ""
    follow_redirects: true
    enable_http2: true
```

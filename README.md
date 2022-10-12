# Multicluster Monitoring System

##### - Using prometheus operator and thanos as a sidecar to collect metrics from each cluster then add these thanos data collectors to main thanos which will act as a data aggregator connected to grafana 

### No tls Setup :
- Install the Prometheus Operator on each data producer

```helm install prometheus-operator   --set prometheus.thanos.create=true   --set operator.service.type=ClusterIP   --set prometheus.service.type=ClusterIP   --set alertmanager.service.type=ClusterIP   --set prometheus.thanos.service.type=NodePort --set node-exporter.hostNetwork=false  --set prometheus.externalLabels.cluster="kube-codescalers-01"   bitnami/kube-prometheus --namespace monitoring --create-namespace```

use this command to get the public ip:port of thanos operator 

```kubectl get svc --namespace monitoring | grep prometheus-operator-kube-p-prometheus-thanos```
if that command didn't work, check network service, pod prometheus-operator-kube-p-prometheus-thanos will be exposed as node port


- Install and configure Thanos on data aggregator cluster

```helm install thanos bitnami/thanos --values thanos_query_value.yaml --namespace thanos --create-namespace```


- Install Grafana

```helm install grafana bitnami/grafana --values grafana_values.yaml  --set admin.password=GRAFANA-PASSWORD```


- Configure Grafana to use Thanos as a data source

use that link in prometheus datasource http://thanos-query.thanos.svc.cluster.local:9090


### Mutual tls setup

- Create client-server Certs using grpc

	- Clone repo https://github.com/grpc/grpc-java/ 
	- ``` cd grpc-java/testing/src/main/resources/certs/```
	- Follow instruction in [README](https://github.com/grpc/grpc-java/blob/master/testing/src/main/resources/certs/README) file.

#### Load Certs into kube clusters as secret

 ##### installing on data producer cluster

- ```kubectl create secret generic peter-config-tbd --from-file=ca.crt=ca.key --from-file=client-key.crt=client.key --from-file=client-cert.crt=client.pem -n monitoring```

- ```helm install prometheus-operator -f thanos-sidecar_mutula-tls_value.yaml bitnami/kube-prometheus --namespace monitoring --create-namespace```


 ##### Installing on data aggregator cluster

- ```kubectl create secret generic peter-config-tbd --from-file=ca.crt=ca.key --from-file=server1-key.crt=server1.key --from-file=server1-cert.crt=server1.pem -n thanos```

- ```helm install thanos bitnami/thanos --values thanos_mutula-tls_value.yaml --namespace thanos --create-namespace```

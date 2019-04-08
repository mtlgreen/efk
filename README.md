# EFK
Elastiscsearch, Kibana and Fluent Bit k8s dev deployment

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop) with Kubernetes enabled

- Helm and Tiller [installed](https://helm.sh/docs/using_helm/#installing-helm)


## Install

Create the `efk` namespace where Elasticsearch, Fluent Bit and Kibana will be deployed to.

```
kubectl create -f efk-ns.yml
```

### Deploy Elasticsearch

We will disable persistence for simplicity. Warning this will consume a lot of memory in your cluster.

```
helm repo update

helm install --name elasticsearch stable/elasticsearch \
    --set master.persistence.enabled=false \
    --set data.persistence.enabled=false \
    -f elasticsearch-values.yaml \
    --namespace efk
```

To access:

```
export ELASTICSEARCH_POD_NAME=$(kubectl get pods --namespace efk -l "app=elasticsearch,component=client,release=elasticsearch" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward --namespace efk $ELASTICSEARCH_POD_NAME 9200:9200
```

Visit http://127.0.0.1:9200 to use Elasticsearch


### Deploy Kibana

```
helm install --name kibana stable/kibana \
    -f kibana-values.yaml \
    --namespace efk
```

To access the Kibana dashboard:

```
export KIBANA_POD_NAME=$(kubectl get pods --namespace efk -l "app=kibana,release=kibana" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward --namespace efk $KIBANA_POD_NAME 5601:5601
```

Visit http://127.0.0.1:5601 to use Kibana


## References

- https://medium.com/@jbsazon/aggregated-kubernetes-container-logs-with-fluent-bit-elasticsearch-and-kibana-5a9708c5dd9a

- https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes

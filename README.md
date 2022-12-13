# Elastic Cloud Kubernetes

Notes about the deployment of ECK using official Elastic Operator.

## Deploy K8S Cluster [Documentation](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html#k8s-deploy-eck)

For this tests we are using the namespace: eck

1. Install CRDs
```bash
kubectl create -f https://download.elastic.co/downloads/eck/2.5.0/crds.yaml
```

2. Install operator RBAC

```bash
kubectl apply -f https://download.elastic.co/downloads/eck/2.5.0/operator.yaml
```

3. Monitor

```bash
kubectl -n elastic-system logs -f statefulset.apps/elastic-operator
```

## Create a namespace

```bash
kubectl apply -f 1-namespace.yml
```

## Deploy an Elasticsearch clusteredit [Link to docs](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-elasticsearch.html#k8s-deploy-elasticsearch)

2. Apply Elasticsearch.yml cluster specification. This will run with one node, unless the yaml file gets updated.

```bash
kubectl apply -f 2-elasticsearch.yml -n eck
```

3. Monitor health and creation progress

```bash
kubectl get elasticsearch -n eck
```

### Access Elasticseach

```bash
kubectl get service quickstart-es-http -n eck
```

1. Get credentials

```bash
PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}' -n eck)
```

2. Request the Elasticsearch endpoint.

From inside the Kubernetes cluster:

```bash
curl -u "elastic:$PASSWORD" -k "https://quickstart-es-http:9200"
```

alternatively you can use port-forward

```bash
kubectl port-forward service/quickstart-es-http 9200 -n eck
```

then request:

```bash
curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
```

expected output:

```json
{
  "name" : "quickstart-es-default-0",
  "cluster_name" : "quickstart",
  "cluster_uuid" : "dEWglsPOTV2LVZQHxdMzJw",
  "version" : {
    "number" : "8.5.3",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "4ed5ee9afac63de92ec98f404ccbed7d3ba9584e",
    "build_date" : "2022-12-05T18:22:22.226119656Z",
    "build_snapshot" : false,
    "lucene_version" : "9.4.2",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Deploy Kibana [Link to docs](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-kibana.html#k8s-deploy-kibana)

> Make sure to have `Elasticsearch` first.

1. SImilar than Elasticsearch we need to apply the operator manifest

```bash
kubectl apply -f 3-kibana.yml -n eck
```

2. Monitor

```bash
kubectl get kibana -n eck
```

### Access Kibana

```bash
kubectl get service quickstart-kb-http -n eck
```

Open https://localhost:5601 in your browser. 

> Your browser will show a warning because the self-signed certificate configured by default is not verified by a known certificate authority and not trusted by your browser. You can temporarily acknowledge the warning for the purposes of this quick start but it is highly recommended that you configure valid certificates for any production deployments.

If you're unable to access, run a port-forward.

```bash
kubectl port-forward service/quickstart-kb-http 5601 -n eck
```

Login as the `elastic` user. The password can be obtained with the following command::

```bash
kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' -n eck | base64 --decode; echo
```


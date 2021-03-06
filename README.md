## Requirements

- [just](https://github.com/casey/just)
- [minikube](https://minikube.sigs.k8s.io)

## Getting Started

### [IMPORTANT!] Enable Minikube Ingress

```sh
$ minikube addons enable ingress
```

### Create the `logging` Namespace

```sh
$ just create-namespace
```

### Start Elasticsearch

```sh
$ just start-elasticsearch
```

### Start fluentd

```sh
$ just start-fluentd
```

### Start Kibana

```sh
$ just start-kibana
```

### Add `kube.xyz` to `/etc/hosts`

```sh
$ minikube ip
192.168.49.2

$ sudo echo "192.168.49.2 kube.xyz" >> /etc/hosts
```

### View Kibana dashboard

Open a browser and visit `http://kube.xyz/kibana`. You should see the Kibana dashboard.

Next, go to the Discover tab in the sidebar and proceed to create a new Index Pattern: `logstash-*` and `@timestamp`.

Proceed back to the Discover tab to see the logs!

## Execute requests against Elasticsearch

Ensure that Ingress is running. Then make a request to `kube.xyz`:

```sh
$ curl kube.xyz
{
  "name" : "elasticsearch-cluster-0",
  "cluster_name" : "elasticsearch-cluster",
  "cluster_uuid" : "5f6Lf1P4TJCXPPsHrTUT1g",
  "version" : {
    "number" : "7.15.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "93d5a7f6192e8a1a12e154a2b81bf6fa7309da0c",
    "build_date" : "2021-11-04T14:04:42.515624022Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

---

Alternatively, we can make requests within the K8s cluster by exec-ing into a Pod first:

```sh
$ kubectl exec -it dnsutils -n logging -- bash
```

Then make requests against the Elasticsearch service:

```sh
# Target the service
$ curl elasticsearch:9200
```

```sh
# Target a specific Pod by domain name
$ curl elasticsearch-cluster-0.elasticsearch:9200
```

```sh
# Target a specific Pod by IP address
$ kubectl get pods -n logging -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
elasticsearch-cluster-0   1/1     Running   0          36m   172.17.0.2   minikube   <none>           <none>

$ curl 172.17.0.2:9200
```

## Digital Ocean Gotchas

Create node pools with **at least 2GB RAM per node**. Otherwise, Elasticsearch pods will crash continuously.

I created nodes with 4GB RAM and 2 vCPUs ($20/month), which worked well for me.

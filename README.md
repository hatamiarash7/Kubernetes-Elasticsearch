# Kubernetes Elasticsearch
 Deploy Elasticsearch in Kubernetes

## Install

Before we start, one needs to know that Elasticsearch best-practices recommend to separate nodes in three roles:
* `Master` nodes - intended for clustering management only, no data, no HTTP API
* `Client` nodes - intended for client usage, no data, with HTTP API
* `Data` nodes - intended for storing and indexing your data, no HTTP API

Deploy ServiceAccount and Services first :

```
kubectl create -f service-account.yml
kubectl create -f service-master.yml
kubectl create -f service-client.yml
```

For RCs we have to deploy them **in order**

```
kubectl create -f master.yml
kubectl create -f client.yml
kubectl create -f data.yml
```

If your cluster has the RBAC authorization mode enabled, create the additional `Role` and `RoleBinding` with :

```
kubectl create -f rbac.yml
```

## Test the service

*Don't forget* that services in Kubernetes are only accessible from containers in the cluster. For different behavior you should [configure the creation of an external load-balancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)

```
$ kubectl get svc elasticsearch
NAME              TYPE             CLUSTER-IP        EXTERNAL-IP     PORT(S)            AGE
elasticsearch     LoadBalancer     10.110.28.138     localhost       9200:32298/TCP     138m
```

From any host on your cluster (that's running `kube-proxy`), run:

```
curl http://10.110.28.138:9200
```

You should see something similar to the following:


```json
{
  "name" : "113902b2-0b40-41ff-966f-ac3732a34341",
  "cluster_name" : "myesdb",
  "cluster_uuid" : "uKHKl0M1SP-NJdVLX21VJQ",
  "version" : {
    "number" : "6.4.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "04711c2",
    "build_date" : "2018-09-26T13:34:09.098244Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

Check cluster information:


```
curl http://10.110.28.138:9200/_cluster/health?pretty
```

You should see something similar to the following:

```json
{
  "cluster_name" : "myesdb",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

## Scaling

Scaling each type of node to handle your cluster is as easy as:

```
kubectl scale --replicas=3 rc es-master
kubectl scale --replicas=3 rc es-client
kubectl scale --replicas=3 rc es-data
```

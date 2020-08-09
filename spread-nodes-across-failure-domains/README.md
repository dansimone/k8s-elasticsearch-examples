# Spreading Nodes Across Failure-Domains

This example of an Elasticsearch cluster using [ECK](https://github.com/elastic/cloud-on-k8s) that 
spreads Elasticsearch nodes of the various types across Kubernetes failure-domains.

Note that for each type of node, we are using `preferredDuringSchedulingIgnoredDuringExecution` rather than 
`requiredDuringSchedulingIgnoredDuringExecution`.  Using `requiredDuringSchedulingIgnoredDuringExecution` would guarantee
that the first <number_of_failure_domains> nodes of each type get spread across failure-domains, but would not allow
each node type to scale any further.  `preferredDuringSchedulingIgnoredDuringExecution` provides less of an anti-affinity 
guarantee, but allows the Elasticsearch cluster to scale in the future.

### Install ECK

Install as described [here](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html)

### Deploy Elasticsearch cluster

```
kubectl apply -f elasticsearch-cluster.yaml
```

### Observe Anti-Affinity Effects

```
kubectl get pod -o wide
NAME                                        READY   STATUS     RESTARTS   AGE     IP             NODE         NOMINATED NODE   READINESS GATES
example-cluster-es-data-0                   1/1     Running    0          4m25s   10.244.3.196   10.0.10.10   <none>           <none>
example-cluster-es-data-1                   1/1     Running    0          4m25s   10.244.0.250   10.0.11.11   <none>           <none>
example-cluster-es-data-2                   1/1     Running    0          4m25s   10.244.1.48    10.0.12.7    <none>           <none>
example-cluster-es-ingest-0                 1/1     Running    0          3m55s   10.244.0.251   10.0.10.11   <none>           <none>
example-cluster-es-masters-0                1/1     Running    0          6m52s   10.244.0.249   10.0.10.18   <none>           <none>
example-cluster-es-masters-1                1/1     Running    0          6m52s   10.244.1.47    10.0.11.4    <none>           <none>
example-cluster-es-masters-2                1/1     Running    0          6m52s   10.244.3.194   10.0.12.13   <none>           <none>
```

In this case:
- For Elasticsearch masters: worker nodes 10.0.10.10, 10.0.11.11, and 10.0.12.7 reside in different failure-domains
- For Elasticsearch data nodes: worker nodes 10.0.10.18, 10.0.11.4, and 10.0.12.13 reside in different failure-domains
tags: #kubernetes #elasticsearch 

# Elasticsearch

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---

## check cluster health

```bash
# get password
PASSWORD=$(kubectl get secrets ${secretname} -o jsonpath='{.data.elastic}' | base64 -d)

# port-forward svc
kubectl port-forward svc/${svc-http} 9200:9200

# get cluster health (column status should be "green")
curl http://localhost:9200/_cat/health\?v -u elastic:${PASSWORD}

# get indices (column health should be "green")
curl http://localhost:9200/_cat/indices\?v -u elastic:${PASSWORD}

# get shards
curl http://localhost:9200/_cat/shards\?v -u elastic:${PASSWORD}
```

## create user password

```bash
# create user locally
elasticsearch-users useradd ${USERNAME} -p 'password'

# get password
cat /usr/local/etc/elasticsearch/users
```

## cluster reroute API
- docs: https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html
- manually reallocate a shard to a node -> if it fails, add `?retry_failed=true`

```bash
curl -u elastic:${PASSWORD} -X POST "localhost:9200/_cluster/reroute?retry_failed=true&pretty" -H 'Content-Type: application/json' -d'
{
  "commands": [
    {
      "allocate_replica": {
        "index": "search", "shard": 1,
        "node": "data-1"
      }
    }
  ]
}
'
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
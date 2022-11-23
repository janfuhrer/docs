# docs/kubernetes: elasticsearch
#kubernetes #elasticsearch

## check cluster health

```bash
# get password
PASSWORD=kubectl get secrets ${secretname} -o jsonpath='{.data.elastic}' | base64 -d

# port-forward svc
kubectl port-forward svc/${svc-http} 9200:9200

# get cluster health (column status should be "green")
curl http://localhost:9200/_cat/health\?v -u elastic:${PASSWORD}

# get indices (column health should be "green")
curl http://localhost:9200/_cat/indices\?v -u elastic:${PASSWORD}
```

## create user password

```bash
# create user locally
elasticsearch-users useradd ${USERNAME} -p 'password'

# get password
cat /usr/local/etc/elasticsearch/users
```
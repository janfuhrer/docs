# docs: kubernetes/snippets
#kubernetes #snippets 

## links
- Prometheus Relabeler: https://relabeler.promlabs.com

## cmdline without ps
- get cmdline in linux file structure `/proc`
- try with PID 1, if the first process starts another binary like `tiny`, choose another low PID

```bash
cat /proc/PID/cmdline | xargs -0 echo
```

## show environment of running process
```bash
cat /proc/PID/environ | tr '\0' '\n'
```

## kubernetes secret & base64
- create a secret without the trailing `\n`
```bash
echo -n "${password}" | base64
```

- get secret base64 decoded (replace `.data.key`)
```bash
kubectl get secret ${secretname} -o go-template="{{.data.key | base64decode}}"
```

## kubectl auth
```bash
kubectl auth can-i get secret/${secretname} --as=system:serviceaccount:${namespace}:${serviceaccount} -n ${namespace}
```

## curl kubernetes api
- example with ServiceAccount

```bash
SERVICE_ACCOUNT=
NAMESPACE=
API=

# get token of SA
SECRET=$(kubectl get serviceaccount ${SERVICE_ACCOUNT} -n ${NAMESPACE} -o json | jq -Mr '.secrets[].name | select(contains("token"))')
TOKEN=$(kubectl get secret ${SECRET} -n ${NAMESPACE} -o json | jq -Mr '.data.token' | base64 -d)

# get k8s-api ca cert
kubectl get secret ${SECRET} -n ${NAMESPACE} -o json | jq -Mr '.data["ca.crt"]' | base64 -d > ca.crt

# example request for a secret
curl -s https://${API}/api/v1/namespaces/${NAMESPACE}/secrets/${SECRET} --header "Authorization: Bearer $TOKEN" --cacert ca.crt | jq -Mr '.data'
```

## get available tags from registry
```bash
curl -X GET -u ${user}:${pass} https://${registry}/v2/${repository}/tags/list
```

## capsule
- get storage requests over all namespaces of a tenant (works only with `Gi`)
```bash
kubectl get tenants.capsule.clastix.io ${TENANT_NAME} -o jsonpath='{.status.namespaces}' | jq -rc '.[]' | while read i; do; kubectl get pvc -n $i -o jsonpath='{.items[*].status.capacity.storage}' | sed "s/Gi/\n/g"; done | awk '{s+=$1} END {printf "%.0f", s}'
```

## loki filter
#loki

```bash
# keycloak
{clustername="clustername",app="keycloak"} |= "appname"

# ingress
{clustername="clustername",namespace="-system",instance="ingress-class"} |= "app.domain"
```
 
## ksniff
#ksniff

Github: https://github.com/eldadru/ksniff

```bash
# use right context
kubectl config use-context ${k8s-context-name}

# start local proxy
proxyon

# start sniffing
kubectl sniff -n ${namespace} ${pod} -o - > test.pcap

# open Wireshark (not live preview)
/Applications/Wireshark.app/Contents/MacOS/Wireshark -r test.pcap
```

## pvc stuck in terminating
```bash
kubectl patch pvc ${pvc-name} -p '{"metadata":{"finalizers":null}}'
```

## HAProxy
#HAProxy

```bash
# show configfile
haproxy -c -f haproxy.cfg

# show socket information
hatop -s /var/run/hap_${servicename}.sock
```

## tunnel-service
#ssh
1. Open local port and tunneling to the ssh-gateway
```bash
ssh -L 30765:localhost:30765 ${ssh-gateway}
```
2. Start a port-forward
```bash
kubectl get svc
kubectl port-forward svc/${service} 30765:80
```

### nextcloud
#nextcloud

- execute php commands in docker container
```bash
kubectl exec -it -n nextcloud ${pod} -- bash

## turn maintenance mode off
su -s /bin/bash www-data -c "php occ maintenance:mode --off"

## add missing indices after upgrade
su -s /bin/bash www-data -c "php occ db:add-missing-indices"
```

## binary
- check if dynamically linked
```bash
ldd ${binary}
```
-> e.g. with go, there can be linked dependencies to glibc
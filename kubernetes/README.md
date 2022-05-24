# docs: kubernetes
## snippets
### cmdline without ps
- try with PID 1, if the first process starts another binary like `tiny`, choose another low PID

```bash
cat /proc/PID/cmdline | xargs -0 echo
```

### kubectl auth
```bash
kubectl auth can-i get secret/${secretname} --as=system:serviceaccount:${namespace}:${serviceaccount} -n ${namespace}
```

### get available tags from registry
```bash
curl -X GET ${user}$:${pass} https://${registry}/v2/${repository}/tags/list 
```

### loki filter
```bash
# keycloak
{clustername="clustername",app="keycloak"} |= "appname"

# ingress
{clustername="clustername",namespace="-system",app_kubernetes_io_instance="ingress-class"} |= "app.domain"
```
 
### ksniff
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

### pvc stuck in terminating
```bash
kubectl patch pvc ${pvc-name} -p '{"metadata":{"finalizers":null}}'
```

### HAProxy
```bash
# show configfile
haproxy -c -f haproxy.cfg

# show socket information
hatop -s /var/run/hap_<servicename>.sock
```

### Tunnel-Service
1. Open local port and tunneling to the ssh-gateway
```bash
ssh -L 30765:localhost:30765 sshGW
```
2. Start a port-forward
```bash
kubectl get svc
kubectl port-forward svc/<service> 30765:80
```

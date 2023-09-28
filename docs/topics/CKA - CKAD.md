tags: #kubernetes #cka #ckad

# CKA - CKAD

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---
## Exam getaways
```bash
# log kubeproxy systemd
/var/log/kube-proxy.log

# logs journald
journalctl -u etcd.service -l

# test networkpolicy
kubectl run test-np --image=busybox --rm -it -- sh
nc -z -v -w 2 np-test-service 80

# get network plugin used
ps -aux | grep /usr/bin/kubelet # get --network-plugin=xxx

# identify the plugin
ls /etc/cni/net.d/

# coredns-configuration is saved in configmap
# check if kube-dns-service! is working properly (has endpoints, ...)
kubectl -n kube-system get ep kube-dns

# logs pod (--previous)

# sort and custom colums
kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage
```

## Snippets from course

### imperative vs declarative
**imperative**: 
- step by step instruction
- what is required, what and how have to be done
- tell Kubernetes what to do (you have to check yourself)
- create objects: `kubectl run/create/expose`
- update objects: `kubectl edit/scale/set`

**declarative**: 
- destination
- system/software knows how to build and how to update
- command: `kubectl apply`
- look at the existing state and decide, what have to be done to reach the desired state
- for changes: compare with the live-object
- for removing fields: compare to the last applied configuration
- the **last applied configuration** is added by `kubectl apply` as annotation to the live-object

### Headless Service
- has no IP on its own
- only creates DNS A-Record for all other pods (`podname.headless-servicename.namespace.svc.cluster.local`)

### Deployments Strategie
- **Recreate**: Scale to zero, then scale back up
- **RollingUpdate**: Scale 1 down, Scale 1 new up, etc.

```bash
# update image of deployment
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1

# rollout command
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
```

### Commands & Arguments
- `command` overwrites the `ENTRYPOINT` of the Dockerfile
- `args` overwrites the `CMD` of the Dockerfile

### JSON Path
```bash
# get names of nodes
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'

# get all osImage-Info of each node
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}'

# get all names of kubeconfig users
kubectl config view --kubeconfig=/root/my-kube-config -o=jsonpath='{$.users[*].name}'

# sort, output in json!
kubectl get pv --sort-by=.spec.capacity.storage

# sort and custom colums
kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage

# get context name
kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}"
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
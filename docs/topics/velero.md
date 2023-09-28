tags: #kubernetes #velero

# Velero

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---

Docs: https://velero.io/docs/main/

install
```bash
brew install velero
```

## restore only pvc data from velero backup
1. Create temporary namespace and restore a specific namespace

```bash
kubectl create ns temp

velero restore create --from-backup ${backupname} --include-namespaces ${namespace} --namespace-mappings ${namespace}$:temp --wait

# wait until restore is completet (init-container with restic restore!)

# scale statefulset and deployments to 0
kubectl scale -n temp sts ${statefulset} --replicas 0
kubectl scale -n temp deployment ${deployment} --replicas 0
```

2. Start a debug pod and copy all data from a pvc to the local machine

- `pv-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
  - name: debugger
    image: busybox
    command: ['sleep', '3600']
    resources:
      requests:
        cpu: 0
        memory: 0
    volumeMounts:
    - mountPath: /data
      name: data
    - mountPath: /backup
      name: backup
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: ${data}
  - name: backup
    persistentVolumeClaim:
      claimName: ${backup}
```

- copy files from pvc's

```bash
kubectl apply -f pv-pod.yaml -n temp

kubectl cp -n temp pv-pod:/data ./data
kubectl cp -n temp pv-pod:/backup ./backup

kubectl delete ns temp
```

## restore only certificates from velero backup
- only kubernetes tls certificates
```bash
kubectl create ns temp

velero restore create --from-backup ${backupname} --include-namespaces ${namespace} --include-resources secrets --namespace-mappings ${namespace}:temp --wait

kubectl get secrets -n temp ${secretname} -o yaml > cert.yaml

kubectl apply -f cert.yaml -n ${namespace}
```

- cert-manager resources
```bash
velero restore create --from-backup ${backupname} --include-namespaces ${namespace} --include-resources '*.cert-manager.io' --exclude-resources=orders.acme.cert-manager.io,challenges.acme.cert-manager.io,certificaterequests.cert-manager.io --wait
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
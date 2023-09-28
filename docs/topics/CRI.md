tags: #kubernetes #cri

# CRI

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---
## crictl

```bash
sudo crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock ps -a
```

# dockerd

```bash
# list namespaces
ctr ns ls

# list container in ns k8s.io
ctr -n k8s.io containers ls
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
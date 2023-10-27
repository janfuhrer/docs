tags: #kubernetes #dns

# DNS

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---

# Debugging

There are some known issues: https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/#known-issues

## Alpine

> If you are using Alpine version 3.17 or earlier as your base image, DNS may not work properly due to a design issue with Alpine. Until musl version 1.24 didn't include TCP fallback to the DNS stub resolver meaning any DNS call above 512 bytes would fail.

## Kubelet

> Linux's libc (a.k.a. glibc) has a limit for the DNS `nameserver` records to 3 by default and Kubernetes needs to consume 1 `nameserver` record. This means that if a local installation already uses 3 `nameserver`s, some of those entries will be lost. To work around this limit, the node can run `dnsmasq`, which will provide more `nameserver` entries. You can also use kubelet's `--resolv-conf` flag.

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
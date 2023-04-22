# docs: kubernetes/CKS
#kubernetes #cks

Various topics learned in the [CKS](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/) course.

# Cluster Setup and Hardening

## kube-bench

> Checks whether Kubernetes is deployed according to security best practices as defined in the CIS Kubernetes Benchmark

[Git-Repository](https://github.com/aquasecurity/kube-bench)

## ServiceAccount tokens

> Versions of Kubernetes before v1.22 automatically created long term credentials for accessing the Kubernetes API. This older mechanism was based on creating token Secrets that could then be mounted into running Pods. In more recent versions, including Kubernetes v1.26, API credentials are obtained directly by using the [TokenRequest](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/) API, and are mounted into Pods using a [projected volume](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#bound-service-account-token-volume). The tokens obtained using this method have bounded lifetimes, and are automatically invalidated when the Pod they are mounted into is deleted.

Link: [Manually create an API token for a ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-an-api-token-for-a-serviceaccount)

```bash
# create sa
kubectl create sa test

# using the TokenRequest API
kubectl create token test

# create a long-lived API token as Secret
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
  annotations:
    kubernetes.io/service-account.name: test
type: kubernetes.io/service-account-token
EOF
```

## Projected Volumes

[Kubernetes Docs](https://kubernetes.io/docs/concepts/storage/projected-volumes/)

> A `projected` volume maps several existing volume sources into the same directory.
> Currently, the following types of volume sources can be projected:
> - secret
> - downwardAPI
> - configMap
> - serviceAccountToken
>
> All sources are required to be in the same namespace as the Pod. For more details, see the [all-in-one volume](https://git.k8s.io/design-proposals-archive/node/all-in-one-volume.md) design document.

**Benefits**
- Build a directory that contain multiple types of configuration and secret data (e.g. a directory containing both config files from configmaps and credentials from secrets)
- easier than use of multiple volumes & volumeMounts ([Example]([https://blog.sebastian-daschner.com/entries/multiple-kubernetes-volumes-directory](https://blog.sebastian-daschner.com/entries/multiple-kubernetes-volumes-directory "https://blog.sebastian-daschner.com/entries/multiple-kubernetes-volumes-directory"))) -> Problem: A container using a ConfigMap as a [`subPath`](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath "https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath") volume mount will not receive ConfigMap updates.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: projected-volume
      mountPath: "/projected-volume"
    - name: config-volume
      mountPath: "/config-volume"
  volumes:
  - name: projected-volume
    projected:
      sources:
      - configMap:
          name: myconfig
      - secret:
          name: mysecret
  - name: config-volume
    configMap:
      name: myconfig
```

## View Logs

```bash
# service logs
journalctl -u etcd.service -l

# kubeadm (pod logs)
kubectl logs etcd-master

# if kubectl doesn't work
docker ps -a
docker logs <container-id>

# cri-o
crictl ps -a
crictl logs <container-id>
```

## Secure Kubelet

If anonymous access is allowed:

```bash
# full access on api
curl -sk https://localhost:10250/pods | jq

# read only access
curl -sk http://localhost:10255/metrics
```

Configuration: 

```bash
# check if a kubelet config-file exists
ps -aux | grep kubelet

# edit configuration-file
vim /var/lib/kubelet/config.yaml
```

- Example: A Kubelet-Parameter `--protect-kernel-defaults` leads to Configuration `protectKernelDefaults`

Configuration hardening:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    # "Unauthorized" if curl on port 10250
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  # "Forbidden" if anonymous enabled
  mode: Webhook # instead of AllwaysAllow
# disable readOnly port 10255
readOnlyPort: 0
```

## Cluster Upgrades

- `kubeadm` didn't install or upgrade `kubelet` on the nodes

### Upgrade cluster

```bash
# upgrade cluster with kubeadm
ssh controlplane01
apt-get upgrade kubeadm=1.26.0-00
kubeadm upgrade plan

# upgrade cluster
kubeadm upgrade apply v1.26.0

# upgrade kubelet
apt-get upgrade kubelet-1.26.0-00

# check version
kubectl get nodes
```

### Upgrade node

```bash
kubectl drain node-1

ssh node-1
apt-get upgrade kubeadm=1.26.0-00
kubeadm upgrade node
apt-get upgrade kubelet=1.26.0-00
systemctl daemon-reload
systemctl restart kubelet

kubectl uncordon node-1
```

## Ingress

### Rewrite target

[Link Nginx Docs](https://kubernetes.github.io/ingress-nginx/examples/rewrite/)

Path from  `<ingress-url>/path` is forwarded to `<svc>/app` (`rewrite-target` replaces `path`)

```yaml
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /app
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
            name: pay-service
            port: 
              number: 8282
```

## Docker Daemon

```bash
# start docker-daemon in foreground
dockerd --debug

# expose daemon as tcp socket
dockerd --host=tcp://<ip-address>:2375

# connect from another host to docker daemon
export DOCKER_HOST="tcp://<ip-address>:2375"
```

- very bad configuration: unencrypted & unauthenticated

### Secure Docker Daemon
- create "Daemon Configuration File" under `/etc/docker/daemon.json`
- default port for encrypted traffic is 2376
- `tls: true` enables authentication
- `tlsverify: true` enables authorization

```json
{
  "debug": true,
  "hosts": ["tcp://<ip-address>:2376"],
  "tls": true,
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem",
  "tlsverify": true,
  "tlscacert": "/var/docker/caserver.pem"
}
```

```bash
# copy certs in ~/.docker

# connect to docker daemon
export DOCKER_HOST="tcp://<ip-address>:2376"
export DOCKER_TLS=true
docker ps
```

## Ciphers and the Kubernetes Control Plane

All the control plane components (API server, controller manager, kubelet, scheduler) have the following two optional arguments:

-   `--tls-min-version` – This argument sets the minimum version of TLS that may be used during connection negotiation. Possible values: `VersionTLS10`, `VersionTLS11`, `VersionTLS12`, `VersionTLS13`, for TLS 1.0 thru TLS 1.3 respectively. The default is `VersionTLS10`.
-   `--tls-cipher-suites` – This argument sets a comma-separated list of cipher suites that may be used during connection negotiation. There are many of these, and the full list may be found on the [api server argument page](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/). If this argument is omitted, the default value is the list provided by the [GoLang cipher suites package](https://go.dev/src/crypto/tls/cipher_suites.go#L53).

`etcd` also has a command line argument to set cipher suites. Thus it is possible to secure api server → etcd communication to use only specific ciphers that they have in common. You would most likely want to select the newest/strongest.

-   `--cipher-suites` – This argument sets a comma-separated list of cipher suites that may be used during connection negotiation. If this argument is omitted, the default value is the list provided by the [GoLang cipher suites package](https://go.dev/src/crypto/tls/cipher_suites.go#L53).

## API Server

[How to Diagnose a Crashed API Server](https://github.com/kodekloudhub/community-faq/blob/main/docs/diagnose-crashed-apiserver.md)

```bash
# Restart `kubelet` so you don't have to wait too long in the following steps
# restart kubelet
systemctl restart kubelet

# Determine if the kubelet can even start the API server
# watch output for 60 seconds
journalctl -fu kubelet | grep apiserver

# Kubelet does launch API server, but it crashes immediately.
# -> look into the pod logs
crictl ps | grep api
crictl logs <container-id>
```

# System Hardening

## Limit Node Access

- least privilege principle
- api-server only via VPN accessable
- only administrator have access to nodes

```bash
# related commands
id
who
last
usermod -s /bin/nologin bob
userdel bob
```

## SSH Hardening

- copy ssh-key to host `ssh-copy-id <user>@<ip-address>`
- edit SSH-Config under `/etc/ssh/sshd_config` 
```bash
# update config
PermitRootLogin no
PasswordAuthentication no

# restart service
systemctl restart sshd
```

## Privilege Escalation in Linux

- `visudo` -> updates `/etc/sudoers`

```bash
# 1. User or Group (bob, %admin)
# 2. Hosts (ALL, localhost)
# 3. User (ALL)
# 4. Command (/bin/ls, ALL)

# user
bob ALL=(ALL:ALL) ALL
alice ALL=(ALL) NOPASSWD: ALL

# group
%admin localhost=/usr/bin/shutdown -r now
```

## Remove Obsolete Packages and Services

```bash
# list all packages
apt list --installed

# list all services
systemctl list-units --type service

# remove a service
systemctl stop <service>
systemctl disable <service>
apt remove <service>
```

## Restrict Kernel Modules

```bash
# loading a module manually
modeprobe <modulename>

# list all loaded modules
lsmod
```

- blacklisting kernel modules in a file `/etc/modprobe.d/<filename>.conf

```
# blacklist some modules
cat <<EOF > /etc/modprobe.d/blacklist.conf
blacklist sctp
blacklist ddcp
EOF

# reboot system
shutdown -r now
lsmod | grep sctp
```

## Disable Open Ports

```bash
# get all open ports
netstat -an | grep -w LISTEN

# search process of port
netstat -natp | grep 9090

# check services file
cat /etc/services | grep -w 53

# check if port is open
nc 127.0.0.1 6443
```

## UFW (Uncomplicated Firewall)

```bash
# install ufw
apt-get install ufw
systemctl enable ufw
systemctl start ufw

# default rules
ufw default allow outgoing
ufw default deny incoming

# add rules
ufw allow from <ip-address> to any port 22 proto tcp
ufw allow from <ip-range> to any port 80 proto tcp
ufw deny 8080
ufw allow 1000:2000/tcp

# activate ufw
ufw enable
ufw status

# delete rules
ufw delete deny 8080
ufw delete <rulenumber> # ufw status numbered

# reset to default
ufw reset

# disable ufw
ufw disable
```

## Linux Syscalls

```bash
# trace syscalls
strace touch /tmp/error.log

# get all syscalls of command
strace -c touch /tmp/error.log

# trace syscall of running process
pidof etcd
strace -p <pid>
```

## Tracing Syscalls on Kubernetes

- [AquaSec Tracee](https://github.com/aquasecurity/tracee)

> Tracee is a runtime security and forensics tool for Linux based cloud deployments. It uses **eBPF** to trace the host OS and applications **at runtime**, and analyzes collected events in order to detect **suspicious behavioral patterns**.

## Seccomp (SecureComputing)

> Seccomp stands for secure computing mode and has been a feature of the Linux kernel since version 2.6.12. It can be used to sandbox the privileges of a process, restricting the calls it is able to make from userspace into the kernel. Kubernetes lets you automatically apply seccomp profiles loaded onto a node to your Pods and containers.

- by default, linux allows all syscalls in user mode
- this can be restricted with seccomp profiles

```bash
# check if seccompo is available
grep -i seccomp /boot/config/config-$(uname -r)

# map syscall number to syscall
grep -w <syscall-number> /usr/include/asm/uinstd_64.h
```

- use in kubernetes: seccompo is disabled by default (`type: Unconfined`)
- add `seccompProfile` in pod-spec but add `allowPrivilegeEscalation` in container-spec -> this prevents the container from acquiring additional rights

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
(...)
```

### Use custom seccomp profile

```bash
# create own profiles
mkdir -p /var/lib/kubelet/seccomp/profiles

cat < EOF >> /var/lib/kubelet/seccomp/profiles/audit.json
{
  "defaultAction": "SCMP_ACT_LOG"
}

cat < EOF >> /var/lib/kubelet/seccomp/profiles/violation.json
{
  "defaultAction": "SCMP_ACT_ERRNO"
}
```

- create pod with custom seccomp-profile

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json
(...)
```

## AppArmor

> AppArmor is a Linux kernel security module that supplements the standard Linux user and group based permissions to confine programs to a limited set of resources. AppArmor can be configured for any application to reduce its potential attack surface and provide greater in-depth defense. It is configured through profiles tuned to allow the access needed by a specific program or container, such as Linux capabilities, network access, file permissions, etc. Each profile can be run in either _enforcing_ mode, which blocks access to disallowed resources, or _complain_ mode, which only reports violations.

```bash
# check if apparmor is running
systemctl status apparmor

# check if enabled
cat /sys/module/apparmor/parameters/enabled
$ Y

# get loaded profile
cat /sys/kernel/security/apparmor/profiles

# see all loaded profiles
aa-status # or apparmor_status
```

### Create custom AppArmor profiles

```bash
# install utils
apt-get install -y apparmor-utils

# example: scan a script
aa-genprof ./add_data.sh

# load a profile
apparmor_parser /etc/apparmor.d/<file>

# disable a profile
apparmor_parser -R /etc/apparmor.d/<file>
ln -s /etc/apparmor.d/<file> /etc/apparmor.d/disable/
```

### AppArmor profiles in Kubernetes

- AppArmor Kernel Module is enabled on all Nodes
- AppArmor Profile must be loaded on all Nodes
- Container Runtime should support AppArmor

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
  annotations:
    container.apparmor.security.beta.kubernetes.io/<container-name>: localhost/<profile-name>
spec:
(...)
```

## Linux Capabilities

> With [Linux capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html), you can grant certain privileges to a process without granting all the privileges of the root user.

- only a few capabilities are available by default

```bash
# check which capabilities are used
getcap /usr/bin/ping

# get capabilities of a running process
getpcaps <pid>
```


# Minimize Microservice Vulnerabilities

## Admission Controllers

- enable an admission controller: extend the `--enable-admission-plugins` parameter of the kube-apiserver configuration
- disable a default admission controller: use parameter `--disable-admission-plugins`
- there are *validating* and *mutating* admission controllers
- the order is first mutating, then validating admission controller

```bash
# get a list of all available admission controllers
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

## OPA (Open Policy Agent)

> Open Policy Agent (OPA) is an open source, general-purpose policy engine that enables unified, context-aware policy enforcement across the entire stack.

- only for authorization, not authentication
- Policy Language is Rego

```bash
# run opa as server in backgroud
opa -s &

# get all policies
curl http://localhost:8181/v1/policies

# load a policy
curl -X PUT --data-binary@filename.rego http://localhost:8181/v1/policies/example1
```

example of a policy:

```
package httpapi.authz
import input
default allow = false
allow {
 input.path == "home"
 input.user == "bob"
} 
```

## OPA in Kubernetes

- create a `ValidatingWebhookConfiguration` to send requests to OPA
- sidecar `kube-mgmt`:
	- replicate Kubernetes resources to OPA: with that we can check in a rule against deployed objects
	- load policies into OPA via Configmap in Kubernetes

## Encrypting Secret Data at Rest

[Link Kubernetes Docs](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

- Secrets are not encrypted in ETCD by default
- add an `EncryptionConfiguration` to the kube-api-server
- after encryption is enabled, only newly created or updated secrets will be encrypted -> manually encrypt all secrets with `kubectl get secrets --all-namespaces -o json | kubectl replace -f -`

## Container Sandboxing

- Containers are not a [**sandbox**](https://en.wikipedia.org/wiki/Sandbox_(computer_security)). While using a single, shared kernel allows for efficiency and performance gains, it also means that container escape is possible with a single vulnerability.
- e.g. we see all processes from every container on the host
- tools already discussed: Seccomp- & AppArmor-Profiles, Capabilities

### gVisor

> **gVisor** is an application kernel, written in Go, that implements a substantial portion of the Linux system surface. It includes an [Open Container Initiative (OCI)](https://www.opencontainers.org/) runtime called `runsc` that provides an isolation boundary between the application and the host kernel. The `runsc` runtime integrates with Docker and Kubernetes, making it simple to run sandboxed containers.

- additional layer isolation between the container and the kernel

### Kata Containers

> Kata Containers is an open source project and community working to build a standard implementation of lightweight Virtual Machines (VMs) that feel and perform like containers, but provide the workload isolation and security advantages of VMs.

- run container as a leightweight VMs
- needs more ressources and nested virtualization, which isn't always supported

### Container Runtime

**Process Example with Docker**: Docker CLI -> Docker Daemon -> containerd -> containerd-shim -> runC -> namespaces/CGroups

- gVisor and Kata Containers use their own *Container Runtime*
- to use one of the tools, we can switch the container runtime

### Using Runtime in Kubernetes

- create a `RuntimeClass` object with the corresponding handler
	- Docker: `runc`
	- gVisor: `runsc`
	- Kata: `kata`

```yaml
apiVersion: node.k8s.io/v1beta1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
```

- use a custom runtimeClass in a pod:

```yaml
apiVersion: v1
kind: pod
metadata:
  name: nginx
spec:
  runtimeClassName: gvisor
(...)
```

## One way SSL vs Mutual SSL

- **One way SSL**: e.g. open a webpage. User Authentication is then implemented mostly with an User/Password
- **Mutual SSL**: the client and server will verify the authenticity of each other through SSL-Certificate

## Pod to Pod encryption

- by default, Pod to Pod communication is unencrypted
- use of ServiceMesh like Istio/Linkerd which enables mTLS in Pod to Pod communication through a sidecar container who acts like a proxy


# Supply Chain Security

## Minimize base image footprint

- base image of docker-image: image with line `FROM scratch`
- **Modular**: each image solves one specific problem
- **Persist State**: store data in external volume or external caching
- **Base image**: use official images, use frequently updated images
- **Slim/ Minimal Images**: use slim/minimal images, use only necessary tools, cleanup, don't include debug tools on production images, use *docker multi-stage build*
- **Vulnerability Scanning**: scan images (e.g. trivy)

## Image Security

- `nginx` image leads to following image-path: `docker.io/library/nginx:latest
	- default library is `library` 
	- default registry is `docker.io`
	- default tag is `latest`
- use private repositories
	- create a custom secret of type `docker-registry`
	- reference secret in the pod spec under `imagePullSecrets`

```bash
kubectl create secret docker-registry secretname
	--docker-username=<username>
	--docker-password=<password>
	--docker-server=<registry-url:port>
	--docker-email=<e-mail>
```

## Whitelist Allowed Registries

- use of `ValidatingAdmissionWebhook` (e.g. OPA) or
- use the `ImagePolicyWebhook` admission controller

- create a `AdmissionConfiguration` 

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: <path-to-kubeconfig>
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: true
```

- enable admission controller and add configuration in the kube-api-server

```bash
--enable-admission-plugins=ImagePolicyWebhook
--admission-control-config-file=/etc/kubernetes/admission-config.yaml
```

## Kubesec

- use [kubesec](https://github.com/controlplaneio/kubesec) for static analyses of kubernetes resources

```bash
# local scan
kubesec scan pod.yaml

# scan by public hosted kubesec endpoint
curl -sSX POST --data-binary @"pod.yaml" https://v2.kubesec.io/scan

# run kubesec as a service locally
kubesec http 8080 &
```

## Trivy

> Trivy is a comprehensive and versatile security scanner. Trivy has _scanners_ that look for security issues, and _targets_ where it can find those issues.

-  [Git-Repository](https://github.com/aquasecurity/trivy)

```bash
# scan an image
trivy image nginx:1.19.0

# filter on severity
trivy image --severity CRITICAL,HIGH nginx:1.19.0

# only display fixable CVEs
trivy image --ignore-unfixed nginx:1.19.0

# scan a local image
docker save nginx:1.19.0 > nginx.tar
trivy image --input nginx.tar

# scan and save result as json to a file
trivy image --input alpine.tar --format json --output /root/alpine.json
```


# Monitoring, Logging and Runtime Security

## Falco

> The Falco Project, originally created by [Sysdig](https://sysdig.com/), is an incubating [CNCF](https://cncf.io/) open source cloud native runtime security tool. Falco makes it easy to consume kernel events, and enrich those events with information from Kubernetes and the rest of the cloud native stack. Falco can also be extended to other data sources by using plugins. Falco has a rich set of security rules specifically built for Kubernetes, Linux, and cloud-native. If a rule is violated in a system, Falco will send an alert notifying the user of the violation and its severity.

**Installation options**
- as package in linux
- as a daemonset with helm

```bash
# check if installed as packet
systemctl status falco

# following falco systemd logs
journalctl -fu falco
```

### Falco Rules

Rule-Structure (see [Rule fields](https://falco.org/docs/reference/rules/rule-fields/)):

```yaml
- rule: <name of the rule>
  desc: <detailed description of the rule>
  condition: <when to filter events matching the rule>
  output: <output to be generated for the event>
  priority: <severity of the event>
```

Example Rule:

```yaml
- rule: Detect Shell inside a container
  desc: Alert if a shell such as bash is open inside the container
  condition: container and proc.name in (linux_shells)
  output: Bash Shell Opened (user=%user.name container=%container.id)
  priority: WARNING
- list: linux_shells
  items: [bash, zsh, ksh, sh, csh]
- macro: container
  condition: container.id != host
```

See [Supported Fields](https://falco.org/docs/reference/rules/supported-fields/) and [Default Macros](https://falco.org/docs/rules/default-macros/)

### Falco Configuration Files

- see [Falco Configuration Options](https://falco.org/docs/reference/daemon/config-options/)
- default configuration file `/etc/falco/falco.yaml` (is logged at startup/ defined systemd file)
- includes rule-files with `rules_file`
- logging default is `text`, can be changed to `json`
- output channels:
	- `stdout`: enabled by default
	- `file`: log to a file
	- `program_output`: external programs like Slack
	- `http`: log to an http endpoint
- overwrite existing rules or add custom rules in the file `/etc/falco/falco_rules.local.yaml`

```bash
# hot reload: reload falco engine & rules without restart falco service
kill -1 $(cat /var/run/falco.pid)
```

Search for Policy, which triggers the event:

```bash
grep -ir 'Package management process launched in container' /etc/falco/
```

## Mutable vs Immutable Infrastructure

- **Inplace Updates**: Infrastructure remains the same, software changes (Mutable Infrastructure) -> can lead to **Configuration Drifts** 
- **Immutable Infrastructure**: Replace whole infrastructure instead of only the software (Immutable Infrastructure)

**Ensure Immutability of Containers at Runtime**
- add securityContext `readOnlyRootFilesystem: true`
- add `emptyDir: {}` directories to allow the container to write files in needed directories
- `privileged` has to be `false` -> can bypass this settings!

## Audit Logs

- disabled by default
- Available Backends: Log-Backend or Webhook-Backend (e.g. for Falco)

**Event-Types**
- RequestReceived
- ResponseStarted
- ResponseComplete
- Panic

### Enable Auditing

- add kube-apiserver parameter `--audit-log-path=/var/log/k8-audit.log` (Log-Backend)
- use audit-policy-file: `--audit-policy-file=/etc/kubernetes/audit-policy.yaml`
- enable retention on age (in days): `--audit-log-maxage=10`
- enable retention on file count: `--audit-log-maxbackup=5`
- enable retention on file size (in MB): `--audit-log-maxsize=100`
- -> add volumes and volumeMounts!

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - (...)
    - --audit-policy-file=/etc/kubernetes/prod-audit.yaml
    - --audit-log-path=/var/log/prod-secrets.log
    - --audit-log-maxage=30
    volumeMounts:
    - mountPath: /etc/kubernetes/prod-audit.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/prod-secrets.log
      name: audit-log
      readOnly: false
  volumes:
  - name: audit
    hostPath:
      path: /etc/kubernetes/prod-audit.yaml
      type: File
  - name: audit-log
    hostPath:
      path: /var/log/prod-secrets.log
      type: FileOrCreate
```

### Audit Policies

- create audit policies in the audit policy file defined with `--audit-policy-file`

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages: ["RequestReceived"] # don't record specific events (Optional)
rules:
- namespaces: ["prod-namespace"]
  verbs: ["delete"]
  resources:
  - groups: " "
    resources: ["pods"]
    resourceNames: ["webapp-pod"]
  level: Metadata # Available Options: None, Metadata, Request, RequestResponse
- level: Metadata
  resources:
  - groups: " "
    resources: ["secrets"]
```

# Overview

Comparison: [https://security.stackexchange.com/a/196888](https://security.stackexchange.com/a/196888 "https://security.stackexchange.com/a/196888")

-   **Seccomp** reduces the chance that a kernel vulnerability will be successfully exploited.
-   **AppArmor** prevents an application from accessing files it should not access.
-   **Capability** dropping reduces the damage a compromised privileged process can do.

# KillerShell

Some other snippets, learned through the CKS Simulator on [killer.sh](https://killer.sh/cks)

- get all context names of kubeconfig
```bash
kubectl config get-contexts -o name
```

- falco check syslog and get pod infos
```bash
cat /var/log/syslog | grep falco | grep nginx | grep process
crictl ps -id 7a5ea6a080d1
crictl pods -id 7a5ea6a080d1
```

 - change api-server service from NodePort to ClusterIP
```bash
# change kube-api parameter as follow
--kubernetes-service-node-port=31000 # delete or set to 0

# delete the kubernetes service
kubectl delete svc kubernetes
```

- enable Pod Security Standard Rules
```bash
# edit namespace and add labels
apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: team-red
    pod-security.kubernetes.io/enforce: baseline # add enforce label
  name: team-red

# delete running pods in namespace to enforce the rule for those
```

- kube-bench
```bash
# on master nodes
kube-bench run --targets=master

# on worker nodes
kube-bench run --targets=node
```

- kubelet add clientCAFile config
```yaml
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
```

- check hash of binary
 ```bash
# echo hash with name of packag, check with binary
echo "$(cat archive.tar.gz.sha256) filename" | sha512sum --check
```

- OPA
```bash
# get crds & objects
kubectl get crd | grep gatekeeper
kubectl get constraint
kubectl get constrainttemplate

# test pod
kubectl run opa-test --image=very-bad-registry.com/image
```

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: BlacklistImages
metadata:
...
spec:
  match:
    kinds:
    - apiGroups:
      - ""
      kinds:
      - Pod
```

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
...
spec:
  crd:
    spec:
      names:
        kind: BlacklistImages
  targets:
  - rego: |
      package k8strustedimages

      images {
        image := input.review.object.spec.containers[_].image
        not startswith(image, "docker-fake.io/")
        not startswith(image, "google-gcr-fake.com/")
        not startswith(image, "very-bad-registry.com/") # ADD THIS LINE
      }

      violation[{"msg": msg}] {
        not images
        msg := "not trusted image!"
      }
    target: admission.k8s.gatekeeper.sh
```

- secure kubernetes dashboard
	- change service of type NodePort to ClusterIP
	- change following parameters
```bash
  template:
    spec:
      containers:
      - args:
        - --namespace=kubernetes-dashboard  
        - --authentication-mode=token        # change or delete, "token" is default
        - --auto-generate-certificates       # add
        #- --enable-skip-login=true          # delete or set to false
        #- --enable-insecure-login           # delete
```

- AppArmor and nodeSelector
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apparmor
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apparmor
  template:
    metadata:
      labels:
        app: apparmor
      annotations:
        container.apparmor.security.beta.kubernetes.io/c1: localhost/very-secure
    spec:
      nodeSelector:
        security: apparmor
```

- check AppArmor profile
```bash
ssh node

crictl pods | grep ${podname} # get pod id
crictl ps -a | grep ${pod-id} # get container id
crictl inspect ${container-id} | grep -i profile # get "apparmorProfile: "very-secure"
```

- runtimeClass & nodeName
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gvisor-test
spec:
  nodeName: cluster1-node2
  runtimeClassName: gvisor
  containers:
  - image: nginx:1.19.2
    name: gvisor-test
```

- read secrets from ETCD (ETCD in Kubernetes stores data under `/registry/{type}/{namespace}/{name}`)
```bash
# go to the controlplan
ssh controlplane

# get etcd connection infos
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd

# read secrets from etcd
ETCDCTL_API=3 etcdctl \
--cert /etc/kubernetes/pki/apiserver-etcd-client.crt \
--key /etc/kubernetes/pki/apiserver-etcd-client.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/${namespace}/${secretname}
```

- curl kubernetes api with serviceaccount token
```bash
curl https://kubernetes.default/api/v1/namespaces/${namespace}/secrets/${secretname} -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" -k
```

- find pod which uses syscall `kill`
```bash
# get node on wich the pods are scheduled
kubectl get pods -o wide
ssh node01

# get container id from pod
crictl ps | grep ${podname}

# get args
crictl insepct ${container-id} | grep -A2 args

# get pid of process
ps -aux | grep ${args}

# strace on  pid, wait a couple of seconds
strace -p ${pid}
```

- audit policy
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:

# log Secret resources audits, level Metadata
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]

# log node related audits, level RequestResponse
- level: RequestResponse
  userGroups: ["system:nodes"]

# for everything else don't log anything
- level: None
```

```bash
# check
cat audit.log | tail | jq


# shows Secret entries
cat audit.log | grep '"resource":"secrets"' | wc -l

# confirms Secret entries are only of level Metadata
cat audit.log | grep '"resource":"secrets"' | grep -v '"level":"Metadata"' | wc -l

# shows RequestResponse level entries
cat audit.log | grep -v '"level":"RequestResponse"' | wc -l

# shows RequestResponse level entries are only for system:nodes
cat audit.log | grep '"level":"RequestResponse"' | grep -v "system:nodes" | wc -l
```

- investigate audit
```bash
# check if user p.auster had access to secret
cat audit.log | grep "p.auster" | wc -l # 28
cat audit.log | grep "p.auster" | grep Secret | wc -l # 2
cat audit.log | grep "p.auster" | grep Secret | grep list | wc -l # 0
cat audit.log | grep "p.auster" | grep Secret | grep get | wc -l # 2

# inspect the actions
cat audit.log | grep "p.auster" | grep Secret | grep get | jq
```

- security risks in Dockerfile
	- Every command creates a new layer and every layer is persistet in the image.
	- This means even if the file `secret-token` get's deleted in layer Z, it's still included with the image in layer X and Y. In this case it would be better to use for example variables passed to Docker.
```
# /opt/course/22/files/Dockerfile-mysql
FROM ubuntu

# Add MySQL configuration
COPY my.cnf /etc/mysql/conf.d/my.cnf
COPY mysqld_charset.cnf /etc/mysql/conf.d/mysqld_charset.cnf

RUN apt-get update && \
    apt-get -yq install mysql-server-5.6 &&

# Add MySQL scripts
COPY import_sql.sh /import_sql.sh
COPY run.sh /run.sh

# Configure credentials
COPY secret-token .                                       # LAYER X
RUN /etc/register.sh ./secret-token                       # LAYER Y
RUN rm ./secret-token # delete secret token again         # LATER Z

EXPOSE 3306
CMD ["/run.sh"]
```

- security risk in deployment: don't expose secrets in environment variables, use secrets instead
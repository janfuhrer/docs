# docs: kubernetes/security
#kubernetes #security

## FairWindsOps
#nova #pluto #goldilocks #polaris

## nova
Github: https://github.com/FairwindsOps/nova
Docs: https://nova.docs.fairwinds.com

> Nova scans your cluster for installed Helm charts, then cross-checks them against all known Helm repositories. If it finds an updated version of the chart you're using, or notices your current version is deprecated, it will let you know.
> 
> Nova can also scan your cluster for out of date container images.

install
```bash
brew tap fairwindsops/tap
brew install fairwindsops/tap/nova
```

use
```bash
# check outdated deployed helm releases
nova find --wide

# check outdated container images
nova find --containers
```

## pluto
Github: https://github.com/FairwindsOps/pluto
Docs: https://pluto.docs.fairwinds.com

> Pluto is a utility to help users find [deprecated Kubernetes apiVersions](https://k8s.io/docs/reference/using-api/deprecation-guide/) in their code repositories and their helm releases.

install
```bash
brew install FairwindsOps/tap/pluto
```

use
```bash
# helm detection in cluster
pluto detect-helm -owide

# file detection in a directory
pluto detect-files -d ${path}

# helm chart checking (local files)
helm template ${chart-dir} | pluto detect -
```

## goldilocks
Github: https://github.com/FairwindsOps/goldilocks
Docs: https://goldilocks.docs.fairwinds.com

> Goldilocks is a utility that can help you identify a starting point for resource requests and limits.

**requirements**
- [vertical-pod-autoscaler (opens new window)](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)configured in the cluster

## polaris
Github: https://github.com/FairwindsOps/polaris
Docs: https://polaris.docs.fairwinds.com

> Polaris runs a variety of checks to ensure that Kubernetes pods and controllers are configured using best practices, helping you avoid problems in the future.

install
```bash
brew tap reactiveops/tap
brew install reactiveops/tap/polaris
```

use
```bash
# run local binary
polaris dashboard --port 8080
```

## checkov
Github: https://github.com/bridgecrewio/checkov

install
```bash
brew install checkov
```

use
```bash
# scan a directory (show only failed checks)
checkov -d ${directory} --compact --quiet
```


## kics
Github: https://github.com/Checkmarx/kics

install
```bash
# clone & build
cd ~/bin
git clone https://github.com/Checkmarx/kics.git
cd kics
make build

## additionaly add alias
alias kics="~/bin/kics/bin/kics"
```

use
```bash
# kick a scan
kics scan -p ${directory} --report-formats json -q ~/bin/kics/assets/queries
```
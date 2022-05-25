# docs: kubernetes/helm
#kubernetes #helm
## manual upload of helm-chart
```bash
# copy index.yaml file
mc cp ${mc-location}/helm-charts/unstable/index.yaml ./

# download specific helm version
curl -O https://get.helm.sh/helm-v3.3.0-linux-amd64.tar.gz
tar -xzf helm-v3.3.0-linux-amd64.tar.gz

# create ${chart.tgz}
helm package .

# update index-file
mkdir packages
mv ${chart.tgz} packages
cp index.yaml packages
./linux-amd64/helm repo index packages --merge packages/index.yaml  --url "https://${s3.domain}/helm-charts/unstable/"

# check index files
diff index.yaml packages/index.yaml

# upload index and chart to s3
mc cp ./packages/* ${mc-location}/helm-charts/unstable/
```

## delete pvc on helm uninstall
By a `helm uninstall` PVCs only are deleted, if they are part of a **Deployment**. PVCs of a **Statefulset** are **not** deleted, there is no way with the helm-command (f.e. `--purge`) → Issue: https://github.com/helm/helm/issues/5156

If we want helm not to delete a pvc from a deployment, we can use the annotation `"helm.sh/resource-policy": keep` (https://helm.sh/docs/howto/charts_tips_and_tricks/#tell-helm-not-to-uninstall-a-resource)

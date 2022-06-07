# docs: kubernetes/flux
#kubernetes #flux

## flux diff
```bash
flux diff kustomization -n ${namespace} ${kustomization-name} --path ./${path-to-entrypoint}
```
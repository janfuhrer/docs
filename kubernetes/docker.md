# docs: kubernetes/docker
#kubernetes #docker
## dockerfile
**best practices**
- Each line in a Dockerfile creates a new layer -> combine commands by using &&
- Insted of using `mv`, directly put files at the right place
- When extracting archives, merge operations in a single layer
- Using Multi-stages builds
## inspect image
Inspect an Image
```bash
docker inspect <image>
```

Export tar-gz file (unpack with `tar -xzf <name>.tgz`)
```bash
docker save -o <name>.tgz image:tag 
```

Inspect the layers of an image
```
brew install dive

dive <image>:<tag>
```

## network
list linux-bridges (first download pakage)
```bash
apt-get install bridge-utils
brctl show
```
tags: #kubernetes #docker

# Docker

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---

## dockerfile
**best practices**
- Each line in a Dockerfile creates a new layer -> combine commands by using `&&`
- Insted of using `mv`, directly put files at the right place
- When extracting archives, merge operations in a single layer
- Use [multi-stages builds](https://docs.docker.com/develop/develop-images/multistage-build/)$

## build docker image on macOS arm

```bash
docker buildx build --platform=linux/amd64 -t imagename:local .
```

## inspect image
Inspect an Image
```bash
docker inspect <image>
```

Export tar-gz file (unpack with `tar -xzf ${name}.tgz`)
```bash
docker save -o ${name}.tgz ${image}:${tag}
```

Inspect the layers of an image
```bash
# install dive
brew install dive

# inspect image with dive
dive ${image}:${tag}
```

## network
list linux-bridges
```bash
# install package
apt-get install bridge-utils

# list bridges
brctl show
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
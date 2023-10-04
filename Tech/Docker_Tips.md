# Docker
#tech-tip #docker
## Multiplatform
#multiarch 
https://docs.docker.com/build/building/multi-platform/

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t [IMAGE] --push .
```
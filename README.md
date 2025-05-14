docker buildx build --platform linux/amd64 -t dundycenic/cobber-nginx:latest --push .
docker run --platform linux/amd64 -p 8080:80 dundycenic/cobber-nginx:latest

# Docker Cheat Sheet

## Environment

```bash
docker version
docker info
docker context ls
docker context show
docker context use <context>
```

## Containers

```bash
docker ps
docker ps -a
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"

docker start <container>
docker stop <container>
docker restart <container>
docker kill <container>

docker rm <container>
docker rm -f <container>              # DANGER
```

Filter:

```bash
docker ps --filter "status=running"
docker ps -a --filter "name=<text>"
docker ps -a --filter "ancestor=<image>"
```

## Logs

```bash
docker logs <container>
docker logs -f <container>
docker logs --tail 200 <container>
docker logs --since 10m <container>
docker logs --since 1h --timestamps <container>
docker logs -f --tail 100 <container>
```

## Shell / exec

```bash
docker exec -it <container> bash
docker exec -it <container> sh
docker exec <container> env
docker exec <container> ps aux
```

Run as root if the image allows it:

```bash
docker exec -u 0 -it <container> sh
```

## Inspect

```bash
docker inspect <container>
docker inspect <image>

docker inspect -f '{{.State.Status}}' <container>
docker inspect -f '{{.State.Health.Status}}' <container>
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container>
docker inspect -f '{{json .Config.Env}}' <container>
```

## Processes and resources

```bash
docker stats
docker stats --no-stream
docker top <container>
docker system df
docker system df -v
```

## Copy files

```bash
docker cp <container>:/path/to/file .
docker cp ./file <container>:/path/to/file
```

## Ports

```bash
docker port <container>
docker inspect -f '{{json .NetworkSettings.Ports}}' <container>
```

## Images

```bash
docker images
docker image ls
docker pull <image>:<tag>
docker tag <source> <target>
docker rmi <image>
docker history <image>
```

Build:

```bash
docker build -t <name>:<tag> .
docker build --no-cache -t <name>:<tag> .
docker build --progress=plain -t <name>:<tag> .
```

BuildKit/buildx:

```bash
docker buildx ls
docker buildx build --platform linux/amd64 -t <name>:<tag> .
docker buildx build --platform linux/amd64,linux/arm64 -t <name>:<tag> --push .
```

## Run ad-hoc containers

```bash
docker run --rm -it alpine sh
docker run --rm -it --entrypoint sh <image>
docker run --rm -p 8080:80 <image>
docker run --rm -e KEY=value <image>
docker run --rm --env-file .env <image>
```

Mount current directory:

```bash
docker run --rm -it -v "$PWD:/work" -w /work <image> sh
```

## Networks

```bash
docker network ls
docker network inspect <network>
docker network create <network>
docker network connect <network> <container>
docker network disconnect <network> <container>
```

## Volumes

```bash
docker volume ls
docker volume inspect <volume>
docker volume create <volume>
docker run --rm -v <volume>:/data alpine ls -la /data
```

## Docker Compose

```bash
docker compose config
docker compose ps
docker compose up
docker compose up -d
docker compose down
docker compose stop
docker compose start
docker compose restart

docker compose logs
docker compose logs -f
docker compose logs -f --tail 200 <service>

docker compose exec <service> sh
docker compose exec <service> bash

docker compose pull
docker compose build
docker compose build --no-cache
docker compose up -d --build
```

Scale:

```bash
docker compose up -d --scale <service>=3
```

## Events and debugging

```bash
docker events
docker events --since 10m
docker diff <container>
docker wait <container>
```

Check restart policy:

```bash
docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' <container>
docker update --restart unless-stopped <container>
```

## Save/load vs export/import

Preserve an image including metadata/layers:

```bash
docker save <image>:<tag> -o image.tar
docker load -i image.tar
```

Export a container filesystem only:

```bash
docker export <container> -o container.tar
docker import container.tar <new-image>:<tag>
```

## Cleanup

```bash
docker container prune
docker image prune
docker image prune -a
docker volume prune
docker network prune
```

**DANGER — broad cleanup:**

```bash
docker system prune
docker system prune -a
docker system prune -a --volumes
```

Always inspect first:

```bash
docker system df -v
```

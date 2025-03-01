```yaml
networks:
  proxy:
    external: true
    name: proxy

services:
  traefik:
    image: traefik:v2.11
    container_name: traefik
    restart: always
    command:
      - "--providers.docker"
      - "--accesslog=true"
      - "--accesslog.filePath=/logs/access.log"
      - "--log.level=DEBUG"
      - "--log.filePath=/logs/treafik.log"
    security_opt:
      - no-new-privileges:true
    ports:
      - "22:2222"
      - "80:80"
      - "443:443"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /data/docker/traefik/data/traefik.yml:/traefik.yml:ro
      - /data/docker/traefik/data/acme.json:/acme.json
      - /data/docker/traefik/data/configurations:/configurations
      - /data/docker/traefik/volumes/logs:/logs
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.traefik-secure.entrypoints=websecure"
      - "traefik.http.routers.traefik-secure.rule=Host(`traefik.valentinhelies.fr`)"
      - "traefik.http.routers.traefik-secure.middlewares=user-auth@file"
      - "traefik.http.routers.traefik-secure.service=api@internal"
```

```yaml
networks:
  proxy:
    external: true
    name: proxy

services:
  registry:
    image: registry:latest
    container_name: registry
    restart: always
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_HTTP_ADDR: "0.0.0.0:5000"
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - /data/docker/registry/volumes/auth:/auth
      - /data/docker/registry/volumes/registry:/var/lib/registry
      - /data/docker/registry/volumes/data:/data
    networks:
      - default
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.registry-secure.entrypoints=websecure"
      - "traefik.http.routers.registry-secure.rule=Host(`registry.valentinhelies.fr`)"
      - "traefik.http.routers.registry-secure.service=registry-secure"
      - "traefik.http.services.registry-secure.loadbalancer.server.port=5000"
```

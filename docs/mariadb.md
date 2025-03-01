```yaml
networks:
  proxy:
    external: true
    name: proxy
  mariadb:
    external: true
    name: mariadb
    driver: bridge

services:
  db:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: pwd
    volumes:
      - /data/docker/mariadb/volumes/data:/var/lib/mysql
    networks:
      - default
      - mariadb

  pma:
    image: phpmyadmin/phpmyadmin
    container_name: pma
    restart: always
    environment:
      PMA_HOST: mariadb
      MARIADB_ROOT_PASSWORD: pwd
    networks:
      - default
      - proxy
      - mariadb
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.pma-secure.entrypoints=websecure"
      - "traefik.http.routers.pma-secure.rule=Host(`pma.valentinhelies.fr`)"
      - "traefik.http.routers.pma-secure.service=pma-secure"
      - "traefik.http.services.pma-secure.loadbalancer.server.port=80"
```

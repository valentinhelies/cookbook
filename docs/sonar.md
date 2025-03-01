```yaml
networks:
  proxy:
    external: true
    name: proxy
  mariadb:
    external: true
    name: proxy

services:
  sonar:
    image: sonarqube:latest
    container_name: sonar
    restart: always
    ports:
      - "9000:9000"
    environment:
      SONARQUBE_JDBC_USERNAME: sonar
      SONARQUBE_JDBC_PASSWORD: pwd
      SONARQUBE_JDBC_URL: "jdbc:mysql://mariadb:3306/sonar"
    volumes:
      - /data/docker/sonar/volumes/data:/opt/sonarqube/data
      - /data/docker/sonar/volumes/extensions:/opt/sonarqube/extensions
      - /data/docker/sonar/volumes/logs:/opt/sonarqube/logs
    networks:
      - default
      - proxy
      - mariadb
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.sonar-secure.entrypoints=websecure"
      - "traefik.http.routers.sonar-secure.rule=Host(`sonar.valentinhelies.fr`)"
      - "traefik.http.routers.sonar-secure.service=sonar-secure"
      - "traefik.http.services.sonar-secure.loadbalancer.server.port=9000"
```

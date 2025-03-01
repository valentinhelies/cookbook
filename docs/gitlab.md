```yaml
networks:
  proxy:
    external: true
    name: proxy
  gitlab:
    name: gitlab

services:
  gitlab:
    image: gitlab/gitlab-ce:17.6.5-ce.0
    container_name: gitlab
    restart: always
    hostname: gitlab.valentinhelies.fr
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url "https://gitlab.valentinhelies.fr"

        # Nginx params to redirect from traefik
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        nginx['proxy_set_headers'] = {
          "X-Forwarded-Proto" => "https",
          "X-Forwarded-Ssl" => "on"
        }

        letsencrypt['enable'] = false

        # SMTP Conf
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = "pro1.mail.ovh.net"
        gitlab_rails['smtp_port'] = 587
        gitlab_rails['gitlab_email_from'] = "gitlab@valentinhelies.fr"
        gitlab_rails['smtp_user_name'] = "gitlab@valentinhelies.fr"
        gitlab_rails['smtp_password'] = "pwd"
        gitlab_rails['smtp_domain'] = "valentinhelies.fr"
        gitlab_rails['smtp_authentication'] = "login"
        gitlab_rails['smtp_enable_starttls_auto'] = true
        gitlab_rails['smtp_tls'] = false
        gitlab_rails['smtp_ssl'] = false
        gitlab_rails['smtp_openssl_verify_mode'] = "none"

        # GITLAB DOCKER IMAGE REGISTRY: so that we can use our docker image registry with gitlab
        registry_external_url 'https://registry.valentinhelies.fr'
        gitlab_rails['registry_enabled'] = true
        gitlab_rails['api_url'] = 'https://registry.valentinhelies.fr'
        registry['enable'] = true
        registry_nginx['enable'] = false
        registry['registry_http_addr'] = "0.0.0.0:5000"
    volumes:
      - /data/docker/gitlab/volumes/gitlab/config:/etc/gitlab
      - /data/docker/gitlab/volumes/gitlab/logs:/etc/log/gitlab
      - /data/docker/gitlab/volumes/gitlab/data:/var/opt/gitlab
    networks:
      - default
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.gitlab-secure.entrypoints=websecure"
      - "traefik.http.routers.gitlab-secure.rule=Host(`gitlab.valentinhelies.fr`)"
      - "traefik.http.routers.gitlab-secure.service=gitlab-secure"
      - "traefik.http.services.gitlab-secure.loadbalancer.server.port=80"

      - "traefik.tcp.routers.gitlab-ssh.rule=HostSNI(`*`)"
      - "traefik.tcp.routers.gitlab-ssh.entrypoints=ssh"
      - "traefik.tcp.routers.gitlab-ssh.service=gitlab-ssh-svc"
      - "traefik.tcp.services.gitlab-ssh-svc.loadbalancer.server.port=22"

      - "traefik.http.routers.registry-secure.entrypoints=websecure"
      - "traefik.http.routers.registry-secure.rule=Host(`registry.valentinhelies.fr`)"
      - "traefik.http.routers.registry-secure.service=registry-secure"
      - "traefik.http.services.registry-secure.loadbalancer.server.port=5000"

  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    restart: always
    volumes:
      - /data/docker/gitlab/volumes/gitlab-runner/config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitlab
```

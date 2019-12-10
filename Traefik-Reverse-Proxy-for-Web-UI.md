The following exposes the Web UI at `http://<container host>/qb` using compose-style syntax (which can be translated to the traefik syntax of your choice)
```yaml
...
services:
  traefik:
    image: traefik
    networks: [traefik-qb-net]
    ports: ["80:80"]
    command: 
      - "--providers.docker=true"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
  qbittorrent:
    image: linuxserver/qbittorrent
    deploy:
      labels:
        - "traefik.enable=true"
        # adding a slash to the end
        - "traefik.http.routers.qb.entrypoints=web"
        - "traefik.http.routers.qb.rule=PathPrefix(`/qb`)"
        - "traefik.http.middlewares.qb-redirect.redirectregex.regex=^(.*)/qb$$"
        - "traefik.http.middlewares.qb-redirect.redirectregex.replacement=$$1/qb/"
        - "traefik.http.middlewares.qb-strip.stripprefix.prefixes=/qb/"
        # appropropriate header changes
        - "traefik.http.middlewares.qb-headers.headers.customrequestheaders.X-Frame-Options=SAMEORIGIN"
        - "traefik.http.middlewares.qb-headers.headers.customrequestheaders.Referer="
        - "traefik.http.middlewares.qb-headers.headers.customrequestheaders.Origin="
        - "traefik.http.routers.qb.middlewares=qb-strip,qb-redirect,qb-headers"
        # loadbalancer to *not* pass the host header
        - "traefik.http.services.qb.loadbalancer.server.port=8080"
        - "traefik.http.services.qb.loadbalancer.passhostheader=false"
        - "traefik.docker.network=traefik-qb-net"
...
```
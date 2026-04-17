# webprotege-nginx

Reverse proxy image for WebProtege. Fronts the UI server, API gateway,
Keycloak, and Kibana on a single HTTP endpoint.

## Contents

- `Dockerfile` — extends the official `nginx:latest` image
- `nginx.conf` — server config copied to `/etc/nginx/conf.d/default.conf`

The config uses lazy DNS (`resolver 127.0.0.11` + `proxy_pass $variable`)
so the proxy starts independently of its upstreams and resolves them on
demand — required for clean cold starts in Docker Compose, where upstream
containers may not exist yet when nginx boots.

## Prerequisites

- Docker

## Build

```bash
docker build -t protegeproject/webprotege-nginx:$(cat VERSION) .
```

The image version is tracked in the `VERSION` file at the repo root and
bumped automatically after each release by the CI workflow.

## Routes

| Location | Upstream | Notes |
|---|---|---|
| `/` | `webprotege-gwt-ui-server:8080` | Rewrites to `/webprotege/…` |
| `/webprotege/` | `webprotege-gwt-ui-server:8080` | |
| `/files/submit`, `/data/` | `webprotege-gwt-api-gateway:7777` | |
| `/wsapps` | `webprotege-gwt-api-gateway:7777` | WebSocket upgrade |
| `/keycloak/` | `webprotege-keycloak:8080` | |
| `/kibana/` | `kibana:5601` | Served under the `/kibana` base path |

## Deployment

See the [webprotege-deploy README](../webprotege-deploy/README.md).

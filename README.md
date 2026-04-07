# docker-caddy

<!-- automd:badges github=umicloud/docker-caddy license -->

[![license](https://img.shields.io/github/license/umicloud/docker-caddy)](https://github.com/umicloud/docker-caddy/blob/main/LICENSE)

<!-- /automd -->

## Environment variables

| Variable | Description | Required |
|----------|-------------|----------|
| `CADDY_EMAIL` | The email address to use for the ACME certificate | ✅ |
| `CLOUDFLARE_API_TOKEN` | The API token to use for the Cloudflare DNS | ✅ |

## Example Caddyfile

```caddyfile [Caddyfile]

## Example Compose file

<!-- automd:file src="./compose.yaml" code -->

```yaml [compose.yaml]
services:
  caddy:
    image: ghcr.io/keksiqc/caddy:latest
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443/tcp
      - 443:443/udp
    environment:
      - CADDY_INGRESS_NETWORKS=caddy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - caddy_data:/data
    networks:
      - caddy
    labels:
      - "caddy.acme_dns=cloudflare {{ env.CLOUDFLARE_API_TOKEN }}"

networks:
  caddy:
    external: true

volumes:
  caddy_data: {}
```

<!-- /automd -->

# docker-caddy

<!-- automd:badges github=umicloud/docker-caddy license -->

[![license](https://img.shields.io/github/license/umicloud/docker-caddy)](https://github.com/umicloud/docker-caddy/blob/main/LICENSE)

<!-- /automd -->

## Environment variables

| Variable               | Description                                       | Required |
| ---------------------- | ------------------------------------------------- | -------- |
| `CADDY_EMAIL`          | The email address to use for the ACME certificate | ✅       |
| `CLOUDFLARE_API_TOKEN` | The API token to use for the Cloudflare DNS       | ✅       |

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

## Default Caddyfile

<!-- automd:file src="./root/etc/caddy/Caddyfile" code -->

``` [Caddyfile]
{
	email {env.CADDY_EMAIL}
	acme_dns cloudflare {env.CLOUDFLARE_API_TOKEN}

	servers {
		trusted_proxies cloudflare {
			interval 12h
			timeout 15s
		}
		trusted_proxies_strict
		client_ip_headers Cf-Connecting-Ip
	}

	log {
		output file /var/log/caddy/access.log {
			roll_size 10MiB
			roll_keep_for 168h
		}
		format json
		level INFO
	}

	crowdsec {
		api_url http://crowdsec:8080
		api_key {env.CROWDSEC_API_KEY}
		appsec_url http://crowdsec:7422
	}
}

(tinyauth) {
	forward_auth tinyauth:8080 {
		uri /api/auth/caddy
	}
}

(logging) {
	log {
		output file /var/log/caddy/{args[0]}.access.log {
			roll_size 10MiB
			roll_keep_for 168h
		}
		format json
		level INFO
	}
}

:3000 {
	handle /health {
		respond "ok" 200
	}

	respond 404
}
```

<!-- /automd -->

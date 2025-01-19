# Caddy with Cloudflare DNS Container
This repository provides a Docker container image of Caddy 2 with the Cloudflare DNS provider module pre-installed. This setup enables Caddy to obtain SSL certificates through DNS-01 challenge verification without exposing your internal services to the internet.

## Why This Container?

- The official Caddy Docker image doesn't include DNS provider modules by default
- DNS-01 challenge verification is required for obtaining SSL certificates when your services aren't publicly accessible
- This container comes with the Cloudflare DNS provider module pre-installed, eliminating the need to build your own image

## Automatic Updates
A GitHub Actions workflow runs daily to:

Check for updates to the official Caddy latest Docker image
Automatically rebuild this container with the latest Caddy version when updates are available
Push the updated image to this repository's container registry

## Usage
You can use this image by specifying the following:
```
ghcr.io/nathancatania/caddy-cloudflare:latest
```

On how to *actually* use this image in practice, see [this excellent post on the Caddy Community forum](https://caddy.community/t/how-to-guide-caddy-v2-cloudflare-dns-01-via-docker/8007).

Example `docker-compose.yml`:
```yaml
services:
  caddy:
    image: ghcr.io/nathancatania/caddy-cloudflare:latest
    container_name: caddy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /volume1/configs/caddy/Caddyfile:/etc/caddy/Caddyfile:ro
      - /volume1/configs/caddy/data:/data
      - /volume1/configs/caddy/config:/config
    networks:
      - proxy
      - internal
    environment:
      - TZ=Australia/Melbourne
      - CLOUDFLARE_API_TOKEN=${CF_API_TOKEN}
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:80"]
      interval: 30s
      timeout: 3s
      retries: 3
```

## Trust

This repository was really created for my own convienience when managing internal services for my Homelab. There is nothing additional added to the image: It is literally just the official Caddy image + the official Caddy Cloudflare DNS provider add-on bundled together.

You can examine everything to see what the pre-build container actually contains, however generally **you should not trust a container published by a random person on the internet**.

If you would like to build this image yourself, you can do so using the provided Dockerfile, or fork this repository and modify it for your own needs.

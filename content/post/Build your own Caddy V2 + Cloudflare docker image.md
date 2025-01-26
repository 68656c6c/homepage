---
share: true
title: Build your own Caddy V2 + Cloudflare docker image
date: 2025-01-15
tags:
  - technology/homelab/docker
  - technology/homelab/caddy
---
# Creating own caddy image that includes cloudflare plugin with a Dockerfile

Create a file, in this case we are creating `Caddy.Dockerfile` with the following:

## Caddy.Dockerfile
```dockerfile
FROM caddy:builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare
FROM caddy:latest
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

And create a `docker-compose.yml` with the following:

## docker-compose.yml
```yaml
services:
  caddy:
    build:
      context: .
      dockerfile: Caddy.Dockerfile
    image: caddy-cloudflare:latest
    restart: unless-stopped
    environment:
      - CF_API_TOKEN=
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./caddydata:/data
      - ./caddyconfig:/config
    networks:
      - caddy_network

  your-app-here:
    networks:
      - caddy_network

networks:
  caddy_network:
    driver: bridge
```

Make sure to add your own app in the `your-app-here` section in the **docker-compose.yml** file.

To build the image, do the following: `docker-compose up -d --build`

## Caddyfile
```yaml
name.example.tld {
  tls {
    dns cloudflare {env.CF_API_TOKEN}
    resolvers 1.1.1.1
  }
  reverse_proxy service-name:1111 {
    header_up X-Real-IP {remote_host}
  }
}
```

Replace `name.example.tld` and `service-name:1111` with what you're trying to host.
I have added `resolvers 1.1.1.1` because of local DNS, the `example.tld` is being used internally and would not resolve the DNS challenge.

If you want to update your new docker container, you just run the command `docker-compose up -d --build` again and it will build it for you.

{{< box important >}}
**This is possibly outdated**

If you want to improve this and host it on GitHub, a better way is possible with using GitHub runner to build this image and then host the image on your GitHub repo
{{< /box >}}
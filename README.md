# caddy-docker-proxy-azcf

This is [caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy), but built to include support for [Azure](https://github.com/caddy-dns/azure) and [Cloudflare](https://github.com/caddy-dns/cloudflare) DNS for ACME.

## Labels for DNS Configuration

These labels add the global configuration for Azure and Cloudflare DNS.
Their aliases are then used in the actual backend containers you want to proxy.

```yaml
caddy_0: (azdns)
caddy_0.tls.dns: azure
# replace the hyphenated values, start
caddy_0.tls.dns.subscription_id: azure-subscription-id
caddy_0.tls.dns.resource_group_name: azure-resource-group-name
caddy_0.tls.dns.tenant_id: azure-tenant-id
caddy_0.tls.dns.client_id: azure-client-id
caddy_0.tls.dns.client_secret: azure-client-secret
# end
caddy_1: (cfdns)
caddy_1.tls.dns: cloudflare
# replace the hyphenated value, start
caddy_1.tls.dns.api_token: cloudflare-API-key
# end
```

### Sample Complete Docker Compose

(includes a configuration for proxy protocol)

```yaml
services:
  caddy:
    image: ghcr.io/timcappalli/caddy-docker-proxy-azcf:latest
    ports:
      - 80:80
      - 443:443/tcp
      - 443:443/udp
    environment:
      - CADDY_INGRESS_NETWORKS=caddy
    networks:
      - caddy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - caddy_data:/data
    restart: unless-stopped
    labels:
      caddy.servers.listener_wrappers.proxy_protocol.allow: "replace with CIDR network"
      caddy.servers.listener_wrappers.proxy_protocol.fallback_policy: ignore
      caddy.servers.listener_wrappers.proxy_protocol.timeout: 2s
      caddy.servers.listener_wrappers.tls:
      caddy_0: (azdns)
      caddy_0.tls.dns: azure
      caddy_0.tls.dns.subscription_id: "replace"
      caddy_0.tls.dns.resource_group_name: "replace"
      caddy_0.tls.dns.tenant_id: "replace"
      caddy_0.tls.dns.client_id: "replace"
      caddy_0.tls.dns.client_secret: "replace"
      caddy_1: (cfdns)
      caddy_1.tls.dns: cloudflare
      caddy_1.tls.dns.api_token: "replace"
      
networks:
  caddy:
    external: true

volumes:
  caddy_data: {}
```

## Backend Container Configuration

For the backend containers you want to proxy through Caddy, use the regular options, plus import the DNS provider for the domain:
`caddy.import: azdns | cfdns`

Full example:

```bash
caddy: testing.cloudauth.dev
caddy.import: cfdns
caddy.reverse_proxy: {{ upstreams 3000 }}
```

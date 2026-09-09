# DNS

## Provider roles

The domain is registered at **GoDaddy**. The authoritative nameservers for `dague.vip` are hosted by **Cloudflare**.

Changing the nameserver delegation told DNS resolvers to use Cloudflare as the authoritative source for the zone. It did not transfer the domain away from GoDaddy.

Cloudflare is used as **authoritative DNS only** for the website. HTTP proxying is disabled.

## Website records

The important relationships are:

```text
dague.vip   -> CloudFront distribution
*.dague.vip -> dague.vip
```

The website records are configured as **DNS only**.

## Why there is no explicit www record

`www.dague.vip` is currently matched by `*.dague.vip`.

CloudFront also accepts `*.dague.vip` as an alternate domain name, and the certificate covers `*.dague.vip`. A separate `www` record would only be needed if `www` should behave differently or the wildcard design is removed.

## DNS-only versus proxied

Current design:

```text
Client -> Cloudflare DNS -> Amazon CloudFront
```

Enabling Cloudflare proxying would put Cloudflare's reverse proxy into the HTTP/TLS path before CloudFront. That was unnecessary for this project and caused problems during setup, so the AWS-facing records were changed to DNS-only.

## DNSSEC

DNSSEC-related protection was enabled during the Cloudflare setup.

DNSSEC protects the authenticity of signed DNS responses. It is separate from HTTPS/TLS, which protects the web connection.

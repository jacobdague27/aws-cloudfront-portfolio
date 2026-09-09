# Troubleshooting

The main troubleshooting rule for this project was:

> Test the shortest path possible, prove that layer works, and then add the next layer.

## Default root object left blank

The bucket contained `index.html`, but requesting the CloudFront root URL did not load it.

The distribution needed:

```text
Default root object: index.html
```

## Leading slash caused AccessDenied

The root object was then mistakenly configured as:

```text
/index.html
```

The root URL still returned `AccessDenied`.

However:

```text
https://<distribution>.cloudfront.net/index.html
```

worked.

The same pattern occurred on the custom domain:

```text
https://dague.vip/index.html -> worked
https://dague.vip/           -> failed
```

That proved the object existed and CloudFront could retrieve it through OAC. It narrowed the failure to root-object handling instead of S3 permissions or DNS.

The fix was:

```text
index.html
```

with **no leading slash**.

## Missing custom certificate

The custom domain was not initially configured with the required certificate.

An ACM certificate was created and validated for:

```text
dague.vip
*.dague.vip
```

The CloudFront certificate was created in `us-east-1` and attached to the distribution.

## Cloudflare proxying

Cloudflare's proxy was initially enabled during DNS testing.

That added another HTTP/TLS layer in front of CloudFront and did not match the intended architecture. The website records were changed to **DNS only**.

## CNAME visibility while proxied

While Cloudflare proxying was enabled, `nslookup` returned Cloudflare edge IPs instead of exposing the configured AWS CNAME target.

That was expected behavior for a proxied Cloudflare hostname. Changing the record to DNS-only made public DNS behavior match the intended design.

## Troubleshooting sequence

1. Confirm `index.html` exists in S3.
2. Test `https://<distribution>.cloudfront.net/index.html`.
3. Test `https://<distribution>.cloudfront.net/`.
4. Test `https://dague.vip/`.
5. If the site shows old content, check CloudFront caching and invalidate the changed path if needed.

On Windows:

```powershell
nslookup dague.vip
nslookup -type=CNAME dague.vip
```

DNS resolution only tests name resolution. It does not prove CloudFront can retrieve the S3 object or handle the HTTP request correctly.

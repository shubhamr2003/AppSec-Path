## HTTP Headers Used for URL or Request Overrides

These headers are non-standard, implementation-specific, and often used by proxies, load balancers, or frameworks. When trusted without validation, they can lead to routing, access-control, or security issues.

### 1. URL-Path Override Headers

These can change the effective path or URL that the backend processes.

|Header|Typical purpose|
|---|---|
|`X-Original-URL`|Overrides the original request URL.|
|`X-Rewrite-URL`|Overrides the URL used for routing.|
|`X-Url`|Sometimes used to specify the target URL.|
|`X-Base-URL`|May override the base path in some frameworks.|
|`X-Forwarded-Prefix`|Indicates a URL prefix added by a proxy.|

PortSwigger and OWASP describe `X-Original-URL` and `X-Rewrite-URL` as headers that can override parts of the request URL and affect routing.

### 2. Host-Override Headers

These can change the hostname that the backend believes was requested.

|Header|Typical purpose|
|---|---|
|`X-Forwarded-Host`|De-facto standard to indicate the original host requested by the client.|
|`X-Host`|Another host-override variant.|
|`X-Forwarded-Server`|Similar to `X-Forwarded-Host`.|
|`X-HTTP-Host-Override`|Host override used by some systems.|

These headers are often used by proxies and load balancers to preserve the original host. If an application trusts them unsafely, it can lead to host-header injection, cache poisoning, or routing problems.[[fastly](https://www.fastly.com/learning/security/what-are-http-host-header-attacks)]

### 3. Protocol and Port Override Headers

These indicate the original protocol and port used by the client.

|Header|Typical purpose|
|---|---|
|`X-Forwarded-Proto`|Indicates whether the client used HTTP or HTTPS. [[docs.aws.amazon](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/x-forwarded-headers.html)]|
|`X-Forwarded-Ssl`|Sometimes used to indicate SSL status.|
|`X-Forwarded-Port`|Indicates the original client port.|
|`X-Forwarded-Scheme`|Similar to `X-Forwarded-Proto`.|

These are useful for correct URL generation and redirects, but can be misused if the application uses them to make security decisions without validation.

### 4. Client-IP Override Headers

These indicate the original client IP address.

|Header|Typical purpose|
|---|---|
|`X-Forwarded-For`|De-facto standard to identify the originating IP address of a client connecting through a proxy. [[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-For)]|
|`X-Real-IP`|Another common IP-forwarding header.|
|`X-Client-IP`|Similar purpose.|
|`X-Originating-IP`|Sometimes used for IP logging.|
|`X-Remote-IP`|Another variant.|

Attackers may spoof these to bypass IP-based access controls, rate limits, or geo-restrictions.

### 5. Method-Override Headers

These allow a client to change the HTTP method seen by the application.

|Header|Typical purpose|
|---|---|
|`X-Method-Override`|Overrides the HTTP method, often to send `PUT` or `DELETE` as `POST`. [[ibm](https://www.ibm.com/docs/en/odm/9.5.0?topic=configurations-configuring-http-methods)]|
|`X-HTTP-Method-Override`|Same purpose, different name.|
|`X-Override-Method`|Another variant.|

For example, a client may send:

```
POST /resource HTTP/1.1
Host: example.com
X-HTTP-Method-Override: DELETE
```

Some frameworks, such as Flask, support `X-HTTP-Method-Override` to allow method overrides in environments that restrict certain methods.[[flask.palletsprojects](https://flask.palletsprojects.com/en/stable/patterns/methodoverrides/)]

### 6. Other Related Headers

|Header|Typical purpose|
|---|---|
|`Forwarded`|Standardized header defined in RFC 7239 to disclose proxying information. [[datatracker.ietf](https://datatracker.ietf.org/doc/html/rfc7239)]|
|`X-Request-Start`|Sometimes used to indicate request timing.|
|`X-Request-ID`|Used for tracing and logging.|
|`X-Content-Base-URL`|Occasionally used to override content base paths.|

## How These Headers Are Used

Legitimate uses include:

- Reverse proxies preserving original request information.
- Load balancers forwarding client details.
- URL-rewrite rules in complex architectures.
- Supporting clients that cannot send certain methods or protocols directly.

Security problems arise when:

- The application trusts these headers without validation.
- Access control depends on the visible URL or host, but the backend uses the header value.
- Logs, caches, or security decisions rely on untrusted header values.

PortSwigger notes that applications may support non-standard headers to override parts of the request URL, potentially affecting routing and processing.

## Example Combinations

### Path override

```
GET /public HTTP/1.1
Host: example.com
X-Original-URL: /admin
```

### Host override

```
GET / HTTP/1.1
Host: example.com
X-Forwarded-Host: attacker.com
```

### Method override

```
POST /resource HTTP/1.1
Host: example.com
X-HTTP-Method-Override: DELETE
```

### Protocol override

```
GET / HTTP/1.1
Host: example.com
X-Forwarded-Proto: https
```

### IP override

```
GET / HTTP/1.1
Host: example.com
X-Forwarded-For: 127.0.0.1
```

## Security Considerations

- Do not trust these headers from untrusted clients.
- Strip or validate them at the edge if not required.
- Apply authorization and access control to the final, effective URL and host.
- Avoid making security decisions based solely on these headers.
- Use the standardized `Forwarded` header where possible instead of multiple custom headers.

OWASP and PortSwigger recommend testing these headers when evaluating authorization schemas and host-header security.[[fastly](https://www.fastly.com/learning/security/what-are-http-host-header-attacks)]
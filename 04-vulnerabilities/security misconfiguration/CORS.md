resources:
https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS

youtube:
[What is CORS](https://youtu.be/WWnR4xptSRk)

## What Is CORS?

**CORS** stands for **Cross-Origin Resource Sharing**. It is a browser security mechanism that allows a website to request resources from a different **origin** when the destination server explicitly permits it.

It is a **browser feature**.

**CORS itself is not a vulnerability.** CORS (Cross-Origin Resource Sharing) is a **browser security mechanism** that controls which origins are allowed to access resources from another origin.

The vulnerability occurs when **CORS is misconfigured**.

For example:
- Frontend: `https://shop.example`
- API: `https://api.example`

These are different origins, but CORS can allow the frontend to communicate with the API.
## What Is an Origin?

ORIGIN = scheme://host:port

```
scheme://host:port
```

Example:

```
https://example.com:443
```

Two URLs have different origins if their scheme, host, or port differs.

| URL                           | Compared with `https://example.com`           |
| ----------------------------- | --------------------------------------------- |
| `https://example.com/profile` | Same origin, different path (path is allowed) |
| `http://example.com`          | Different scheme                              |
| `https://api.example.com`     | Different host                                |
| `https://example.com:8443`    | Different port                                |

Even `https://example.com` and `https://api.example.com` are different origins because their hostnames differ.

## Why Does CORS Exist?

Browsers use a security rule called the **Same-Origin Policy**, or SOP. It prevents JavaScript running on one website from freely reading sensitive data from another website. CORS provides a controlled exception to that rule.[[owasp](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing)][[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy)]

Imagine you are logged in to your bank:

```
https://bank.example
```

If any website could silently read your bank account data through JavaScript, that would be dangerous. The browser therefore blocks cross-origin reading unless the bank’s server says that the requesting origin is allowed.

CORS does not completely remove the Same-Origin Policy. It allows the server to specify which other origins may access its responses.[[portswigger](https://portswigger.net/web-security/cors)]

## Simple CORS Example

Suppose JavaScript on this website:

```
https://frontend.example
```

requests data from:

```
https://api.example
```

The browser sends an `Origin` header:

```
GET /users HTTP/1.1
Host: api.example
Origin: https://frontend.example
```

If the API trusts that frontend, it responds:

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://frontend.example
Content-Type: application/json
```

The browser sees that the API allows `https://frontend.example`, so it makes the response available to the frontend’s JavaScript.

If the server does not allow that origin, the browser blocks the JavaScript from reading the response.

## Important CORS Headers

|Header|Purpose|
|---|---|
|`Origin`|Identifies the website making the cross-origin request.|
|`Access-Control-Allow-Origin`|Specifies which origin may read the response.|
|`Access-Control-Allow-Methods`|Lists permitted methods such as `GET`, `POST`, or `DELETE`.|
|`Access-Control-Allow-Headers`|Lists non-simple request headers the client may send.|
|`Access-Control-Allow-Credentials`|Allows credentials such as cookies to be included.|
|`Access-Control-Max-Age`|Tells the browser how long it may cache the preflight result.|

Example:

```
Access-Control-Allow-Origin: https://frontend.example
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Authorization, Content-Type
```

This means:

- Only `https://frontend.example` is trusted.
- `GET` and `POST` requests are allowed.
- The frontend may send `Authorization` and `Content-Type` headers.

## What Is a Preflight Request?

For some cross-origin requests, the browser first sends an `OPTIONS` request to ask the server whether the real request is allowed. This is called a **preflight request**.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)][[owasp](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing)]

For example:

```
OPTIONS /users HTTP/1.1
Host: api.example
Origin: https://frontend.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type
```

The server may respond:

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://frontend.example
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: Authorization, Content-Type
```

The browser then sends the actual `POST` request.

A preflight commonly happens when:

- The method is not a simple `GET`, `POST`, or `HEAD`.
- The request uses methods such as `PUT`, `PATCH`, or `DELETE`.
- The request includes custom headers.
- The request uses certain content types, such as JSON.

## CORS and Cookies

By default, browsers do not include cross-origin cookies in many requests. A frontend must explicitly request credentialed access:

```
fetch("https://api.example/profile", {
  credentials: "include"
});
```

The server must then respond with:

```
Access-Control-Allow-Origin: https://frontend.example
Access-Control-Allow-Credentials: true
```

A server should not use this insecure combination:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

For credentialed requests, the server should specify an exact trusted origin rather than allowing every origin. OWASP recommends allowing only selected, trusted domains and avoiding sensitive cross-origin exposure.[[cheatsheetseries.owasp](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)][[cheatsheetseries.owasp](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)]

## CORS Security Misconfiguration

Common mistakes include:

```
Access-Control-Allow-Origin: *
```

This allows any origin to read responses that are not protected by credentials. It may be acceptable for genuinely public data, but it is dangerous for sensitive information.

Other problems include:

- Reflecting any value from the `Origin` header without validation.
- Trusting attacker-controlled subdomains.
- Allowing credentials for untrusted origins.
- Allowing unnecessary methods such as `DELETE`.
- Allowing sensitive headers unnecessarily.
- Applying permissive CORS rules to the entire application instead of selected endpoints.

As a pentester, test whether an untrusted origin can read sensitive responses, especially responses containing account information, personal data, or API keys. CORS testing should be performed only against systems you own or are explicitly authorized to test.

## Common Misunderstandings

### CORS is not authentication

CORS does not log users in and does not replace access control. The server must still authenticate the user and authorize every requested resource.

### CORS is mainly a browser restriction

CORS controls whether browser-based JavaScript can read a response. Tools such as `curl`, Burp Suite, or server-side programs are not stopped by the browser’s CORS enforcement. The server must still implement proper authentication and authorization.

### CORS does not automatically prevent CSRF

CORS and CSRF are related to browser requests but are different security topics. CORS controls whether JavaScript can read a response; CSRF concerns unwanted state-changing requests made using a victim’s credentials.

## Simple Summary

> **CORS is a permission system that lets a server decide which other websites may read its responses through browser JavaScript.**

Remember the basic flow:

```
Website A sends request
        ↓
Server checks the Origin
        ↓
Server returns CORS headers
        ↓
Browser allows or blocks JavaScript from reading the response
```

The most important header to remember is:

```
Access-Control-Allow-Origin
```

It tells the browser which origin is allowed to access the response.

## Why can CORS become a security misconfiguration?

CORS itself is not a vulnerability. It becomes a problem when the server gives permission to **untrusted origins**, especially for sensitive or authenticated data.

A misconfigured CORS policy may allow an attacker-controlled website to make a browser request to a target application and read the response. If the victim is logged in, the response could contain private information. PortSwigger describes this risk as allowing a trusted or attacker-controlled domain to interact with the application in the logged-in user’s security context.
## What Are HTTP Headers?

**HTTP headers** are small pieces of additional information sent along with an HTTP request or response. They help the browser and server understand **how to process the message**, what type of data is being sent, and what rules should be followed.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)][[developer.mozilla](https://developer.mozilla.org/en-US/docs/Glossary/HTTP_header)]

Think of an HTTP request or response as a parcel:

- **Headers** = the label and instructions on the parcel.
- **Body** = the actual content inside the parcel.

A header usually looks like this:

```
Header-Name: value
```

Example:

```
Content-Type: application/json
```

This means: **“The data in the body is JSON.”**

## Where Headers Appear

### Request headers

Request headers are sent by the **client**, usually your browser or an application, to the server. They provide information about the request, the client, and the type of response the client can accept.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Glossary/Request_header)]

Example:

```
GET /products HTTP/1.1
Host: example.com
User-Agent: Chrome
Accept: application/json
Authorization: Bearer token123
```

Common request headers:

|Header|Simple meaning|
|---|---|
|`Host`|The website or server being contacted.|
|`User-Agent`|Information about the browser or application making the request.|
|`Accept`|The response formats the client can understand, such as HTML or JSON.|
|`Accept-Language`|The preferred language, such as English or Hindi.|
|`Accept-Encoding`|Compression formats the client supports, such as gzip or br.|
|`Authorization`|Credentials or a token used to access protected resources.|
|`Cookie`|Data used for sessions, preferences, or login state.|
|`Referer`|The page from which the request originated.|
|`Content-Type`|The format of the data being sent in the request body.|
|`Content-Length`|The size of the request body.|

For example, when you submit a login form, the browser may send a `POST` request with `Content-Type` and `Authorization`-related information.

### Response headers

Response headers are sent by the **server** back to the browser or client. They describe the response and tell the client how to handle it.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Messages)]

Example:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 86
Cache-Control: no-cache
Set-Cookie: session_id=abc123
```

Common response headers:

|Header|Simple meaning|
|---|---|
|`Content-Type`|The format of the returned data, such as HTML, JSON, or an image.|
|`Content-Length`|The size of the response body.|
|`Set-Cookie`|Tells the browser to save a cookie.|
|`Location`|Provides a new URL during a redirect.|
|`Cache-Control`|Controls whether and how long the response can be cached.|
|`Server`|Identifies information about the server software; often hidden for security.|
|`ETag`|An identifier used to check whether a resource has changed.|
|`Last-Modified`|Shows when the resource was last changed.|
|`Content-Encoding`|Shows whether the response is compressed.|
|`Strict-Transport-Security`|Tells the browser to use HTTPS.|

## Main Types of HTTP Headers

For beginners, headers are commonly explained using four categories. Modern HTTP documentation often classifies headers by their purpose and whether they appear in requests, responses, or both.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)][[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Accept)]

### 1. General headers

General headers provide information about the HTTP message itself. They are not specifically about the client, server, or content.

Examples:

```
Date: Wed, 26 Aug 2026 06:30:00 GMT
Cache-Control: no-cache
```

Common examples include:

- `Date` — when the message was created.
- `Cache-Control` — caching instructions.
- `Connection` — connection behavior in older HTTP versions.

### 2. Request headers

Request headers provide details from the client to the server.

Examples:

```
User-Agent: Mozilla/5.0
Accept: text/html
Accept-Language: en-IN
```

They answer questions such as:

- Who is sending the request?
- What type of response is acceptable?
- What language is preferred?
- Is the user already logged in?
- What data format is being sent?

### 3. Response headers

Response headers provide details from the server to the client.

Examples:

```
Content-Type: text/html
Set-Cookie: session_id=abc123
Location: /login
```

They answer questions such as:

- What data did the server return?
- Should the browser save a cookie?
- Should the browser go to another URL?
- Can the response be cached?
- What security rules should the browser follow?

### 4. Representation or content headers

These headers describe the **data in the message body**, including its format, language, encoding, and size. Older learning material may call these **entity headers**.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)][[github](https://github.com/orgs/mdn/discussions/807)]

Examples:

```
Content-Type: application/json
Content-Length: 250
Content-Encoding: gzip
Content-Language: en
```

Important representation headers:

|Header|Example|Purpose|
|---|---|---|
|`Content-Type`|`application/json`|Describes the body format.|
|`Content-Length`|`2500`|Gives the body size in bytes.|
|`Content-Encoding`|`gzip`|Shows how the body is compressed.|
|`Content-Language`|`en`|Indicates the language of the content.|

## Complete Example

### Browser sends a request

```
GET /profile HTTP/1.1
Host: example.com
User-Agent: Chrome
Accept: text/html
Cookie: session_id=abc123
```

Meaning:

- `GET /profile` — request the profile page.
- `Host` — contact `example.com`.
- `User-Agent` — the request came from Chrome.
- `Accept` — the browser can understand HTML.
- `Cookie` — the browser has a session identifier.

### Server sends a response

```
HTTP/1.1 200 OK
Content-Type: text/html
Cache-Control: no-cache
Set-Cookie: session_id=xyz789
```

Meaning:

- `200 OK` — the request succeeded.
- `Content-Type: text/html` — the response contains HTML.
- `Cache-Control: no-cache` — do not use an old cached copy.
- `Set-Cookie` — save a new session cookie.

The actual webpage would appear after a blank line:

```
<html>
  <body>
    <h1>My Profile</h1>
  </body>
</html>
```

## Headers Important for Web Security

Since you are learning web-application penetration testing, pay special attention to these:

|Header|Security purpose|
|---|---|
|`Authorization`|Carries authentication credentials or tokens.|
|`Cookie`|Carries session information.|
|`Set-Cookie`|May contain security flags such as `Secure`, `HttpOnly`, and `SameSite`.|
|`Content-Security-Policy`|Restricts scripts and other resources to reduce XSS risk.|
|`Strict-Transport-Security`|Forces the browser to use HTTPS.|
|`X-Content-Type-Options: nosniff`|Helps prevent MIME-sniffing attacks.|
|`X-Frame-Options`|Helps prevent clickjacking.|
|`Access-Control-Allow-Origin`|Controls which origins can access a resource through CORS.|
|`Referer`|May reveal the page from which a request came.|
|`Origin`|Identifies the origin that made a cross-origin request.|

## Simple Summary

|Type|Sent by|Main purpose|
|---|---|---|
|Request headers|Browser/client|Describe the request and client.|
|Response headers|Server|Describe the response and server instructions.|
|General headers|Client or server|Give information about the HTTP message.|
|Representation headers|Client or server|Describe the body’s format, size, language, or encoding.|

The easiest way to remember them is:

> **Request headers tell the server what the client wants. Response headers tell the client what the server returned and how to handle it.**

## Main purpose of HTTP headers

The main purpose of HTTP headers is to carry **additional information and instructions** about an HTTP request or response without putting that information inside the actual content.

For example, headers can tell the server or browser:

- What resource is being requested.
- What type of data is being sent.
- What response formats the client accepts.
- Whether the user is logged in.
- How long a response can be cached.
- Whether the connection should use HTTPS.
- How large or compressed the message body is.

In simple words:

> **HTTP headers are metadata and instructions that help the client and server correctly process the message.**[[rfc-editor](https://www.rfc-editor.org/rfc/rfc9110.html)][[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP)]

## Headers versus the body

Consider this response:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 42
Cache-Control: max-age=3600
```

```
<h1>Welcome to my website</h1>
```

The HTML is the **body**. The headers explain how to handle it:

- `200 OK` — the request succeeded.
- `Content-Type: text/html` — the body contains HTML.
- `Content-Length: 42` — the body has a particular size.
- `Cache-Control` — the browser may cache the response for a specified period.

Without headers, the browser might receive the content but not know exactly how to interpret or manage it.

## Why were HTTP headers introduced?

The earliest HTTP version, **HTTP/0.9**, was extremely simple. A client requested a resource, and the server returned raw data, usually HTML. It did not provide a well-developed way to describe the data or communicate additional instructions.

As the web became more useful, this simple approach was not enough. HTTP needed to support:

- Different types of content, such as HTML, images, audio, and video.
- Information about the size and encoding of data.
- Caching to avoid downloading unchanged files repeatedly.
- Authentication and cookies.
- Proxies and gateways.
- Multiple websites hosted on one server.
- Client preferences, such as preferred language.
- Rules for requests and responses.

HTTP/1.0 therefore introduced MIME-like messages containing **metadata about transferred data** and modifiers that affected request and response behavior.[[datatracker.ietf](https://datatracker.ietf.org/doc/html/rfc2068)]

So, headers were introduced to make HTTP more **flexible, descriptive, and extensible**.

## A simple analogy

Imagine sending a parcel.

### Without headers

You send only the contents:

```
Some HTML text
```

The receiver may not know:

- What the content is.
- How large it is.
- Which language it uses.
- Whether it is compressed.
- Whether it should be stored for later.
- Where it came from.

### With headers

You send the contents with a label:

```
Type: HTML
Language: English
Size: 42 bytes
Cache: Store for one hour
```

The parcel’s contents are still the same, but the label helps the receiver process them correctly.

## Examples of problems headers solve

### 1. Identifying the content type

```
Content-Type: application/json
```

This tells the client that the body contains JSON rather than HTML.

### 2. Content negotiation

The client can say:

```
Accept: application/json
```

This means:

> “If possible, send the response as JSON.”

The `Accept` header allows the client and server to negotiate a suitable representation of the resource.[[w3](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)][[httpwg](https://httpwg.org/specs/rfc7231.html)]

### 3. Authentication

```
Authorization: Bearer token123
```

This allows a client to provide credentials or an access token when requesting a protected resource.

### 4. Session management

The server can send:

```
Set-Cookie: session_id=abc123
```

The browser later sends:

```
Cookie: session_id=abc123
```

This allows the server to recognize the user’s session.

### 5. Caching

```
Cache-Control: max-age=3600
```

This tells the browser or an intermediate cache how long it may reuse the response.

### 6. Compression

The client can indicate:

```
Accept-Encoding: gzip, br
```

The server may respond with:

```
Content-Encoding: gzip
```

This reduces the amount of data transferred.

### 7. Virtual hosting

The `Host` header tells the server which website the client wants:

```
Host: example.com
```

This is important when one server hosts many domain names.

## Important distinction

HTTP headers do **not** usually contain the main page or API data. Instead:

- **Headers** describe the message and provide instructions.
- **The body** contains the actual resource or submitted data.
- **The status line** tells the general result, such as `200 OK` or `404 Not Found`.

HTTP specifications describe headers as a major extension point, meaning new headers can be added over time without redesigning the entire protocol.[[rfc-editor](https://www.rfc-editor.org/rfc/rfc9110.html)]

## Final answer

HTTP headers were introduced because the original simple HTTP communication needed a way to carry more information than just a URL and raw content.

Their main purpose is to help clients, servers, proxies, and caches understand:

> **what is being requested, what is being returned, how it should be processed, and what rules apply to the communication.**

Without headers, modern features such as content types, cookies, authentication, compression, caching, HTTPS enforcement, and API communication would be difficult or impossible to implement cleanly.
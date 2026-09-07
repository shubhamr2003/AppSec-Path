python -m SimpleHTTPServer 8080: to import or share a file

php -S 127.0.0.1:808: to host a server on localhost

- favicon.ico is the path of tiny icons that load when a website is loaded

_source:_ https://developer.mozilla.org/en-US/docs/Web/HTTP (overall http)
https://restfulapi.net/http-methods/

==learn curl command==

1. _**HTTP Methods**_
	
| Method      | Action         | Purpose                                                                          | Safe? | CRUD Equivalent |
| ----------- | -------------- | -------------------------------------------------------------------------------- | ----- | --------------- |
| **GET**     | Read/View      | Retrieve data from the server (no changes made)                                  | ✅ Yes | **R**ead        |
| **POST**    | Create New     | Send data to create a new resource on the server                                 | ❌ No  | **C**reate      |
| **PUT**     | Full Update    | Replace an existing resource completely                                          | ❌ No  | **U**pdate      |
| **PATCH**   | Partial Update | Update only specific parts of a resource (not the whole thing)                   | ❌ No  | **U**pdate      |
| **DELETE**  | Delete         | Update only specific parts of a resource (not the whole thing)                   | ❌ No  | **D**elete      |
| **HEAD**    | Get Headers    | Same as GET, but only returns the **headers** (metadata), not the actual content | ✅ Yes | Read (metadata) |
| **OPTIONS** | Check Methods  | Find out what HTTP methods are supported for a resource                          | ✅ Yes | Info            |
| CONNECT     |                |                                                                                  |       |                 |
| TRACE       |                |                                                                                  |       |                 |
	   
2. _**HTTP Headers**_
	
	_sources:_ 
	- [Simple and in detail](https://blog.postman.com/what-are-http-headers/ )
	- [HTTP Overview (yt)](https://youtu.be/8q5mc1AEtYo?si=nesuh5ygg6pG4XbF )
	- [Cache Header]( https://stackoverflow.com/questions/10314174/difference-between-pragma-and-cache-control-headers)
	- [Security Headers (yt)](https://youtu.be/064yDG7Rz80?si=J79zrKAFG0k1jm57 )
	
	- HTTP headers are metadata fields that accompany HTTP requests and responses, providing additional context about the communication between clients and servers. They contain key-value pairs that define content types, authentication credentials, caching behavior, and security policies.
	
	- You can think of an HTTP header as a shipping label on a package. The package contains the actual data (such as a product or document), but the label provides the delivery system with important details, including where it originated, its destination, handling instructions, and its contents.
	  
	_**Types of HTTP Headers:**_
	
	1. Request
		 Some common HTTP Request Headers are:
		  - Accept
		  - User Agent 
		  - Authorization
		  - Content Type
		  - Cookie
		  - Accept Lang
		  - Accept Encoding
		  - Refer
		  - If Modified since
		   	
	2. Response ...
		Some common HTTP Request Headers are
		 - Content Type
		 - Cache Control
		 - Server
		 - Set Cookie
		 - Cookie Length
		 - Location
		 - E Tag
		 - Access-Control-Allow-Origin
		 - Strict-Transport-Security
		 
	3. Representation: Describe the encoding, format, and other characteristics of the message body in both requests and responses. ...
		   a
		   
	4. Payload: Contain information about the payload data, including content length, encoding, and range information for partial content delivery. ...
		a
		
	5. Security Headers ...
		a   
	   
3. _**HTTP Status Code**_
	   
	_sources:_ 
	- [fun way to remember status codes]( https://http.cat/)
	- https://en.wikipedia.org/wiki/List_of_HTTP_status_codes
	- [visual representation of status codes](https://www.quattr.com/enhance-experience/http-status-codes-explained)
	
| Class             | Range | Meaning                                        | Example                                    |
| ----------------- | ----- | ---------------------------------------------- | ------------------------------------------ |
| **Informational** | 1xx   | Request received, process continuing           | 100 Continue                               |
| **Success**       | 2xx   | Request succeeded, accepted                    | 200 OK, 201 Created                        |
| **Redirection**   | 3xx   | Further action needed to complete              | 301 Moved Permanently, 304 Not Modified    |
| **Client Error**  | 4xx   | Request has syntax error or can't be fulfilled | 400 Bad Request, 404 Not Found             |
| **Server Error**  | 5xx   | Server failed to fulfill valid request         | 500 Internal Server Error, 502 Bad Gateway |

| Code    | Name                | What It Means                            | When Used                                            |
| ------- | ------------------- | ---------------------------------------- | ---------------------------------------------------- |
| **100** | Continue            | Server received headers, send the body   | Client should continue sending request body          |
| **101** | Switching Protocols | Server agrees to switch protocols        | Used with `Upgrade` header (e.g., WebSocket)         |
| **103** | Early Hints         | Server sends hints before final response | Browser can preload resources while server prepares  |

| Code    | Name                          | What It Means                                          | Real Example                           |
| ------- | ----------------------------- | ------------------------------------------------------ | -------------------------------------- |
| **200** | OK                            | **Request succeeded** (most common)                    | Loading a normal webpage               |
| **201** | Created                       | New resource was **successfully created**              | After POST creates a new user          |
| **202** | Accepted                      | Request accepted, but **processing not complete**      | Long task (e.g., video upload) started |
| **203** | Non-Authoritative Information | Response succeeded, but info from another source       | Proxy server modified the response     |
| **204** | No Content                    | Request succeeded, but **no content to show**          | DELETE request (nothing to return)     |
| **205** | Reset Content                 | Success, but **reset the document**                    | Clear a form after submission          |
| **206** | Partial Content               | Server sends **only part** of the resource             | Downloading large file in chunks       |
| **207** | Multi-Status                  | Multiple responses in one (for WebDAV)                 | Bulk operations on files               |
| **208** | Already Reported              | Same member already reported earlier                   | Used with WebDAV for loops             |
| **226** | IM Used                       | Server fulfilled an IM (Instance Manipulation) request | Advanced HTTP extensions               |

| Code    | Name                      | What It Means                                  | When Used                                    |
| ------- | ------------------------- | ---------------------------------------------- | -------------------------------------------- |
| **300** | Multiple Choices          | Resource has **multiple versions**, choose one | Server offers different formats (HTML, JSON) |
| **301** | Moved Permanently         | Resource **permanently moved** to new URL      | Old URL → New URL (update bookmarks)         |
| **302** | Found (Moved Temporarily) | Resource **temporarily moved** to new URL      | Short-term redirect (e.g., maintenance)      |
| **303** | See Other                 | Go to a **different URL** to find the result   | After POST, redirect to success page         |
| **304** | Not Modified              | Resource **not changed** since last time       | Browser uses **cached copy** instead         |
| **305** | Use Proxy                 | Must access via **proxy server**               | Rare, security requirement                   |
| **306** | Use Proxy (unused)        | Reserved                                       | Not used anymore                             |
| **307** | Temporary Redirect        | **Temporarily** at new URL (keep method)       | Like 302, but keeps POST/GET same            |
| **308** | Permanent Redirect        | **Permanently** at new URL (keep method)       | Like 301, but keeps POST/GET same            |
| **309** | Resolution Failure        | DNS resolution failed                          | Rare, network issue                          |

| Code    | Name                                 | What It Means                                  | Real Example                                |
| ------- | ------------------------------------ | ---------------------------------------------- | ------------------------------------------- |
| **400** | Bad Request                          | Request has **bad syntax** (can't understand)  | Malformed JSON, invalid parameters          |
| **401** | Unauthorized                         | **Authentication needed** (not logged in)      | Login required to access page               |
| **402** | Payment Required                     | Reserved for future use                        | Not used yet                                |
| **403** | Forbidden                            | Server **refuses** (even if logged in)         | No permission to access file                |
| **404** | Not Found                            | **Resource doesn't exist** (most common error) | Deleted page, wrong URL                     |
| **405** | Method Not Allowed                   | HTTP method not supported                      | Using DELETE when only GET allowed          |
| **406** | Not Acceptable                       | Can't create response client accepts           | Server only has XML, client wants JSON      |
| **407** | Proxy Authentication Required        | Must authenticate with **proxy**               | Corporate proxy needs login                 |
| **408** | Request Timeout                      | Server **timed out** waiting                   | Slow network, request took too long         |
| **409** | Conflict                             | Request **conflicts** with current state       | Editing someone else's changes              |
| **410** | Gone                                 | Resource **permanently gone**                  | Deleted content, no replacement             |
| **411** | Length Required                      | Missing `Content-Length` header                | Server needs size info                      |
| **412** | Precondition Failed                  | Request header condition failed                | `If-Match` header didn't match              |
| **413** | Content Too Large                    | Request body **too big**                       | Upload file exceeds limit                   |
| **414** | URI Too Long                         | URL is **too long**                            | Too many query parameters                   |
| **415** | Unsupported Media Type               | Wrong **file format**                          | Sending PDF when server expects JPG         |
| **416** | Range Not Satisfiable                | Can't provide requested portion                | Requesting bytes 1000-2000 of 500-byte file |
| **417** | Expectation Failed                   | `Expect` header failed                         | Rare, server doesn't support requirement    |
| **418** | I'm a Teapot                         | Easter egg (not real)                          | From coffee pot joke                        |
| **419** | Authentication Timeout               | Auth timed out (PHP)                           | Login expired                               |
| **420** | Enhance Your Calm                    | Too many requests (Twitter)                    | Rate limiting [                             |
| **421** | Misdirected Request                  | Request to wrong server                        | Connection reuse issue                      |
| **422** | Unprocessable Content                | Valid but can't process                        | Invalid XML structure                       |
| **423** | Locked                               | Resource is locked                             | File being edited by someone                |
| **424** | Failed Dependency                    | Previous request failed                        | Dependency chain broken                     |
| **425** | Too Early                            | Request too early                              | TLS 1.3 early data                          |
| **426** | Upgrade Required                     | Need newer protocol                            | Client needs HTTP/2                         |
| **428** | Unrepeatable                         | Request not repeatable                         | Race condition                              |
| **429** | Too Many Requests                    | **Rate limited** (spam detected)               | Too many requests in 1 minute               |
| **431** | Request Header Fields Too Large      | Headers too big                                | Cookie too large                            |
| **444** | No Response                          | Server closes connection                       | nginx error                                 |
| **449** | Retry With                           | Retry with different method                    | Windows Update                              |
| **450** | Blocked by Windows Parental Controls | Parental controls blocking                     | Windows feature                             |
| **451** | Unavailable For Legal Reasons        | **Blocked by law**                             | Government censorship                       |
| **499** | Client Closed Request                | Client disconnected                            | Request cancelled                           |

| Code    | Name                            | What It Means                              | Real Example                  |
| ------- | ------------------------------- | ------------------------------------------ | ----------------------------- |
| **500** | Internal Server Error           | **Generic server error** (most common 5xx) | Code bug, database crash      |
| **501** | Not Implemented                 | Server doesn't support this method         | Using HTTP/2 on HTTP/1 server |
| **502** | Bad Gateway                     | Gateway/proxy got **invalid response**     | Server behind proxy crashed   |
| **503** | Service Unavailable             | Server **overloaded or down**              | Too many users, maintenance   |
| **504** | Gateway Timeout                 | Gateway **timed out** waiting              | Server behind proxy too slow  |
| **505** | HTTP Version Not Supported      | Server doesn't support your HTTP version   | Using HTTP/3 on HTTP/1 server |
| **506** | Variant Also Negotiates         | Content negotiation issue                  | Complex redirect loop         |
| **507** | Insufficient Storage            | Server can't store response                | Disk full, quota exceeded     |
| **508** | Loop Detected                   | Infinite loop detected                     | Bad redirect chain            |
| **509** | Bandwidth Limit Exceeded        | Bandwidth quota used up                    | Hosting limit hit             |
| **510** | Not Extended                    | Extension needed                           | Policy not met                |
| **511** | Network Authentication Required | Need to authenticate for network           | Public WiFi login             |
| **598** | Network Read Timeout            | Network read timed out                     | Slow connection               |
| **599** | Network Connect Timeout         | Network connect timed out                  | Can't reach server            |

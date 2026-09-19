![recon](./images/recon.png)
![method](./images/method.jpg)


```text
Target
  ↓
1. Scope & asset discovery
  ↓
2. Subdomain discovery
  ↓
3. DNS / infrastructure
  ↓
4. Live host discovery
  ↓
5. Port/service discovery
  ↓
6. Technology fingerprinting
  ↓
7. URL / endpoint discovery
  ↓
8. JavaScript analysis
  ↓
9. Parameter discovery
  ↓
10. Content / directory discovery
  ↓
11. Vulnerability-focused testing
```

## 1. First: understand passive vs active recon

This is more important than memorizing tools.

### Passive recon
```text
Certificate Transparency
WHOIS
DNS records
Search engines
Internet indexes
Public datasets
GitHub
Wayback Machine
```

### Active recon
```text
DNS queries
Port scanning
HTTP requests
Directory fuzzing
Endpoint probing
Service enumeration
```

# 2. Subdomain enumeration
```text
example.com
api.example.com
dev.example.com
staging.example.com
admin.example.com
old.example.com
```
### Tools worth learning

- Amass
- Subfinder
- crt.sh: Certificate Transparency can reveal subdomains that have appeared in TLS certificates.
- Assetfinder

# 3. DNS enumeration

DNS records:
- A
- AAAA
- CNAME
- MX
- NS
- TXT
- SOA

```
Also understand:
```text
DNS → IP
DNS → another hostname
DNS → mail server
DNS → nameserver
```

# 4. Find live hosts
Finding 5,000 subdomains doesn't mean 5,000 websites are actually alive.
You need to determine:
```text
Which hosts respond to HTTP/HTTPS?
```
### Tool: 
**httpx**
This should definitely be in your toolkit.
Learn what information `httpx` can identify:

```text
Status code
Title
Web server
Technology
Redirects
Content length
TLS information
```

# 5. Port and service discovery
You already have experience with **Nmap**, which is excellent.

# 6. Technology fingerprinting

What technology is this application using?
For example:
```text
Nginx
Apache
Express
Django
Laravel
WordPress
React
Next.js
ASP.NET
Cloudflare
AWS
```
### Tools

- httpx
- WhatWeb
- Wappalyzer

For example:
```text
WordPress
   ↓
WordPress-specific attack surface
```
or:
```text
GraphQL
   ↓
GraphQL endpoint
   ↓
API testing
```

# 7. URL discovery

This is **extremely important for your web pentesting goals**.
You want to discover URLs such as:
```text
/api/users
/api/v1/users
/admin
/login
/upload
/reset-password
/graphql
/api/orders
```

### Tools worth knowing
- Katana: Great for crawling/discovering URLs.
- GAU: Get URLs from public sources.
- Waybackurls: Historical URLs from the Wayback Machine.

You should understand the difference:
```text
Crawler
   ↓
Discovers URLs by interacting with the application

Historical URL collection
   ↓
Finds URLs that existed previously
```

# 8. JavaScript reconnaissance
This is particularly relevant to **your current XSS + web pentesting learning**.
JavaScript files can reveal:
```text
API endpoints
Hidden routes
Parameters
Internal functionality
Feature flags
Third-party services
Potential secrets
```

For example:

```text
app.js
 ↓
/api/v1/users
/api/v1/admin
/api/v1/orders
/graphql
```

### Tools
- Burp Suite
- Katana
- LinkFinder
- SecretFinder
- JSluice

# 9. Directory/content discovery

```text
/admin
/backup
/test
/dev
/config
/uploads
/.git
/robots.txt
```

### Tool: 
**ffuf**

Understand:
```text
FUZZ
 ↓
wordlist
 ↓
requests
 ↓
responses
 ↓
interesting status/size
```
Also learn how to avoid drowning in false positives.

# 10. Parameter discovery
This is extremely valuable for finding vulnerabilities.
Suppose you discover:
```text
https://example.com/product?id=123
```

You want to understand:
```text
id
user
redirect
url
file
search
page
sort
role
```

### Tools

**Arjun**: Useful for discovering HTTP parameters.
Also learn manual parameter testing through Burp.
This connects directly to:
- IDOR/BOLA
- SQL injection
- XSS
- SSRF
- Open redirects
- Access-control testing

# 11. Content discovery wordlists

You should become familiar with **SecLists**.
Particularly:
```text
Discovery/
    Web-Content/
```

You'll encounter wordlists for:
```text
Directories
Files
Parameters
Subdomains
Common endpoints
```

# 12. Search-engine reconnaissance

```text
site:example.com
```
and combinations involving:
```text
site:
filetype:
inurl:
intitle:
```

# 13. Certificate Transparency

Learn how certificates can reveal infrastructure.
For example:
```text
example.com
      ↓
TLS certificate
      ↓
Certificate Transparency logs
      ↓
api.example.com
dev.example.com
staging.example.com
```
A commonly used public service is **crt.sh**.

# 14. Cloud asset discovery

Eventually learn the basics of:
```text
AWS
Azure
GCP
```
and concepts like:
```text
S3 buckets
CloudFront
Azure Blob Storage
Cloud storage
Cloud IP ranges
CNAMEs
```

# 15. What about Shodan/Censys?
They can help discover publicly exposed:
```text
IP addresses
Ports
Services
Certificates
Technologies
```
They're useful for **passive/Internet-wide reconnaissance**, but you don't need to become an expert in them immediately.

# Core recon toolkit

|Purpose|Tool|
|---|---|
|Proxy/manual testing|**Burp Suite**|
|Subdomains|**Amass**|
|Subdomains|**Subfinder**|
|Live hosts|**httpx**|
|Ports/services|**Nmap**|
|URL crawling|**Katana**|
|Historical URLs|**GAU / Waybackurls**|
|Content discovery|**ffuf**|
|Parameters|**Arjun**|
|DNS|**dig**|
|Tech detection|**WhatWeb**|
|Wordlists|**SecLists**|
|Internet exposure|**Shodan / Censys**|
|Certificate discovery|**crt.sh**|

# The workflow I'd recommend for you

Suppose you have an authorized bug bounty target:

```text
example.com
```

Your workflow could eventually look like:

```text
                 example.com
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
   Passive Recon             DNS Recon
          │                       │
   Amass/Subfinder              dig
          │
          ↓
   subdomains.txt
          │
          ↓
        httpx
          │
          ↓
   Live web applications
          │
     ┌────┴────────┐
     ↓             ↓
   Katana         ffuf
     ↓             ↓
   URLs         directories
     │
     ├──────────────┐
     ↓              ↓
 JavaScript      Parameters
     │              │
     ↓              ↓
 endpoints       Arjun
     │              │
     └───────┬──────┘
             ↓
        Burp Suite
             ↓
     Manual testing
             ↓
 ┌───────────┼────────────┐
 ↓           ↓            ↓
XSS        IDOR        SQLi/SSRF
             etc.
```

**This is the part I want you to focus on:** recon isn't just "run Amass → run Nmap → run ffuf." The objective is to continuously **expand and understand the attack surface**.

---

# Resources I'd prioritize

### 1. PortSwigger Web Security Academy

Since you're already using it, keep it as your **main web-security learning platform**. Your current XSS work fits perfectly here.

### 2. OWASP Web Security Testing Guide

Use it as a methodology/reference rather than trying to read it cover-to-cover.

Focus on:

```text
Information Gathering
Configuration/Deployment Management Testing
Identity Management
Authentication
Authorization
Session Management
Input Validation
```

### 3. ProjectDiscovery documentation
Since you'll likely use `httpx`, `katana`, and related tools, learn the tools from their documentation rather than relying entirely on random YouTube commands.


# Most importantly for YOU

**Broken Access Control → XSS → API testing → bug bounty**
I'd prioritize recon in this order:
```text
1. DNS basics
2. Passive subdomain enumeration
3. Amass + Subfinder
4. httpx
5. Nmap
6. URL discovery
7. Katana + GAU
8. ffuf/content discovery
9. JavaScript reconnaissance
10. Parameter discovery
11. Burp Suite
12. Cloud/Internet-wide recon
```

A good workflow is:
```text
Learn recon concept
      ↓
Practice on your own lab
      ↓
Add one tool
      ↓
Understand its output
      ↓
Use discovered endpoints in Burp
      ↓
Test XSS / IDOR / API authorization
```

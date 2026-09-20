resources:
- [ffuf](http://ffuf.me/)
- [OWASP](https://community.owasp.org/Fuzzing)

yt:
- [fuzzing basics](https://youtu.be/0v1CTSyRpMU)
- [ffuf tutorial](https://youtu.be/HFdFpr-Lf9Y)
- [hackersploit ffuf](https://youtu.be/9Hik0xy9qd0)
- 

_Fuzz testing_, or _fuzzing_, is a software testing technique aimed at identifying bugs, vulnerabilities, or unexpected behavior by automatically providing a program with unexpected, malformed, or semi-malformed inputs. [OWASP](https://community.owasp.org/Fuzzing)
Fuzzing is the art of automatic bug finding, and its role is to find software implementation faults, and identify them if possible.
## fuzzer implementations
A fuzzer is a program which injects automatically semi-random data into a program/stack and detect bugs.

The data-generation part is made of generators, and vulnerability identification relies on debugging tools. Generators usually use combinations of static fuzzing vectors (known-to-be-dangerous values), or totally random data. New generation fuzzers use genetic algorithms to link injected data and observed impact. Such tools are not public yet.

Tools:
- ffuf
- dirbuster

Types:
- directory/file
- subdomain fuzzing
- parameter
- value

fuzzing wordlists/seclists:
## Recursive fuzzing:
when ffuf finds a directory (or path) that looks valid, it automatically starts fuzzing inside that directory too, without you having to run a new command manually.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]

Think of it as:  
when you find `/admin/`, now fuzz `/admin/FUZZ` 
if you find `/admin/api/`, now fuzz `/admin/api/FUZZ`, and so on, up to a depth you choose.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]
### Basic idea with an example

Normal (non-recursive) fuzzing:

```
ffuf -u "https://target.com/FUZZ" -w wordlist.txt -fc 404
```

This only tests paths like:

- `https://target.com/login`
- `https://target.com/admin`
- `https://target.com/api`
- etc.

If it finds `/admin/` (status 200/301), it **stops there** unless you run another command for `/admin/FUZZ`.

Recursive fuzzing:

```
ffuf -u "https://target.com/FUZZ" \
  -w wordlist.txt \
  -recursion \
  -recursion-depth 3 \
  -recursion-strategy found-only \
  -fc 404
```

Now ffuf will:

1. Fuzz `https://target.com/FUZZ`.
2. When it finds something like `https://target.com/admin/` (a directory), it:
    - Adds a new job to fuzz `https://target.com/admin/FUZZ`.
3. If it then finds `https://target.com/admin/api/`, and your depth allows, it:
    - Starts fuzzing `https://target.com/admin/api/FUZZ`.
4. Continues until it hits the `-recursion-depth` limit.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]
### Key ffuf recursion options

These are the main flags you should understand:
#### `-recursion`
Turns on recursive fuzzing.
```
-recursion
```
#### `-recursion-depth N`
Limits how deep ffuf will go.
- `-recursion-depth 1` → only root + one level (`/`, `/admin/`, but not `/admin/api/`).
- `-recursion-depth 3` → up to three levels deep.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]
Example:
```
-recursion-depth 3
```
This prevents infinite or extremely deep crawling.
#### `-recursion-strategy`
Controls **which responses trigger recursion**.
Common strategies:
- `found-only`  
    Only recurse on “found” directories (usually status codes you consider as “real”, e.g. 200, 301, 302).  
    This is the safest and most common.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]
- `found-and-redirects` (or similar, depending on version)  
    Also recurse on some redirects; can be useful but noisier.

You typically want:
```
-recursion-strategy found-only
```
combined with your normal filters like `-fc 404`.
#### `-recursion-base`
Used when your URL pattern isn’t simply `.../FUZZ` but something like:
```
-u "https://target.com/FUZZ/"
```
or when you want to control how the next level is built. In most basic cases, you don’t need to touch this; ffuf infers it from your `-u` and `FUZZ` position.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]
### When to use recursive fuzzing

Use it when:
- You’re doing **directory/file discovery** on a web app.
- You expect nested structures like:
    - `/admin/`, `/admin/api/`, `/admin/users/`
    - `/api/v1/`, `/api/v2/`
- You want ffuf to **automatically explore subdirectories** as it finds them.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]

Avoid or limit it when:
- The target is large and you don’t want huge traffic.
- You’re rate-limited or in a strict bug bounty program.
- You only care about top-level paths.

In those cases, use a small `-recursion-depth` (1 or 2) or skip recursion entirely.
### How this helps in bug bounty / appsec
Recursive fuzzing helps you:
- Discover hidden admin panels, APIs, and internal tools nested under found directories.
- Find deeper endpoints that might have weaker security (e.g., `/admin/debug/`, `/api/internal/`).
- Automate what would otherwise be many manual ffuf runs.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]

Just be mindful of:
- Program rules on scanning intensity.
- Noise and WAF triggers from aggressive recursion.
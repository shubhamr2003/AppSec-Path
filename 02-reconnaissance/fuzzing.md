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


## ffuf

ffuf commands you should be familiar with
## 1. `-mc` — Match Status Codes
`-mc` means **match HTTP status codes**.

Example:
```bash
ffuf -u http://192.168.189.130/dvwa/FUZZ \
-w /home/kali/big.txt \
-mc 200,301,302,403
```

This means:
> Only show results where the server returns `200`, `301`, `302`, or `403`.

For example:
```text
/admin       → 403  ✅ show
/login.php   → 200  ✅ show
/test        → 404  ❌ don't show
```
### When to use it
Useful when you know which status codes you're interested in.

# 2. `-fc` — Filter Status Codes
`-fc` is the opposite idea.

It means:
> **Don't show responses with these status codes.**

Example:
```bash
ffuf -u http://192.168.189.130/dvwa/FUZZ \
-w /home/kali/big.txt \
-fc 404
```

If ffuf gets:
```text
/admin       → 403
/login       → 200
/random      → 404
/test        → 404
```

It displays:
```text
/admin       → 403
/login       → 200
```
and hides the `404`s.
### Very common usage
```bash
-fc 404
```
because directory/file fuzzing often generates lots of `404 Not Found` responses.

# 3. `-fs` — Filter by Response Size
This one is **very important**.
`-fs` means:
> Don't show responses having a particular **response size**.

Suppose you fuzz:
```text
/admin
/login
/random
/test
```

and the application responds:
```text
/admin      → 404 → 1543 bytes
/login      → 200 → 4210 bytes
/random     → 404 → 1543 bytes
/test       → 404 → 1543 bytes
```

The `404`s all have the same size:
```text
1543
```

You can filter them:
```bash
-fs 1543
```
Now ffuf hides those responses.
This is useful when an application returns a **custom 404 page** with status `200`.

For example:
```text
/random       → 200 → 8456 bytes
/asdfasdf     → 200 → 8456 bytes
/xyz123       → 200 → 8456 bytes
/admin        → 200 → 10231 bytes
```
Status-code filtering won't help because **everything is 200**.

But:
```bash
-fs 8456
```
can remove the fake results.

# 4. `-fw` — Filter by Word Count
`-fw` means:
> Filter responses based on their **number of words**.

Example:
```text
/random → 200 → 152 words
/test   → 200 → 152 words
/admin  → 200 → 194 words
```

You could use:
```bash
-fw 152
```
Then ffuf hides the responses containing 152 words.
This is another way of identifying and filtering **false positives**

# 5. `-fl` — Filter by Line Count
Similar idea:
```bash
-fl 25
```

means:
> Hide responses containing exactly 25 lines.

Example:
```text
/random → 200 → 25 lines
/test   → 200 → 25 lines
/admin  → 200 → 42 lines
```
`-fl 25` hides the first two.

# 6. `-e` — Extensions
This is extremely useful for directory/file discovery.

Suppose your wordlist contains:
```text
login
admin
config
```

You can tell ffuf to automatically try extensions:
```bash
-e .php,.txt,.bak
```

So ffuf tries things like:
```text
/login
/login.php
/login.txt
/login.bak

/admin
/admin.php
/admin.txt
/admin.bak

/config
/config.php
/config.txt
/config.bak
```

For a PHP application such as DVWA:
```bash
-e .php,.txt,.bak
```
can be useful.

# 7. `-t` — Threads
You already saw:
```bash
-t 40
```
This means **40 concurrent requests**.

Example:
```bash
-t 10
```
→ slower, lighter

```bash
-t 40
```
→ faster, more concurrent requests

```bash
-t 100
```
→ significantly more aggressive

# 8. `-r` — Follow Redirects
You currently have:
```text
Follow redirects : false
```

You can enable it:
```bash
-r
```

Example:
```bash
ffuf -u http://192.168.189.130/dvwa/FUZZ \
-w /home/kali/big.txt \
-r
```

If:
```text
/admin → 302 → /login.php
```
ffuf follows the redirect.

# 9. `-v` — Verbose
```bash
-v
```
gives you more information about results.

Example:
```bash
ffuf ... -v
```
Useful while you're learning because you can see more details about the discovered URLs.

# 10. `-o` — Save Results
You can save your scan:
```bash
-o results.json
```

and specify the format:
```bash
-of json
```

Example:
```bash
ffuf -u http://192.168.189.130/dvwa/FUZZ \
-w /home/kali/big.txt \
-o results.json \
-of json
```
Other formats include HTML, CSV, etc.

For your current **web pentesting learning**, focus on these:

```text
-u     Target URL
-w     Wordlist
-FUZZ  Fuzzing position
-mc    Match status
-fc    Filter status
-fs    Filter response size
-fw    Filter word count
-fl    Filter line count
-e     Extensions
-t     Threads
-r     Follow redirects
-o     Output
```

## A realistic DVWA example

Start simple:

```bash
ffuf -u http://192.168.189.130/dvwa/FUZZ \
-w /home/kali/big.txt \
-mc 200,301,302,403
```

Then if you're getting tons of `404`s:

```bash
ffuf -u http://192.168.189.130/dvwa/FUZZ \
-w /home/kali/big.txt \
-fc 404
```

For PHP files:

```bash
ffuf -u http://192.168.189.130/dvwa/FUZZ \
-w /home/kali/big.txt \
-e .php,.txt,.bak \
-fc 404
```

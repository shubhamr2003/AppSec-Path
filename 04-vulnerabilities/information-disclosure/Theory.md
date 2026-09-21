resource:
[portswigger](https://portswigger.net/web-security/information-disclosure/exploiting#how-to-test-for-information-disclosure-vulnerabilities)

## What is information disclosure?

Information disclosure, also known as information leakage, is when a website unintentionally reveals sensitive information to its users. Depending on the context, websites may leak all kinds of information to a potential attacker, including:

- Data about other users, such as usernames or financial information
- Sensitive commercial or business data
- Technical details about the website and its infrastructure

The dangers of leaking sensitive user or business data are fairly obvious, but disclosing technical information can sometimes be just as serious. Although some of this information will be of limited use, it can potentially be a starting point for exposing an additional attack surface, which may contain other interesting vulnerabilities. The knowledge that you are able to gather could even provide the missing piece of the puzzle when trying to construct complex, high-severity attacks.

Occasionally, sensitive information might be carelessly leaked to users who are simply browsing the website in a normal fashion. More commonly, however, an attacker needs to elicit the information disclosure by interacting with the website in unexpected or malicious ways. They will then carefully study the website's responses to try and identify interesting behavior.

### Examples of information disclosure

Some basic examples of information disclosure are as follows:

- Revealing the names of hidden directories, their structure, and their contents via a `robots.txt` file or directory listing
- Providing access to source code files via temporary backups
- Explicitly mentioning database table or column names in error messages
- Unnecessarily exposing highly sensitive information, such as credit card details
- Hard-coding API keys, IP addresses, database credentials, and so on in the source code
- Hinting at the existence or absence of resources, usernames, and so on via subtle differences in application behavior
## How do information disclosure vulnerabilities arise?

Information disclosure vulnerabilities can arise in countless different ways, but these can broadly be categorized as follows:

- **Failure to remove internal content from public content**. For example, developer comments in markup are sometimes visible to users in the production environment.
- **Insecure configuration of the website and related technologies**. For example, failing to disable debugging and diagnostic features can sometimes provide attackers with useful tools to help them obtain sensitive information. Default configurations can also leave websites vulnerable, for example, by displaying overly verbose error messages.
- **Flawed design and behavior of the application**. For example, if a website returns distinct responses when different error states occur, this can also allow attackers to [enumerate sensitive data](https://portswigger.net/web-security/authentication/password-based#username-enumeration), such as valid user credentials.
## How to test for information disclosure vulnerabilities

Generally speaking, it is important not to develop "tunnel vision" during testing. In other words, you should avoid focussing too narrowly on a particular vulnerability. Sensitive data can be leaked in all kinds of places, so it is important not to miss anything that could be useful later. You will often find sensitive data while testing for something else. A key skill is being able to recognize interesting information whenever and wherever you do come across it.

The following are some examples of high-level techniques and tools that you can use to help identify information disclosure vulnerabilities during testing.

- [Fuzzing](https://portswigger.net/web-security/information-disclosure/exploiting#fuzzing)
- [Using Burp Scanner](https://portswigger.net/web-security/information-disclosure/exploiting#using-burp-scanner)
- [Using Burp's engagement tools](https://portswigger.net/web-security/information-disclosure/exploiting#using-burp-s-engagement-tools)
- [Engineering informative responses](https://portswigger.net/web-security/information-disclosure/exploiting#engineering-informative-responses)

## Common sources of information disclosure

Information disclosure can occur in a wide variety of contexts within a website. The following are some common examples of places where you can look to see if sensitive information is exposed.
- Files for web crawlers
- Directory listings
- Developer comments
- Error messages
- Debugging data
- User account pages
- Backup files
- Insecure configuration
- Version control history

## 1) Files for web crawlers

### What it is

Websites often publish files that tell search engines and crawlers what to index:

- `robots.txt`
- `sitemap.xml`
- Sometimes `.well-known/...` files used by bots or verification.

These files are meant for crawlers, but they’re **publicly accessible** and often reveal:

- Hidden or “private” paths the site doesn’t want users to see.
- Admin panels, API endpoints, staging environments.
- Internal directories like `/backup/`, `/old/`, `/admin/`, `/api/internal/`.[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### Why it leaks information

Developers think: “Search engines shouldn’t index this,” so they add:

```
# robots.txt
Disallow: /admin/
Disallow: /backup/
Disallow: /api/internal/
Disallow: /staging/
```

But anyone can fetch:

```
https://target.com/robots.txt
https://target.com/sitemap.xml
```

and see exactly which paths exist.

### Where to look

- `https://target.com/robots.txt`
- `https://target.com/sitemap.xml`
- Sometimes nested sitemaps referenced inside `sitemap.xml`.

### How to test / use it

1. Fetch these files manually or with a script.
2. Extract all paths (especially `Disallow` entries in `robots.txt` and `<loc>` entries in `sitemap.xml`).
3. Visit them in a browser or fuzz them with ffuf/Burp.
4. Look for:
    
    - Admin interfaces.
    - Debug/staging pages.
    - Backup directories.
    - Internal APIs.

This is often the **first step in recon** because it’s low-noise and can immediately reveal interesting targets.[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## 2) Directory listings

### What it is

When a web server is misconfigured and a directory has no index file (like `index.html` or `index.php`), some servers automatically show a **listing of all files** in that directory:

```
Index of /backup/
-----------------
../
db.sql
config.php.bak
logs/
```

This is called **directory listing** or **directory indexing**.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

### Why it leaks information

Instead of a 403/404, you get a full file list. Attackers can:

- Download backups, configs, logs.
- Discover sensitive files they wouldn’t have guessed.
- Map out the application structure.

### Where to look

Common paths to try:

- `/backup/`
- `/backups/`
- `/logs/`
- `/uploads/`
- `/files/`
- `/admin/`
- `/old/`
- `/test/`
- Any path you found via `robots.txt`, subdomain enumeration, or fuzzing.

### How to test / exploit

1. Fuzz directories with ffuf:
    
    ```
    ffuf -u "[https://target.com/FUZZ](https://target.com/FUZZ)" \
      -w wordlist.txt \
      -fc 404 \
      -t 30
    ```
    
2. For each 200/301 response that’s a directory, open it in a browser.
3. If you see a file listing:
    
    - Download interesting files (`.sql`, `.bak`, `.log`, `.env`, `.php`, `.zip`).
    - Inspect for credentials, PII, internal URLs, tokens.

Even one exposed backup file can lead to full compromise.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

---

## 3) Developer comments

### What it is

Developers often leave comments in:

- HTML: `<!-- TODO: remove debug endpoint -->`
- JavaScript: `// FIXME: hard-coded API key for testing`
- CSS: `/* Remove this before prod */`

These comments are sent to the browser and visible in “View Source” or DevTools.[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### Why it leaks information

Comments can reveal:

- Hidden endpoints: `<!-- API: https://internal-api.internal/v1 -->`
- Credentials or keys (sometimes).
- Logic about auth, roles, feature flags.
- Upcoming features or admin paths.

Example:

```
<!-- Admin panel: /supersecretadmin -->
<!-- Debug mode: set ?debug=1 -->
```

### Where to look

- Main HTML pages (home, login, dashboard).
- JS files (`app.js`, `main.*.js`, chunk files in SPAs).
- CSS files (less common but possible).

### How to test / exploit

1. In browser:
    
    - Right-click → “View Page Source”.
    - Search for keywords: `TODO`, `FIXME`, `admin`, `debug`, `key`, `token`, `password`, `secret`.
2. In Burp:
    
    - Use “Search” across all responses for patterns like:
        
        - `<!--`, `//`, `/*`
        - `admin`, `debug`, `staging`, `internal`.
3. Follow any paths or hints:
    
    - Visit `/supersecretadmin`.
    - Try `?debug=1` on key pages.
    - Look for internal URLs and test for SSRF or further recon.

Comments alone may be “low” severity, but if they reveal admin panels, debug modes, or secrets, they become high-value.[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## 4) Error messages

### What it is

When something goes wrong (invalid input, DB error, exception), the app returns an error page or JSON response. If not properly handled, these can include:

- Stack traces.
- SQL queries.
- File paths.
- Framework internals.
- Database schema details.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

### Why it leaks information

Verbose errors help attackers:

- Understand the tech stack (language, framework).
- See exact SQL queries → easier SQLi.
- Learn file system layout → path traversal, LFI, RCE.
- Identify libraries and versions → known CVEs.

Example:

```
PDOException: SQLSTATE[42000]: Syntax error...
SELECT * FROM users WHERE email = 'test' AND status = 1
in /var/www/app/src/UserRepo.php on line 57
```

Now you know:

- It’s PHP + PDO.
- There’s a `users` table with `email` and `status`.
- Full server path.

### Where to look

Trigger errors by:

- Sending invalid types:
    
    - `?id=not_a_number` where an integer is expected.
- Missing required fields.
- Extremely long inputs.
- Malformed JSON.
- Wrong HTTP methods (e.g., `DELETE` where not allowed).

Check:

- HTML error pages.
- JSON error responses (`{"error": "...", "stack": "..."}`).
- HTTP 500/502/503 responses.

### How to test / exploit

1. In Burp Repeater, modify parameters:
    
    - Change `id=123` → `id=abc`.
    - Remove required fields.
    - Send malformed JSON: `{"user": {` (incomplete).
2. Observe responses:
    
    - Look for stack traces, SQL fragments, paths.
3. Use disclosed info to:
    
    - Craft better SQLi payloads.
    - Guess table/column names.
    - Understand framework for targeted exploits.

Many programs treat detailed error disclosure as at least **medium**, especially if it clearly aids further attacks.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

---

## 5) Debugging data

### What it is

Apps sometimes run with debug flags or expose debugging endpoints that return:

- Detailed error pages with interactive consoles.
- Environment variables.
- Configuration details.
- Request/response dumps.
- Profiling data.

Examples:

- Django debug pages.
- Laravel debug/error pages.
- Spring Boot Actuator (`/actuator`, `/actuator/env`, `/actuator/heapdump`).
- Custom debug endpoints like `/debug`, `/debugvars`, `/phpinfo.php`.[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### Why it leaks information

Debug pages often show:

- All environment variables (DB passwords, API keys).
- Full settings (email servers, internal URLs).
- Installed packages and versions.
- Sometimes code execution interfaces (e.g., Django debug console).

### Where to look

Common paths:

- `/debug`
- `/phpinfo.php`
- `/actuator`, `/actuator/env`, `/actuator/health`, `/actuator/mappings`
- `/console`, `/admin/console`
- `/swagger`, `/api-docs` (can reveal endpoints and schemas)
- Query parameters like `?debug=1`, `?show_errors=1`, `?env=dev`.

### How to test / exploit

1. Fuzz for debug-like paths:
    
    - Use wordlists with `debug`, `actuator`, `phpinfo`, `console`.
2. Try toggling debug flags:
    
    - `https://target.com/?debug=1`
    - `https://target.com/login?show_errors=true`
3. Inspect responses for:
    
    - Environment variables.
    - DB credentials.
    - Internal service URLs.
4. If you find something like Django’s debug page or an unprotected Actuator endpoint:
    
    - Treat as critical; it may allow code execution or full config disclosure.

Debug endpoints are often left enabled by mistake in staging or even production.[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## 6) User account pages

### What it is

Pages or APIs that display user profile/account information:

- `/profile`, `/account`, `/settings`
- `/api/user`, `/api/profile`, `/api/me`

These can leak:

- PII: emails, phone numbers, addresses.
- Internal IDs, roles, permissions.
- Security settings: MFA status, recovery emails.
- Linked accounts (OAuth, social logins).

### Why it leaks information

Two main patterns:

1. **Over-fetching**:  
    The API returns more fields than the UI shows.  
    Example: UI shows only username, but JSON includes email, phone, role.
2. **IDOR/BOLA on user resources**:  
    Changing `user_id` or similar parameter lets you view other users’ profiles.  
    This is both IDOR and information disclosure.

### Where to look

- Profile/settings pages in the app.
- Any endpoint returning user data:
    
    - `/api/users/{id}`
    - `/api/profile`
    - `/api/account`
- Mobile app APIs (inspect traffic with Burp/Proxy).

### How to test / exploit

1. Log in as user A.
2. Capture profile/account requests in Burp.
3. Inspect JSON/HTML:
    
    - Compare what’s shown in UI vs what’s in the response.
    - Look for extra fields (email, phone, role, MFA flags).
4. For IDOR:
    
    - Create user B.
    - As user A, request `/api/users/<B_ID>` or modify `user_id` in profile requests.
    - See if you can retrieve B’s data.

Impact:

- Mass PII exposure.
- Enumeration of valid user IDs/emails.
- Potential account takeover if sensitive fields (tokens, recovery data) are exposed.

---

## 7) Backup files

### What it is

Developers or admins sometimes leave backup copies of files on the server:

- `config.php.bak`
- `db.sql`, `dump.sql`
- `site.zip`, `backup.tar.gz`
- `config.old`, `.bak`, `.orig`, `.save`

These can be directly downloadable if the path is known or guessable.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

### Why it leaks information

Backups often contain:

- Database dumps → all user data, passwords (hashed or not), sessions.
- Config files → DB credentials, API keys, internal URLs.
- Source code → logic, secrets, hardcoded tokens.

### Where to look

Common patterns:

- `/backup/`, `/backups/`
- `/db/`, `/database/`
- Root or config directories:
    
    - `config.php.bak`
    - `.env.bak`
    - `settings.py.orig`
- Names related to the app:
    
    - `site.zip`, `app.sql`, `prod_backup.sql`

Use:

- Fuzzing with backup-focused wordlists.
- Insights from `robots.txt`, comments, error messages.

### How to test / exploit

1. Fuzz for backup files:
    
    ```
    ffuf -u "[https://target.com/FUZZ](https://target.com/FUZZ)" \
      -w backups-wordlist.txt \
      -fc 404
    ```
    
2. For each 200 response:
    
    - Download the file.
    - Inspect for:
        
        - DB credentials.
        - API keys.
        - User data.
3. If you find a DB dump:
    
    - Check for password hashes, emails, tokens.
    - Report as critical if it exposes many users’ data.

Even a single config backup with DB credentials can lead to full compromise.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

---

## 8) Insecure configuration

### What it is

Misconfigurations that cause the server or app to reveal more than intended:

- Directory listings enabled (covered earlier).
- Default pages with version info.
- Exposed admin interfaces without proper protection.
- Permissive CORS policies.
- Verbose HTTP headers (`Server`, `X-Powered-By`).
- Publicly accessible management consoles (e.g., Tomcat manager, phpMyAdmin).[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### Why it leaks information

These configurations:

- Reveal technology stack and versions.
- Expose admin tools that can be brute-forced or exploited.
- Allow other origins to read sensitive responses (CORS).
- Show internal hostnames, IPs, or infrastructure details.

Examples:

```
Server: nginx/1.18.0 (Ubuntu)
X-Powered-By: PHP/7.4.3
```

Or:

- `phpMyAdmin` exposed at `/phpmyadmin/`.
- Tomcat manager at `/manager/html`.

### Where to look

- HTTP headers on all responses.
- Default landing pages.
- Common admin/management paths:
    
    - `/phpmyadmin/`
    - `/manager/html`
    - `/admin/`
    - `/console/`
- CORS headers:
    
    - `Access-Control-Allow-Origin: *` with credentials.

### How to test / exploit

1. Inspect headers in Burp:
    
    - Look for `Server`, `X-Powered-By`, custom debug headers.
2. Fuzz for admin/management consoles.
3. Test CORS:
    
    - Send requests with `Origin: https://attacker.com`.
    - See if `Access-Control-Allow-Origin` echoes your origin and allows credentials.
4. For each exposed console:
    
    - Try default credentials.
    - Search for known vulnerabilities for that software/version.

Insecure configuration is often rated based on what it exposes (versions only → low; admin console → medium/high).[[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## 9) Version control history

### What it is

Accidentally exposed version control directories or files:

- `.git/`
- `.svn/`
- `.hg/`
- Sometimes partial repos or `.git/config`, `.git/HEAD`, etc.

If accessible, attackers can reconstruct the **entire source code history**, including:

- Deleted files.
- Old versions with secrets.
- Commit messages revealing internals.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

### Why it leaks information

A public `.git` directory allows:

- Downloading objects and reconstructing the repo.
- Viewing all commits, branches, tags.
- Recovering removed secrets (API keys, passwords) that were committed then deleted.

Tools exist to automate this (e.g., `git-dumper`, custom scripts).

### Where to look

- `https://target.com/.git/`
- `https://target.com/.git/config`
- `https://target.com/.svn/`
- Subdirectories where the app is deployed incorrectly.

### How to test / exploit

1. Check for `.git`:
    
    ```
    curl -I [https://target.com/.git/config](https://target.com/.git/config)
    ```
    
    If you get 200 OK and a config file, `.git` is exposed.
    
2. Use tools (ethically, only on authorized targets) to:
    
    - Dump the repo.
    - Inspect commit history.
3. Search for:
    
    - Hard-coded API keys, tokens, DB passwords.
    - Internal URLs, infrastructure details.
    - Removed features or endpoints.

Exposed `.git` is often treated as **high/critical**, especially if it reveals secrets or full source.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)]

---

## How to use this in practice

When testing a target:

1. Start with low-noise checks:
    
    - `robots.txt`, `sitemap.xml`.
    - Basic headers.
    - View source for comments.
2. Then move to active testing:
    
    - Fuzz for directories, backups, debug paths.
    - Trigger errors intentionally.
    - Inspect user/account APIs for over-fetching and IDOR.
3. For anything interesting:
    
    - Download files.
    - Reconstruct repos if `.git` is exposed.
    - Chain disclosures into deeper issues (SQLi, auth bypass, RCE).

If you want, I can turn this into a compact checklist you can keep open while testing any new target.

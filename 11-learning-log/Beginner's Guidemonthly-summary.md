### 1. PortSwigger Web Security Academy — your #1 resource

PortSwigger Web Security Academy

[PortSwigger Web Security Academy](https://portswigger.net/web-security?utm_source=chatgpt.com)

Keep this as your **main practical training platform**. It has structured learning paths and hands-on labs covering SQLi, XSS, authentication, access control, SSRF, API testing, business logic, WebSockets, and more.

**Use it for:**  
`Learn vulnerability → practice → understand exploitation`

For your current level, I'd prioritize:

1. Authentication
    
2. Access Control
    
3. SQL Injection
    
4. XSS
    
5. CSRF
    
6. SSRF
    
7. File Upload
    
8. Path Traversal
    
9. API Testing
    
10. Business Logic
    
11. Race Conditions
    
12. Web Cache Poisoning
    

---

## 2. Intigriti Hacking Labs — excellent transition toward bug bounty

[Intigriti Hacking Labs](https://labs.intigriti.io/?utm_source=chatgpt.com)

This is particularly relevant **after PortSwigger**, because it explicitly combines a bug-bounty roadmap with interactive labs. Its current roadmap covers reconnaissance, core vulnerabilities such as BAC, SQLi, XSS, SSRF and information disclosure, and then moves toward reporting your first bug. ([Intigriti Labs](https://labs.intigriti.io/?utm_source=chatgpt.com "Hacking Labs | Intigriti"))

I'd put this high on your list.

**Use it for:**  
`Web security knowledge → bug bounty methodology`

---

## 3. Hacker101 — free + CTFs

[Hacker101](https://www.hacker101.com/?utm_source=chatgpt.com)

Hacker101 is HackerOne's free web-security education platform. It combines lessons with CTF challenges based on real-world vulnerability concepts. ([HackerOne](https://www.hackerone.com/hackers/how-to-start-hacking?utm_source=chatgpt.com "Start Hacking & Join the Largest Hacker Community | HackerOne"))

This is useful because it makes you think:

> "I have an application. Where would I look for a bug?"

rather than:

> "The lab tells me this is an XSS lab."

---

# 4. Bugcrowd University

[Bugcrowd University on GitHub](https://github.com/bugcrowd/bugcrowd_university?utm_source=chatgpt.com)

This is one I'd definitely add to your `09-resources`.

It's free and open source and includes modules on:

- Recon & Discovery
    
- Burp Suite
    
- Broken Access Control
    
- XSS
    
- SSRF
    
- GitHub recon
    
- Sensitive data exposure
    
- XXE
    
- Advanced Burp Suite
    

It also includes videos and lab material for several modules. ([GitHub](https://github.com/bugcrowd/bugcrowd_university?utm_source=chatgpt.com "GitHub - bugcrowd/bugcrowd_university: Open source education content for the researcher community · GitHub"))

---

# 5. Intigriti Hackademy

[Intigriti Hackademy](https://www.intigriti.com/researchers/hackademy?utm_source=chatgpt.com)

This is fantastic for **reading rather than doing labs**.

It provides vulnerability explanations, real-world examples, writeups, bug-bounty tips and videos. ([Intigriti](https://www.intigriti.com/researchers/hackademy?utm_source=chatgpt.com "Hackademy | Intigriti"))

Use it when you've finished a vulnerability category and want to see:

> "How does this vulnerability actually appear in bug bounty programs?"

---

# 6. HackerOne Hacktivity — VERY important

[HackerOne Hacktivity](https://hackerone.com/hacktivity?utm_source=chatgpt.com)

This is where you start transitioning from **training labs to real-world vulnerability research**.

Hacktivity lets you search disclosed HackerOne reports by vulnerability/weakness and program. ([HackerOne Help Center](https://docs.hackerone.com/en/articles/8410358-hacktivity?utm_source=chatgpt.com "Hacktivity | HackerOne Help Center"))

For example, after learning IDOR:

Search Hacktivity for:

> `IDOR`

Then read 10 reports.

Don't just look at the payload.

Study:

```text
How did they find it?
        ↓
What endpoint did they notice?
        ↓
What assumption did the developer make?
        ↓
What did the researcher manipulate?
        ↓
How did they prove impact?
        ↓
How did they write the report?
```

**This is where your bug-bounty mindset starts developing.**

---

# 7. HackTricks

[HackTricks Web Pentesting](https://book.hacktricks.wiki/en/index.html?utm_source=chatgpt.com)

Use this as your **reference manual** rather than your primary course.

It's especially useful for:

- Recon
    
- Enumeration
    
- Web testing
    
- API testing
    
- Authentication
    
- Cloud
    
- SSRF
    
- File upload
    
- Command injection
    
- Infrastructure
    

Think:

**PortSwigger = classroom**

**HackTricks = field notebook**

---

# 8. Bug Bounty Hunter Methodology

Study Jason Haddix's methodology.

[Bug Bounty Hunter Methodology v3](https://www.bugcrowd.com/resources/levelup/bug-bounty-hunter-methodology-v3/?utm_source=chatgpt.com)

This is important because eventually you need to move from:

> "I know how to exploit XSS."

to:

> "I know how to approach an unfamiliar application."

Bugcrowd specifically describes this methodology as focusing on web application testing and bug-hunting methodology. ([Bugcrowd](https://www.bugcrowd.com/resources/levelup/bug-bounty-hunter-methodology-v3/?utm_source=chatgpt.com "Bug Bounty Hunter Methodology v3 | Bugcrowd"))

---

# 9. PentesterLab

[PentesterLab](https://pentesterlab.com/?utm_source=chatgpt.com)

This is another excellent practical resource, particularly when you want to move toward **more realistic application-security exercises**.

I'd put it after you've built a good foundation with PortSwigger.

---

# 10. Real-world writeups

Once you're comfortable with the basics, start reading:

### HackerOne

[HackerOne](https://www.hackerone.com/hackers?utm_source=chatgpt.com)

Use disclosed reports and Hacktivity to study real findings. HackerOne currently provides both educational resources and access to bug-bounty programs. ([HackerOne](https://www.hackerone.com/hackers?utm_source=chatgpt.com "HackerOne for Hackers"))

### Intigriti

[Intigriti](https://www.intigriti.com/?utm_source=chatgpt.com)

Their Hackademy and labs are particularly useful for learning from real-world examples. ([Intigriti](https://www.intigriti.com/researchers/hackademy?utm_source=chatgpt.com "Hackademy | Intigriti"))

---

# How I'd structure YOUR learning

Since you're already working on `AppSec-Path`, I'd follow this:

```text
                WEB FUNDAMENTALS
                       ↓
              PORTSWIGGER ACADEMY
                       ↓
           ┌───────────┴───────────┐
           ↓                       ↓
      DVWA / Juice Shop       Burp Suite
           │                       │
           └───────────┬───────────┘
                       ↓
              INTIGRITI HACKING LABS
                       ↓
                HACKER101 CTF
                       ↓
             BUGCROWD UNIVERSITY
                       ↓
            BUG HUNTER METHODOLOGY
                       ↓
                  HACKTRICKS
                       ↓
              HACKTIVITY REPORTS
                       ↓
            REAL BUG BOUNTY PROGRAMS
```

You **don't need to complete every resource** before moving forward.

---

# The important transition

I'd divide your journey into **three stages**.

### 🟢 Stage 1 — Learn vulnerabilities

Use:

- PortSwigger
    
- DVWA
    
- Juice Shop
    

Goal:

> **"I understand how common web vulnerabilities work."**

---

### 🟡 Stage 2 — Learn to hunt

Use:

- Intigriti Labs
    
- Hacker101 CTF
    
- Bugcrowd University
    
- HackTricks
    
- Bug Hunter Methodology
    

Goal:

> **"Give me an unfamiliar application and I know where to start looking."**

---

### 🔴 Stage 3 — Learn from real hunters

Use:

- HackerOne Hacktivity
    
- Intigriti writeups
    
- Public disclosure reports
    
- Researcher blogs
    
- GitHub security research
    

Goal:

> **"I understand how vulnerabilities are discovered in real applications."**

Then move into **authorized public bug-bounty programs**.

---

## And add this to your `AppSec-Path`

I'd make your resources folder:

```text
09-resources/
│
├── platforms.md
├── books.md
├── courses.md
├── websites.md
├── youtube.md
├── blogs.md
├── writeups.md
├── recon.md
└── useful-repositories.md
```

And put these under `platforms.md`:

|Resource|Primary Use|Priority|
|---|---|---|
|PortSwigger Academy|Web vulnerabilities + labs|⭐⭐⭐⭐⭐|
|Intigriti Hacking Labs|Bug bounty practice|⭐⭐⭐⭐⭐|
|Hacker101|Web security + CTF|⭐⭐⭐⭐|
|Bugcrowd University|Methodology + techniques|⭐⭐⭐⭐|
|DVWA|Basic vulnerability practice|⭐⭐⭐|
|Juice Shop|Realistic vulnerable application|⭐⭐⭐⭐|
|PentesterLab|Practical AppSec|⭐⭐⭐⭐|
|HackTricks|Reference|⭐⭐⭐⭐⭐|
|HackerOne Hacktivity|Real-world reports|⭐⭐⭐⭐⭐|

### If I were following your journey

**Don't jump into live bug bounty hunting yet.**

Finish a solid chunk of PortSwigger first, especially **Authentication, Access Control, SQLi, XSS, SSRF and API testing**. Then use Intigriti Labs/Hacker101 to practice hunting without being spoon-fed the vulnerability. After that, spend significant time reading disclosed HackerOne reports before selecting a real program.

That progression will fit very nicely with the **AppSec-Path GitHub + Obsidian system** you're building.
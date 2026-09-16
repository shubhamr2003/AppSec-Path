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
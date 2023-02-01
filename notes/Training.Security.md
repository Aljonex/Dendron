---
id: byiruhqux7nod55fi7g7mxr
title: Security
desc: ''
updated: 1675246781769
created: 1674660404817
---
The *Open Web Application Security Project (**OWASP**)* is an organization dedicated to enhancing security of web apps.

### OWASP Top 10 Vulnerabilites
[[Top 10 | https://owasp.org/www-project-top-ten/]]
1. **Broken Access Control** - access control enforces policies so users can only act within permissions, failure usually leads to unauthorized information disclosure, modification or deletion to occur. Solutions include *deny by default, access control mechanisms*.
2. **Cryptographic Failures** - Previously called *Sensitive Data Exposure*, often leads to exposure of sensitive data. Prevention methods include *identify sensitive data, don't store it unnecessarily, encrypt all sensitive data at rest, classify it and apply controls*.
3. **Injection** - apps are vulnerable when *user-supplied data isn't validated, dynamic queries or non-parameterized calls are used directly in interpreter, hostile data used in ORM search params* preventions mainly revolve around *using safe API, positive server-side input validation, escape special characters*. 
Can also use things like LIMIT, to limit the amount sent back
4. **Insecure Design** - a broad category of missing or ineffective control design (different to insecure implementation), to prevent you should *establish and use secure dev lifecycle and library of secure design patterns, use threat modelling, integrate security language, write unit and integration tests to validate all critical flows are resistant to threat model*.
5. **Security Misconfiguration** - basically if things are missing or unecessary things are installed, defaults are still enabled. Can be prevented with *minimal platform, automated processes to verify effectiveness of configurations*.
6. **Vulnerable and Outdated Components** - this is a struggle to test and is likely bad if you don't know versions of ALL components you use. Out of date can be much less supported and be weak, you should also be scanning for vulnerabilites regularly. Prevention via *patch management* which will *remove unused dependencies, continuously inventory all components, only obtain components from official sources*.
7. **Identification and Authentication Failures** - confirmation of the user's identity, authentication, session management is critical to protect against authentication-related attacks. Prevent with *MFA, weak password checks, align password length with NIST standards*.
8. **Software and Data Integrity Failures** - relate to code and infrastructure that doesn't protect against integrity violations, example is app relying on plugins, libraries, modules from untrusted sources. Insecure CI/CD pipeline can introduce potential for unauthorized access, malicious code or system compromise. Prevent with *digital signatures to verify software source, ensure libraries are using trusted repos, ensure there's review process for code and config changes to minimize chance of malicious code entering*.
9. **Security Logging and Monitoring Failures** - without logging and moitoring, breaches don't get detected. To prevent *log with sufficient user context to identify suspicious activity, ensure logs are generated so log management solutions can parse easily.
10. **Server-Side Request Forgery** - **SSRF** occurs whenever web app is fetching remote resource without validating user-supplied URL. To prevent from the <u>network layer</u> you can *enforce "deny by default" firewall policies* and from the <u>application layer</u> you can *sanitize and validate user-supplied input data, enforce URL schema, disable HTTP redirections*.
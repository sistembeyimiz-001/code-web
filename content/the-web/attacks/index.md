---
title: Attacks
author: vwkd
index: 15
tags:
  - the-web
---

- techniques to exploit vulnerabilities in websites



## Cross Site Scripting (XSS)

- attack that forces user of a website to execute unwanted JS, e.g. read cookies
- relies on JS having full access to all resources on page
- exploits trust that user has for domain
- beware: can make CSRF through XSS, or XSS through CSRF ❗️
- reflected XSS: JS is in HTTP request and reflected back in HTTP response, e.g. in parameter, search query, invalid path of URL, etc.
- stored XSS: JS is stored on website, e.g. in database, third party library, browser extension, etc.
- exploit: inject JS into page, e.g. web page that accepts user input without sanitizing it



## Cross-Site Request Forgery (CSRF)

- attack that forces user of a website to execute unwanted request, e.g. delete resource
- relies on cookies being sent with every request
- exploits trust that domain has for user
- beware: can make CSRF through XSS, or XSS through CSRF ❗️
- exploit: inject URL with sideffect, e.g. link, image, hidden form, etc.



## Resources

- [OWASP - Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [OWASP - Cross Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf)
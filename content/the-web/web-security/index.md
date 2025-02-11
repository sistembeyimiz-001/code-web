---
title: Web security
author: vwkd
index: 14
tags:
  - the-web
---

- measures to make a website secure
- beware: most security measures are useless without HTTPS, since data can be manipulated in transit ❗️
- essential part of website not afterthought
- be proactive not reactive, assume you'll be hacked
- use threat model to determine individual security
- needs to continually evolve
- only as strong as weakest link



## Threat model

- assessment of threats
- profile of attackers
- desirable assets to attacker
- valuable assets to oneself
- attack vectors
- vulnerabilities



## Principles

- least privilege, e.g. APIs, system resources, database, VCS, allow list instead of deny list, private methods instead of public, etc.
- simpler is more secure, reduce complexity instead of adding, e.g. use built-in instead of custom functions, use proven libraries instead of reinventing the wheel, etc.
- never trust no one, e.g. user, own first-party apps, etc.
- expect the unexpected, edge cases, e.g. no input, too much input, special characters, submit form from another domain, URL manipulation, etc.
- in-depth, use multiple layers, not just perimeter, e.g. physical, technical, administrative, etc.
- obscurity, no information leakage, e.g. no file extension in URL, no server configuration in HTTP header, error messages, redacted WHOIS, no company information on social media or in forums, etc.



## Input

- never trust incoming data, all input is potentially dangerous, e.g. forms, cookies, query parameter, request body, etc.
- rate limit, HTTP status code 429 too many requests
- size limit, cut off large payloads
- validation: integer overflow due to large number, file type, etc.
- sanitization: language specific characters, sandbox uploaded files, filenames must contain only URL safe characters, use lowercase only to avoid issue of case in-/sensitivity, etc.



## Storage

- encrypt private data, need to store key safely



## Logging

- log errors, sensitive actions, suspicious activity
- log date/time, source
- don't log sensitive data, e.g. passwords
- don't leak implementation, e.g. error messages



## Social

- no sensitive information on public websites, e.g. social media, forums
- redacted WHOIS
- social engineering, e.g. execute files, plug in devices into ports, etc
- phishing, e.g. email, etc.



## HTTP response headers

- CSP, limit resources from own page, no inline scripts, etc.
- `Strict-Transport-Security` to mandate HTTPS
- `X-Frame-Options` to disable website being loaded in iframe, e.g. on malicious copy
- CORS, allows requests only from specific domains ???
    limit request origin for scripts to own website, e.g. not on evil website
    beware: anchor, images, etc. still can be loaded cross-origin, embedded on other websites
- cookies http-only, secure such that not transmitted over HTTP
- don't leak implementation, e.g. server configuration



## Routes

- limit HTTP methods, don't expose sensible information or side effects on unauthorised routes, e.g. read, write, delete
- limit URL manipulation, don't expose any API that can be misused, e.g. list of all usernames
- limit API call frequency, e.g. on input pages like login, to trusted clients, etc.
- don't leak implementation, e.g. file type in URL



## External resources

- passive content: can't interact with page, e.g. images
- don't allow passive content where tracking is unwanted
- active content: can interact with rest of page, e.g. script, AJAX
- don't allow active content where taking over entire page is unwanted
- use hash integrity verification on script tags
- load every resource over HTTPS, no mixed content



## Sensitive data

- input form with HTTPS action, also surrounding page HTTPS to prevent manipulation of form action attribute data
- use sandboxed iframe for sensitive input, e.g. payment data, login
- use third-party login / payment system, e.g. via Amazon
- don't use active content on page with sensitive data



## Resources

- OWASP
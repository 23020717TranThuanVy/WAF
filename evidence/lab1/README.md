# Lab 1 evidence

This folder contains the setup and runtime checks for the Nginx + ModSecurity + OWASP CRS gateway in front of Juice Shop.

- compose-check.txt and compose-resolved.yml: Compose validation and resolved service configuration.
- backend-check.json and waf-check.json: HTTP page checks for ports 3000 and 8080.
- assets-check.json: local JavaScript and stylesheet asset checks.
- readiness-checks.json: readiness summary.
- runtime.json and setup-check.json: running containers, active ModSecurity, and proxy configuration.
- waf-config.txt and nginx-runtime-excerpts.txt: Compose and active Nginx excerpts.
- waf-logs.txt: filtered log excerpts.
- web-browser.png and waf-browser.png: captured page screenshots. The WAF screenshot shows its page shell before product cards render; browser-check.json records separate product API checks.

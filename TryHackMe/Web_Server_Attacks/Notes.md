# Web Server Attacks - Notes

## Identifying Web Servers

Before testing a web application, identify the web server and framework in use. This helps determine possible misconfigurations and attack vectors. The `Server` header often reveals the web server software, while the `X-Powered-By` header commonly identifies frameworks such as Express.

```bash
curl -sI http://<target-ip>:PORT
```

---

## Apache

Apache commonly exposes version information through the `Server` header. Misconfigurations include enabled directory listing, publicly accessible `/server-status`, exposed backup files (`.bak`), and `.htpasswd` files. These can reveal sensitive information, configuration details, or credentials.

```bash
curl -sI http://<target-ip>:80
```

```bash
gobuster dir -u http://<target-ip>:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt
```

---

## Python HTTP Server

Python's built-in HTTP server is intended for development but is often mistakenly exposed in production. It serves every file in the current directory, including hidden files such as `.env`. If no `index.html` exists, directory listing is automatically enabled, making it easy to discover backup archives, source code, and configuration files.

```bash
curl http://<target-ip>:8000/
```

```bash
curl http://<target-ip>:8000/.env
```

---

## Node.js (Express)

Express applications usually reveal themselves through the `X-Powered-By` header. Poorly configured applications may expose verbose error messages, debug endpoints, API routes, environment variables, and static configuration files. These disclosures often reveal internal paths, database credentials, API endpoints, and application settings.

```bash
curl http://<target-ip>:3000/api/routes
```

```bash
curl http://<target-ip>:3000/api/debug/env
```

---

## Nginx

Nginx may disclose its version through the `Server` header. When the `autoindex` directive is enabled, directories can be browsed directly. The `/nginx_status` endpoint may also expose server statistics if it is publicly accessible. Misconfigured directories often contain deployment notes, configuration files, and backups.

```bash
curl -sI http://<target-ip>:8080
```

```bash
curl http://<target-ip>:8080/nginx_status
```

---

## Security Headers

Security headers strengthen browser-side security by protecting against attacks such as clickjacking, MIME sniffing, cross-site scripting, and information leakage. Important headers include `X-Frame-Options`, `X-Content-Type-Options`, `Content-Security-Policy`, `Referrer-Policy`, and `Strict-Transport-Security`. Missing these headers does not always indicate a vulnerability, but it reduces the application's overall security posture.

---

## Nikto

Nikto is an automated web server scanner used to detect common misconfigurations and known issues. It can identify missing security headers, exposed status pages, outdated software, backup files, default content, and directory indexing.

```bash
nikto -h http://<target-ip>:80
```

---

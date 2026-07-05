## Identifying Web Servers

Identified the web server software by analyzing HTTP response headers and default error pages.

### Server Response Headers

Used `curl` to retrieve HTTP response headers and identify the web server.

**Command**

```bash
curl -sI http://<target-ip>:<port>
```

**Observation**

- Identified Apache, Python HTTP Server, and Nginx through the `Server` header.
- Observed that Express does not expose a `Server` header by default.

**Screenshot**

![Server Headers](screenshots/server-header.png)

---

### X-Powered-By Header

Checked the `X-Powered-By` header to identify the application framework.

**Observation**

- Detected the **Express** framework using the `X-Powered-By` header.

**Screenshot**

![X-Powered-By](screenshots/x-powered-by.png)

---

### Default Error Pages

Requested a non-existent page to analyze the default 404 error page.

**Command**

```bash
curl -s http://<target-ip>:<port>/nonexistent-page
```

**Observation**

- Different web servers returned unique default error pages, which can help identify the underlying server software.

---

## Answers

**Q:** What value does the `Server` header return for the Python HTTP Server running on port **8000**?

**A:** `SimpleHTTP/0.6 Python/3.12.3`

**Q:** Which HTTP header reveals the application framework running on port **3000**?

**A:** `X-Powered-By`

**Q:** What web server software is running on port **8080**?

**A:** `nginx`

---
## Python HTTP Server

Investigated a Python HTTP Server to identify exposed files and insecure configurations.

### Directory Listing

Accessed the web server and observed that directory listing was enabled, revealing all publicly accessible files.

**Command**

```bash
curl -s http://<target-ip>:8000/
```

**Observation**

- Directory listing was enabled.
- Multiple files were publicly accessible.

**Screenshot**

![Directory Listing](screenshots/python-directory.png)

---

### Accessing Sensitive Files

Retrieved the exposed `.env` file containing application configuration.

**Command**

```bash
curl -s http://<target-ip>:8000/.env
```

**Observation**

- Discovered database credentials and application secrets.

**Screenshot**

![.env File](screenshots/python-env.png)

---

### Inspecting Backup Files

Downloaded and extracted a backup archive to inspect its contents.

**Commands**

```bash
curl -s http://<target-ip>:8000/backup.zip -o backup.zip

unzip backup.zip
```

**Observation**

- The backup archive contained sensitive files, including the lab flag.

**Screenshot**

![Backup Archive](screenshots/python-backup.png)

---

## Answers

**Q:** What database password is stored in the `.env` file?

**A:** `S3cur3DBPass!`

**Q:** What is the flag found inside the backup archive?

**A:** `THM{py_server_exposed}`

---
## Apache Web Server

Investigated an Apache web server to identify common misconfigurations and exposed resources.

### Version Disclosure

Retrieved the HTTP response headers to identify the Apache version.

**Command**

```bash
curl -sI http://<target-ip>:80
```

**Observation**

- Identified the web server as **Apache**.
- The `Server` header revealed the Apache version and operating system.

**Screenshot**

![Apache Header](screenshots/apache-header.png)

---

### Directory Listing

Accessed a directory with indexing enabled.

**Observation**

- Directory listing was enabled under `/files/`.
- Identified downloadable files that could expose sensitive information.

**Screenshot**

![Directory Listing](screenshots/apache-directory.png)

---

### Server Status

Accessed the Apache `server-status` page.

**Observation**

- The `mod_status` page exposed server statistics, active connections, and server information.

**Screenshot**

![Server Status](screenshots/server-status.png)

---

### Gobuster Enumeration

Used Gobuster to discover hidden files and directories.

**Command**

```bash
gobuster dir -u http://<target-ip>:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt -t 20
```

**Observation**

- Discovered `/backup.bak`.
- Retrieved the backup file, which contained sensitive configuration information and credentials.

**Screenshot**

![Gobuster Scan](screenshots/apache-gobuster.png)

---

## Answers

**Q:** What Apache module exposes real-time server statistics at `/server-status`?

**A:** `mod_status`

**Q:** What is the flag found in the `/files/` directory?

**A:** `THM{apache_dir_listing}`

---
## Node.js (Express)

Analyzed a Node.js Express application to identify exposed debug information, API routes, environment variables, and static files.

### Framework Fingerprinting

Checked the HTTP response headers to identify the framework.

**Command**

```bash
curl -sI http://<target-ip>:3000
```

**Observation**

- Identified the application framework as **Express** using the `X-Powered-By` header.

**Screenshot**

![Express Header](screenshots/express-header.png)

---

### Application Information

Retrieved the application status and version.

**Command**

```bash
curl -s http://<target-ip>:3000
```

**Observation**

- Retrieved the application name and version.

**Screenshot**

![Application Version](screenshots/app-version.png)

---

### Verbose Error Messages

Triggered an API error to inspect the response.

**Command**

```bash
curl -s http://<target-ip>:3000/api/users
```

**Observation**

- The response exposed stack traces, internal file paths, and database query information.

**Screenshot**

![Verbose Error](screenshots/verbose-error.png)

---

### Route Enumeration

Enumerated available API endpoints.

**Command**

```bash
curl -s http://<target-ip>:3000/api/routes
```

**Observation**

- Discovered hidden API routes including debug endpoints.

**Screenshot**

![Routes](screenshots/routes.png)

---

### Environment Variables

Checked the debug endpoint for sensitive information.

**Command**

```bash
curl -s http://<target-ip>:3000/api/debug/env
```

**Observation**

- Exposed environment variables including `NODE_ENV` and database credentials.

**Screenshot**

![Environment Variables](screenshots/env.png)

---

### Static Files

Reviewed publicly accessible static files.

**Command**

```bash
curl -s http://<target-ip>:3000/static/config.js
```

**Observation**

- Found client-side configuration containing internal API details and the challenge flag.

**Screenshot**

![Static Config](screenshots/static-config.png)

---

## Answers

**Q:** What value does the `NODE_ENV` variable have in the debug endpoint response?

**A:** `development`

**Q:** What is the flag found in the static files served by the Node.js application?

**A:** `THM{express_debug_leaks}`

---
## Nginx

Investigated an Nginx web server to identify version disclosure, directory listing, and exposed status pages.

### Version Disclosure

Checked the HTTP response headers.

**Command**

```bash
curl -sI http://<target-ip>:8080
```

**Observation**

- Identified the web server as **Nginx**.
- The `Server` header revealed the Nginx version.

**Screenshot**

![Nginx Header](screenshots/nginx-header.png)

---

### Directory Listing

Accessed the `/files/` directory.

**Command**

```bash
curl -s http://<target-ip>:8080/files/
```

**Observation**

- Directory listing was enabled using the `autoindex` directive.
- Found several files, including backups and server configuration files.

**Screenshot**

![Directory Listing](screenshots/nginx-directory.png)

---

### Nginx Status

Checked the server status endpoint.

**Command**

```bash
curl -s http://<target-ip>:8080/nginx_status
```

**Observation**

- Retrieved server statistics such as active connections and total requests.

**Screenshot**

![Nginx Status](screenshots/nginx-status.png)

---

## Answers

**Q:** What Nginx directive enables directory listing in a location block?

**A:** `autoindex on`

**Q:** What URL path exposes Nginx connection statistics?

**A:** `/nginx_status`

**Q:** What is the flag found in the directory listing on port 8080?

**A:** `THM{nginx_autoindex}`

---
## Common Misconfigurations Across Web Servers

Identified common security misconfigurations shared across Apache, Python HTTP Server, Express, and Nginx.

### Security Headers

Checked whether common HTTP security headers were present.

**Command**

```bash
for port in 80 8000 3000 8080; do
echo "=== Port $port ==="
curl -sI http://<target-ip>:$port/ | grep -iE "x-frame-options|x-content-type|content-security-policy|strict-transport|referrer-policy"
done
```

**Observation**

- None of the web servers returned common security headers.
- Missing headers increase the risk of attacks such as clickjacking, MIME sniffing, and XSS.

**Screenshot**

![Security Headers](screenshots/security-headers.png)

---

### Nikto Scan

Scanned the Apache server for common vulnerabilities.

**Command**

```bash
nikto -h http://<target-ip>:80 -nointeractive
```

**Observation**

- Detected missing security headers.
- Found directory indexing enabled.
- Identified the exposed `/server-status` page.
- Reported other common web server misconfigurations.

**Screenshot**

![Nikto Scan](screenshots/nikto-scan.png)

---

## Answers

**Q:** Which security header, when missing, allows a page to be embedded in an iframe on another domain?

**A:** `X-Frame-Options`

**Q:** What finding text does Nikto report when it detects directory indexing on the `/files/` path?

**A:** `Directory indexing found`

---

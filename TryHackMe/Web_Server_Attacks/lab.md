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

### X-Powered-By Header

Checked the `X-Powered-By` header to identify the application framework.

**Observation**

- Detected the **Express** framework using the `X-Powered-By` header.
### Default Error Pages

Requested a non-existent page to analyze the default 404 error page.

**Command**

```bash
curl -s http://<target-ip>:<port>/nonexistent-page
```

**Observation**

- Different web servers returned unique default error pages, which can help identify the underlying server software.

## Answers

**Q:** What value does the `Server` header return for the Python HTTP Server running on port **8000**?

**A:** `SimpleHTTP/0.6 Python/3.12.3`

**Q:** Which HTTP header reveals the application framework running on port **3000**?

**A:** `X-Powered-By`

**Q:** What web server software is running on port **8080**?

**A:** `nginx`

-------------------------------------------------------------------------
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

### Accessing Sensitive Files

Retrieved the exposed `.env` file containing application configuration.

**Command**

```bash
curl -s http://<target-ip>:8000/.env
```

**Observation**

- Discovered database credentials and application secrets.


### Inspecting Backup Files

Downloaded and extracted a backup archive to inspect its contents.

**Commands**

```bash
curl -s http://<target-ip>:8000/backup.zip -o backup.zip

unzip backup.zip
```

**Observation**

- The backup archive contained sensitive files, including the lab flag.

## Answers

**Q:** What database password is stored in the `.env` file?

**A:** `S3cur3DBPass!`

**Q:** What is the flag found inside the backup archive?

**A:** `THM{py_server_exposed}`

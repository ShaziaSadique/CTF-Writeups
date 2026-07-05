## Manual Discovery – Common Files

### robots.txt

Accessed the `robots.txt` file to identify directories excluded from search engine indexing.

**Observation:**
- Discovered the `/staff-portal` directory.
- Although excluded from web crawlers, the directory remained directly accessible, demonstrating that `robots.txt` is not a security mechanism.

### sitemap.xml

Reviewed the `sitemap.xml` file to identify publicly listed pages and application endpoints.

**Observation:**
- Identified pages such as `/news` and `/contact`.
- Found additional paths including `/customers/login` and `/s3cr3t-area`.
- The presence of URLs with query parameters (e.g., `id=`) highlighted potential areas for further security testing.

----------------------------------------------------------------------------------------------------------------------------------------------------------------
## Manual Discovery – HTTP Headers

Used `curl` with the verbose (`-v`) option to inspect the HTTP response headers returned by the web server.

**Command**

```bash
curl http://<target-ip> -v
```

**Observation**

- Identified the web server as **Nginx**.
- Detected the `X-Powered-By` header, revealing the **THM Framework**.
- Found a custom `X-FLAG` header containing the lab flag.

## Framework Stack

Investigated the framework used by the application by reviewing the page source and following the documentation link.

**Observation**

- Identified the administration portal.
- Logged in using the default credentials provided in the lab (`admin/admin`).
- Retrieved the flag from the admin dashboard.

----------------------------------------------------------------------------------------------------------------------------------------------------------
## OSINT – Search Engines & Web Tools

Used OSINT (Open-Source Intelligence) techniques to gather publicly available information about a target website.

### Google Dorking

Performed advanced Google searches using search operators to identify publicly indexed resources.

**Examples**

- `site:` – Search within a specific website.
- `inurl:` – Find pages containing specific words in the URL.
- `filetype:` – Search for specific file types (e.g., PDF).
- `intitle:` – Search for keywords in page titles.

**Observation**

Google Dorking can reveal login pages, documents, admin panels, and other publicly indexed resources.

### Wappalyzer

Used the Wappalyzer browser extension to identify the technologies used by a website.

**Observation**

Wappalyzer detects technologies such as:
- Web frameworks
- CMS platforms
- Programming languages
- Web servers
- Analytics tools

---------------------------------------------------------------------------------------------------------------------------------------------------
## OSINT – Repositories & Archives

Used public repositories and web archives to gather information about the target.

### Wayback Machine

Reviewed archived versions of the website to identify removed pages, old login portals, and historical content.

### GitHub

Searched public GitHub repositories for exposed configuration files, credentials, API keys, and useful information in commit history.

### Amazon S3 Buckets

Checked for publicly accessible Amazon S3 buckets using common company naming patterns to identify exposed files or backups.


## Answers

**Q:** What is the website address for the Wayback Machine?

**A:** `https://web.archive.org/`

**Q:** What URL format do Amazon S3 buckets end in?

**A:** `.s3.amazonaws.com`
----------------------------------------------------------------------------------------------
## Automated Discovery – Gobuster Fundamentals

Used **Gobuster** to discover hidden directories and files on the target web server using a wordlist.

**Command**

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
```

**Observation**

- Discovered hidden directories such as `/assets`, `/private`, and `/monthly`.
- Found accessible pages including `/contact` and `/news`.
- Identified a log file: `/development.log`.
- Located `robots.txt` and `sitemap.xml`, providing additional reconnaissance information.
- 
## Answers

**Q:** What is the name of the directory beginning with `/mo` that was discovered?

**A:** `/monthly`

**Q:** What is the name of the log file that was discovered?

**A:** `/development.log`

----------------------------------------------------------------------------------------------------------------
## Automated Discovery – Subdomains & Virtual Hosts

Used **Gobuster** to enumerate subdomains and virtual hosts, helping identify additional applications and services associated with the target.

### DNS Enumeration

Performed DNS brute-forcing to discover subdomains using a wordlist.

**Command**

```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
```

**Observation**

Discovered several subdomains, including:

- `shop.example.thm`## Answers

**Q:** Apart from `dns` and `-w`, which shorthand flag is required for DNS mode?

**A:** `-d`

**Q:** How many virtual hosts on `acmeitsupport.thm` responded with **HTTP 200**?

**A:** `3`
- `www.shop.example.thm`
- `academy.example.thm`
- `primary.example.thm`

  
### Virtual Host Enumeration

Performed virtual host enumeration to identify hidden websites hosted on the same IP address.

**Command**

```bash
gobuster vhost -u http://<target-ip> --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```

**Observation**

- Identified virtual hosts responding with **HTTP 200 OK**.
- Virtual host enumeration helps discover hidden applications that are not publicly listed in DNS.


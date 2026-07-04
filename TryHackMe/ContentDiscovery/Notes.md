## Manual Discovery-Common Files
*Robots.txt*   
It is a file that tells search engines which parts of a site they can or cannot index. Sometimes admins list sensitive folders here (e.g., `/staff-portal`).  

*Sitemap.xml*   
It is a file that lists all the pages the site owner wants search engines to know about.  
It Can reveal hidden or sensitive endpoints (e.g., `/customers/login`, `/s3cr3t-area`).   
Also shows parameters like `id=` in URLs, which hint at possible input points to test. Useful for mapping the attack surface quickly during reconnaissance.  

-----------------------------------------------------------------------------------------------------------------------
## Manual Discovery - Headers & Framework Stack
*HTTP Headers*  
HTTP Headers are revealed when the web server responds to a request.  
It includes Server and X-Powered-By that expose the web server software and laguage/framework the application runs on.  

Explore the page source and copyright issues for the framework that is used.  

-----------------------------------------------------------------------------------------------------------------------
## OSINT - Search Engines & Web Tools
*OSINT*  
Open-Source Intelligence is the process of gathering publicly available information about a target without directly interacting with it.  
 `site:` | Search within a specific website
 `inurl:` | Find pages with a word in the URL
 `filetype:` | Search for specific file types 
 `intitle:` | Find pages with a word in the title
 `intext:` | Search page content 
 `cache:` | View Google's cached page 
Example:  
site:tryhackme.com filetype:pdf  

*Wappalyzer*   
It is a browser extension and online tool that helps to find a website's frameworks, CMS platforms, CDNs, etc   

-----------------------------------------------------------------------------------------------------------------------
## OSINT - Repositories & Archives
*Wayback Machine*   
It is used for finding pages that have been removed from the live site but may still be accessible.  
Search public GitHub repositories for exposed credentials, configuration files, and sensitive data. Review commit history, as deleted secrets may still exist in older commits.  

*S3 Bucket*  
Amazon Simple Storage Service is a cloud storage platform that many organisations use to host files and static website content.   

-----------------------------------------------------------------------------------------------------------------------
## Automated Discovery - Gobuster Fundamentals
*Gobuster*  
Gobuster is an open-source enumeration tool used to discover hidden directories, files, and web pages on a target website.  
It uses a wordlist to send requests to the web server and identifies accessible resources based on the server's responses.  

Basic Command  
gobuster dir -u http://<target-ip> -w /path/to/wordlist  
    
Useful Flags  

- `-t` : Number of threads
- `-w` : Wordlist
- `-o` : Save output
- `-x` : Search file extensions
- `-r` : Follow redirects

-----------------------------------------------------------------------------------------------------------------------
## Automated Discovery - Subdomains & Virtual Hosts
Subdomain enumeration identifies subdomains of a target domain that may host separate applications or services with different security configurations.  

Gobuster's `dns` mode brute-forces subdomains using a wordlist.  

Basic Command    
gobuster dns -d example.com -w /path/to/wordlist  

Useful Flags  

- `-d` : Target domain
- `-w` : Wordlist
- `-i` : Show IP addresses
- `-r` : Custom DNS resolver

*Virtual Host Enumeration*  

Virtual hosts allow multiple websites to run on the same IP address. Gobuster's vhost mode discovers hidden websites by testing different Host headers.  

Basic Command  
gobuster vhost -u http://<target-ip> --domain example.com -w /path/to/wordlist  

Subdomain: Exists as a DNS record (e.g., `blog.example.com`).  

Virtual Host: Hosted on the same IP and identified using the HTTP `Host` header.  

-----------------------------------------------------------------------------------------------------------------------

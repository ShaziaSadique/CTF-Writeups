## Manual Discovery-Common Files
## Robots.txt
- A file that tells search engines which parts of a site they can or cannot index.
- Sometimes admins list sensitive folders here (e.g., `/staff-portal`).

## Sitemap.xml
- A file that lists all the pages the site owner wants search engines to know about.
- Can reveal hidden or sensitive endpoints (e.g., `/customers/login`, `/s3cr3t-area`).
- Also shows parameters like `id=` in URLs, which hint at possible input points to test.
- Useful for mapping the attack surface quickly during reconnaissance.

-----------------------------------------------------------------------------------------------------------------------
## Manual Discovery - Headers & Framework Stack
## HTTP Headers
Http headers are revealed when the web server responds to a request
It includes Server and X-Powered-By that expose the web server software and laguage/framework the application runs on.

## Framework Stack
Explore the page source and copyright issues for the framework that is used.

-----------------------------------------------------------------------------------------------------------------------


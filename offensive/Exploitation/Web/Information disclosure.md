# Overview
Information disclosure may not be a huge vulnerability, but the information leaked can assest you in finding high severity vulnerabilities. Generally speaking, you want to give the website malformed requests and check the errors, thru this type of vulnerability you can gather more information about the website. Information disclosure can occur in multiple places listed below:
- <b>Files for web crawlers</b> - like `robots.txt` and `sitemap.xml` 'hidden directories' -
- <b>Directory listings</b> - dork for 'intitle: Index of' - config and backup files -
- <b>Developer comments</b> - very rare but may be useful -
- <b>Error messages</b> - might give the the sql query and other information about the backend -
- <b>Debugging data</b> - overly verbose error messages help you understand website logic -
- <b>Version control history</b> - .git directories hold valuable information about the source code-
- <b>Insecure configuration</b> - The application uses third party software configured improperly 
You want to induce the application to send error by changing parameter values/data types and checking the output.
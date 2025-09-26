# Things you should look for
## Tech Stack
The tech stack determines the wordlists you're going to use in content discovery phase. You can use [whatruns](https://www.whatruns.com/) and [wappalyzer](https://www.wappalyzer.com/) to identify the tech stack of the domain, then you can choose the associated wordlists.

## Config Files
Config files hold valuable information about the target. After identifying the tech stack, you can look for config files. for example in an `ASP.NET` app you want to look for `Appsettings.json`. You can find these files by bruteforce or by looking online in docs or help forms for that tech.

## Wordlist by source code
The [Source2URL](https://github.com/danielmiessler/Source2URL/blob/master/Source2URL) tool takes the source code of the application - if publicly available - and outputs URLs or endpoints.

## Dumped Images
You can search for dumped images of the target - especially if using docker - in [docker hub](https://hub.docker.com/) 

## Historical Wordlists
By historical I mean from the wayback machine, security trails and alien vault. One such tool is [gau](https://github.com/lc/gau), another tool is [waymore](github.com/xnl-h4ck3r/waymore) which does the same is `gau` but it downloads the pages and parses them for parameters and links, you can also view the pages for dev comments.

## 401 and 403
Eventually you'll encounter a `401` or a `403` status code on a page like `/admin`. In that case you should recursively bruteforce that path, because in some cases the authorization checks are applied to the first two pages (e.g. `/admin/dashboard`) and if you were able to find other pages after `/dashborad` you might bypass the auth checks. Also, every time you encounter a `403` or a `401` page you should check the [wayback archive](https://web.archive.org/)

## Mobile Endpoints
Try checking for mobile apps for the target, once you find and download the mobile app check that it communicates to an in scope domain - via burpsuite - and if so, you can extract endpoints from the apk of that application using [apkleaks](https://github.com/dwisiswant0/apkleaks) 
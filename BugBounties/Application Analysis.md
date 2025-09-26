# six important questions
There are important questions you should ask your self when approaching a target;

## 1- How does the app pass data?
Does the app pass data via parameters? (e.g. `/resource?param1=val1&param2=val2` ). Or does it pass the data via rest? (e.g. `/route/resource/sub-resource/..`). This information is important because if you didn't know how the app sends data to the backend, you wouldn't be able to do injection vulnerabilities for example, if the app talks in rest format and SQL injection would be like `/users/username/'` unlike a parameter type communication `/usernames?user='` .

## 2- How/Where does the app talk about users?
This question is split into 2 parts. How the app talks about users refers to the way the app references users, for example via `UID`, `emails`, `username`, `UUID`. Where the app talks about users references the places the app checks for user identification like `Cookies` and `API calls` . Understanding the way the app talks about users is crucial to finding several bug classes related to Access, Authorization, Logic, Information Disclosure.

## 3- Does the site have multi-tenancy or user levels?
This information dictates the type of vulnerabilities we'll search for. If the target has multiple user levels, we can search for `Broken Access Control`, where we can login as a low privilege user and try to access admin only functionality/information. You can use [Autorize](portswigger.net/bappstore/f9bbac8c4acf4aefa4d7dc92a991af2f) which is a tool that you give low privilege cookies and then browse the application is a high privilege user, the tool will then repeat the same requests but with the low privilege cookies. You can also not provide any cookies to test for unauthenticated access

## 4- Does the site have a unique threat model?
Alot of websites have a unique threat model, and by that I mean unique data (PII) that the target want to protect. An example of a normal PII is email addresses and credit card numbers, location etc... but some times the application has PII that is unique, like twitch has a streamer key, the streamer key is unique to twitch. 

## 5- Has there been past security research & vulns?
If you are approaching a target, it's worth taking a look at previous reported vulnerabilities. Some times the developers added a patch for that vulnerability and you may be able to bypass that patch using another method/technique for exploitation.

## 6- How does the app handle vulnerabilities
By this I mean look for documentation/help forms about the framework/tech stack in use and how it handles XSS,SQLi,Code injection etc... and if the framework has a pretty good handler for XSS you shouldn't prioritize it.

# Crawling
crawling is the act of parsing the HTML pages for information, weather it's links,emails,comments etc... you can use [hakrawler](https://github.com/hakluke/hakrawler) and [gospider](https://github.com/jaeles-project/gospider) 

# JavaScript Parsing
The tools mentioned in the Crawling section do parse the js files they find, but we also want to parse the js inline code found in html pages, we can use [xnlinkfinder](https://github.com/xnl-h4ck3r/xnLinkFinder) to achieve this.

# JavaScript (Libs/Dependencies)
You can use [retire.js](retirejs.github.io/retire.js/) to look for js libraries with vulnerable versions. I installed the Firefox extension

# Parameter Analysis
https://github.com/g0ldencybersec/sus_params/

# Heat Mapping
Refers to the areas or places in an application where things normally get bad, like file upload. This is a mind map from Jhaddix in [this talk](https://www.youtube.com/watch?v=FqnSAa2KmBI);
![[Pasted image 20250830172334.png]]










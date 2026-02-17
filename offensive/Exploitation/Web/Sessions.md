# Overview


# Session Hijacking
Session Hijacking occurs when the attacker obtains the authentication cookie of the victim, the attacker can simply paste the cookie in the browser and should get access to the victim account.

# Session Fixation
Fixation means putting something in place, in this vulnerability the web application uses a value in a get parameter to generate a session token for the user. Let's take an example;
```text
http://fixation.com/login?redirect=/dashboard&token=secret_token
```
When you find a URL like that, the first thing that should come to mind is; is this `token` parameter is being stored as a cookie? in the case of this example when you check the cookie `PHPSESSID` you will find the value `secret_token`, you can then try to login and view the token again. If the token remains the same then there is a session fixation vulnerability. You can send someone a link to login like;
```text
http://fixation.com/login?redirect=/dashboard&token=attacker_token
```
And when they login there `PHPSESSID` token will be `attacker_token` which gives you the ability to add that cookie to your browser and hijack their session. 
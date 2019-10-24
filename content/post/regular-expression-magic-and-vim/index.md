---
date: "2013-08-20 22:19:35"
title: "Regular Expression Magic and ViM"
---
Every now and then I have an editing task for which Eclipse is just not up to the job.
Usually it involves making a lot of changes all at once.
(I realize that Eclipse has regex find/replace, but I feel much better working in ViM in these cases.)

Here is an example:  I have a block of Java enum code that I want to split out into an enum and two maps.

Here is the regular expression I use to parse the enum declaration:

``` regular-expression
/\v\s*([A-Z]{2,})\((\s*[^,]*),\s*("[^"]*")\s*,\s*("[^"]*")\s*\)\s*([,;])\s*
```

Here is an example of an enum declaration that matches this regular expression:

``` javascript
FACEBOOK(Constants.FACEBOOK, "fbconnect://success", "fbconnect://success?error_reason"),
```

I then run three s// commands on the input:

``` regular-expression
%s//\1(\2)\5/g
%s//callbackMap.put(Provider.\1, \3);/g
%s//cancelMap.put(Provider.\1, \4);/g
```

After each command I copy the buffer and paste it back into Eclipse, then hit `u` to undo my changes so that I can run the next command on the original buffer contents.

Here's what I get (these aren't meant to be used in isolation like this, of course):

``` javascript
FACEBOOK(Constants.FACEBOOK),
callbackMap.put(Provider.FACEBOOK, "fbconnect://success");
cancelMap.put(Provider.FACEBOOK, "fbconnect://success?error_reason");
```

Input:

``` javascript
FACEBOOK(Constants.FACEBOOK, "fbconnect://success", "fbconnect://success?error_reason"),
TWITTER( Constants.TWITTER, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do", "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do?denied"),
LINKEDIN(Constants.LINKEDIN, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do", "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do?oauth_problem"),
MYSPACE( Constants.MYSPACE, "http://socialauth.in", "http://socialauth.in/?oauth_problem"),
RUNKEEPER( Constants.RUNKEEPER, "http://socialauth.in/socialauthdemo/socialauthSuccessAction.do", "http://socialauth.in/socialauthdemo/socialauthSuccessAction.do/?error"),
YAHOO(Constants.YAHOO, "http://socialauth.in/socialauthdemo", "http://socialauth.in/socialauthdemo/?oauth_problem"),
FOURSQUARE( Constants.FOURSQUARE, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do", "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem"),
GOOGLE( Constants.GOOGLE, "http://socialauth.in/socialauthdemo", "http://socialauth.in/socialauthdemo/?oauth_problem"),
YAMMER(Constants.YAMMER, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do", "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem"),
SALESFORCE( Constants.SALESFORCE, "https://socialauth.in:8443/socialauthdemo/socialAuthSuccessAction.do", "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem"),
GOOGLEPLUS( Constants.GOOGLE_PLUS, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do", "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem"),
EMAIL(SHARE_MAIL, "", ""),
MMS(SHARE_MMS, "", "");
```


Output #1:

``` javascript
FACEBOOK(Constants.FACEBOOK),
TWITTER( Constants.TWITTER),
LINKEDIN(Constants.LINKEDIN),
MYSPACE( Constants.MYSPACE),
RUNKEEPER( Constants.RUNKEEPER),
YAHOO(Constants.YAHOO),
FOURSQUARE( Constants.FOURSQUARE),
GOOGLE( Constants.GOOGLE),
YAMMER(Constants.YAMMER),
SALESFORCE( Constants.SALESFORCE),
GOOGLEPLUS( Constants.GOOGLE_PLUS),
EMAIL(SHARE_MAIL),
MMS(SHARE_MMS);
```


Output #2:

``` javascript
callbackMap.put(Provider.FACEBOOK, "fbconnect://success");
callbackMap.put(Provider.TWITTER, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do");
callbackMap.put(Provider.LINKEDIN, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do");
callbackMap.put(Provider.MYSPACE, "http://socialauth.in");
callbackMap.put(Provider.RUNKEEPER, "http://socialauth.in/socialauthdemo/socialauthSuccessAction.do");
callbackMap.put(Provider.YAHOO, "http://socialauth.in/socialauthdemo");
callbackMap.put(Provider.FOURSQUARE, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do");
callbackMap.put(Provider.GOOGLE, "http://socialauth.in/socialauthdemo");
callbackMap.put(Provider.YAMMER, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do");
callbackMap.put(Provider.SALESFORCE, "https://socialauth.in:8443/socialauthdemo/socialAuthSuccessAction.do");
callbackMap.put(Provider.GOOGLEPLUS, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do");
callbackMap.put(Provider.EMAIL, "");
callbackMap.put(Provider.MMS, "");
```

Output #3:

``` javascript
cancelMap.put(Provider.FACEBOOK, "fbconnect://success?error_reason");
cancelMap.put(Provider.TWITTER, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do?denied");
cancelMap.put(Provider.LINKEDIN, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do?oauth_problem");
cancelMap.put(Provider.MYSPACE, "http://socialauth.in/?oauth_problem");
cancelMap.put(Provider.RUNKEEPER, "http://socialauth.in/socialauthdemo/socialauthSuccessAction.do/?error");
cancelMap.put(Provider.YAHOO, "http://socialauth.in/socialauthdemo/?oauth_problem");
cancelMap.put(Provider.FOURSQUARE, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem");
cancelMap.put(Provider.GOOGLE, "http://socialauth.in/socialauthdemo/?oauth_problem");
cancelMap.put(Provider.YAMMER, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem");
cancelMap.put(Provider.SALESFORCE, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem");
cancelMap.put(Provider.GOOGLEPLUS, "http://socialauth.in/socialauthdemo/socialAuthSuccessAction.do/?oauth_problem");
cancelMap.put(Provider.EMAIL, "");
cancelMap.put(Provider.MMS, "");
```

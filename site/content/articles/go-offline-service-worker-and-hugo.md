---
title: Go offline! Service Worker and Hugo
date: 2017-10-19T17:46:34.206Z
description: >-
  After mobile first, offline first and progressive web apps (PWA) are the
  current trend at the moment. Service Workers are crucial for both of them. A
  service worker is basically a script acting like a proxy between the browser
  and the network. You will find a simple example how to install a service
  worker for your Hugo satic site to make it blazing fast.
image: null
---
What is this all about?

If you are completely new to Service Workers and wondering what this is all about. Please have a look at the following resources:

Your First Progressive Web App at Google Developers
Service Worker API at MDN Mozilla Developer Network
Service Worker Revolution at Ponyfoo
Everything you need to know to create offline-first web apps. at Github (pazguille/offlinefirst)
After you did all these readings - or at least understood what this is all about. This is what we will create within the next couple of lines:

Installing a Service Worker from an Example in Hugo
Deliver a custom offline page if a user has no network and the page is not in cache
Deliver a custom 404 page if a user access lands requests a page with Status Code >= 400
Add a manifest.json for mobile homescreen appearance configuration like splash-screens, orientation lock or background tasks.
Prerequisites

create an offline-Page

Make sure, that you create a custom offline page for your users when they are offline.

As an example you can create the following files:
```
|-- content
|  |-- offline.md
|-- layouts
|  |-- offline/single.html
```
content\/offline.md

```
+++
date = "2016-10-16T19:28:41+02:00"
draft = false
title = "Oops, you are offline."
type = "offline"
+++

You should try to find some internet connection to browse here.
```
layouts\/offline\/single.html

```
<html><head><title>{{ .Title }}</title></head><body><h1>{{ .Title }}</h1>
    {{ .Content }}
 </body></html>
```

_Truly a very minimalistic example._You can for sure create your own offline page with whatever content you like :-\)

From the example we now have a page called`offline/index.html`generated. Check.

### create a custom 404 page {#create-a-custom-404-page}

If your project does not already have a custom 404 page, you can read in the[Hugo Docs how to create a custom 404 page](https://gohugo.io/templates/404/), or just follow the basic instructions below:

For this you will need the following files:

```
|-- content
|  |-- 404.md
|-- layouts
|  |-- 404.html
```

content\/404.md

```
+++
date = "2016-10-16T19:28:41+02:00"
draft = false
title = "Darn... Page not found."
+++

You should look somewhere else.
```

layouts\/404.html

```
<html><head><title>{{ .Title }}</title></head><body><h1>{{ .Title }}</h1>
    {{ .Content }}
 </body></html>
```

### Create Website \/ App Icons {#create-website-app-icons}

App Icons are basically favicons shown up in the splash screen when the website gets loaded from the Homescreen.

Recommended sizes are:

* 128px × 128px
* 144px × 144px
* 152px × 152px
* 192px × 192px
* 256px × 256px

You can use a favicon generator like[favicomatic.com](http://www.favicomatic.com/)to generate them quickly.

Then put the png’s in your`/static`folder. For example:

```
|-- static
|  |-- appicons
|  |  |-- icon-128x128.png
|  |  |-- icon-144x144.png
|  |  |-- icon-152x152.png
|  |  |-- icon-192x192.png
|  |  |-- icon-256x256.png

```

---

## Install the manifest.json {#install-the-manifest-json}

Now the real work comes in place where we create and setup the`manifest.json`file into static.

For this we will use a pre-made[example of a manifest.json](https://github.com/wildhaber/offline-first-sw/blob/master/manifest.json)from the`offline-first-sw`-Repo.

Put this file into the`static/`-folder as well. Please note that it needs to be on the root like:

```
|-- static
|  |-- manifest.json

```

You can do this copy paste work by hand or if you are on a Linux\/Unix environment, use the following command:

```
# assumed you are at your hugo root directory
cd static
wget https://raw.githubusercontent.com/wildhaber/offline-first-sw/master/manifest.js

```

Then you should have a file like this in your static-folder:

```
{"name":"<your-apps-name>","short_name":"<your-apps-shortname>","icons":[{"src":"/img/icons/logo-128x128.png","sizes":"128x128","type":"image/png"},{"src":"/img/icons/logo-144x144.png","sizes":"144x144","type":"image/png"},{"src":"/img/icons/logo-152x152.png","sizes":"152x152","type":"image/png"},{"src":"/img/icons/logo-192x192.png","sizes":"192x192","type":"image/png"},{"src":"/img/icons/logo-256x256.png","sizes":"256x256","type":"image/png"}],"start_url":"/index.html","display":"standalone","orientation":"portrait","background_color":"#000000","theme_color":"#000000"}
```

Adjust the values as you like.

### Link the manifest.json in your layout {#link-the-manifest-json-in-your-layout}

That the browser finds your`manifest.json`you need to add the following snippet within the`<head>`-Section of your templates:

```
<link rel="manifest" href="/manifest.json">
```

---

## Install the Service Worker {#install-the-service-worker}

For this we will use also use the pre-made[Service Worker Example](https://github.com/wildhaber/offline-first-sw/blob/master/sw.js)from the[`offline-first-sw`](https://github.com/wildhaber/offline-first-sw)-Repo.

Put this file \(`sw.js`\) into the`static/`-folder as well. Please note that it needs to be on the root like:

```
|-- static
|  |-- sw.js

```

You can do this copy paste work by hand or if you are on a Linux\/Unix environment, use the following command:

```
# assumed you are at your hugo root directory
cd static
wget https://raw.githubusercontent.com/wildhaber/offline-first-sw/master/sw.js

```

Then you should have a file like this in your static-folder:

```
const CACHE_VERSION =1;const BASE_CACHE_FILES =['/style.css','/script.js','/search.json','/manifest.json','/favicon.png',];const OFFLINE_CACHE_FILES =['/style.css','/script.js','/offline/index.html',];const NOT_FOUND_CACHE_FILES =['/style.css','/script.js','/404.html',];const OFFLINE_PAGE ='/offline/index.html';const NOT_FOUND_PAGE ='/404.html';const CACHE_VERSIONS ={
    assets:'assets-v'+ CACHE_VERSION,
    content:'content-v'+ CACHE_VERSION,
    offline:'offline-v'+ CACHE_VERSION,
    notFound:'404-v'+ CACHE_VERSION,};// Define MAX_TTL's in SECONDS for specific file extensionsconst MAX_TTL ={'/':3600,
    html:3600,
    json:86400,
    js:86400,
    css:86400,};const CACHE_BLACKLIST =[//(str) => {//    return !str.startsWith('http://localhost') && !str.startsWith('https://gohugohq.com');//},];const SUPPORTED_METHODS =['GET',];/**
 * isBlackListed
 * @param {string} url
 * @returns {boolean}
 */functionisBlacklisted(url){return(CACHE_BLACKLIST.length >0)?!CACHE_BLACKLIST.filter((rule)=>{if(typeof rule ==='function'){return!rule(url);}else{returnfalse;}}).length :false}/**
 * getFileExtension
 * @param {string} url
 * @returns {string}
 */functiongetFileExtension(url){let extension = url.split('.').reverse()[0].split('?')[0];return(extension.endsWith('/'))?'/': extension;}/**
 * getTTL
 * @param {string} url
 */functiongetTTL(url){if(typeof url ==='string'){let extension =getFileExtension(url);if(typeof MAX_TTL[extension]==='number'){return MAX_TTL[extension];}else{returnnull;}}else{returnnull;}}/**
 * installServiceWorker
 * @returns {Promise}
 */functioninstallServiceWorker(){return Promise.all([
            caches.open(CACHE_VERSIONS.assets).then((cache)=>{return cache.addAll(BASE_CACHE_FILES);}),
            caches.open(CACHE_VERSIONS.offline).then((cache)=>{return cache.addAll(OFFLINE_CACHE_FILES);}),
            caches.open(CACHE_VERSIONS.notFound).then((cache)=>{return cache.addAll(NOT_FOUND_CACHE_FILES);})]);}/**
 * cleanupLegacyCache
 * @returns {Promise}
 */functioncleanupLegacyCache(){let currentCaches = Object.keys(CACHE_VERSIONS).map((key)=>{return CACHE_VERSIONS[key];});returnnewPromise((resolve, reject)=>{

            caches.keys().then((keys)=>{return legacyKeys = keys.filter((key)=>{return!~currentCaches.indexOf(key);});}).then((legacy)=>{if(legacy.length){
                            Promise.all(
                                legacy.map((legacyKey)=>{return caches.delete(legacyKey)})).then(()=>{resolve()}).catch((err)=>{reject(err);});}else{resolve();}}).catch(()=>{reject();});});}


self.addEventListener('install', event =>{
        event.waitUntil(installServiceWorker());});// The activate handler takes care of cleaning up old caches.
self.addEventListener('activate', event =>{
        event.waitUntil(
            Promise.all([cleanupLegacyCache(),]).catch((err)=>{
                        event.skipWaiting();}));});

self.addEventListener('fetch', event =>{

        event.respondWith(
            caches.open(CACHE_VERSIONS.content).then((cache)=>{return cache.match(event.request).then((response)=>{if(response){let headers = response.headers.entries();let date =null;for(let pair of headers){if(pair[0]==='date'){
                                                date =newDate(pair[1]);}}if(date){let age =parseInt((newDate().getTime()- date.getTime())/1000);let ttl =getTTL(event.request.url);if(ttl &amp;&amp; age > ttl){returnnewPromise((resolve)=>{returnfetch(event.request).then((updatedResponse)=>{if(updatedResponse){
                                                                        cache.put(event.request, updatedResponse.clone());resolve(updatedResponse);}else{resolve(response)}}).catch(()=>{resolve(response);});}).catch((err)=>{return response;});}else{return response;}}else{return response;}}else{returnnull;}}).then((response)=>{if(response){return response;}else{returnfetch(event.request).then((response)=>{if(response.status <400){if(~SUPPORTED_METHODS.indexOf(event.request.method)&amp;&amp;!isBlacklisted(event.request.url)){
                                                            cache.put(event.request, response.clone());}return response;}else{return caches.open(CACHE_VERSIONS.notFound).then((cache)=>{return cache.match(NOT_FOUND_PAGE);})}}).then((response)=>{if(response){return response;}}).catch(()=>{return caches.open(CACHE_VERSIONS.offline).then((offlineCache)=>{return offlineCache.match(OFFLINE_PAGE)})});}}).catch((error)=>{
                                    console.error('  Error in fetch handler:', error);throw error;});}));});
```

Now you can configure how you want the Service Worker behaves:

#### BASE\_CACHE\_FILES`{ array }` {#base-cache-files-array}

```
const BASE_CACHE_FILES =['/style.css','/script.js','/search.json','/manifest.json','/favicon.png',];
```

Define files that in this list which always needs to be cached from the beginning.

#### OFFLINE\_CACHE\_FILES`{ array }` {#offline-cache-files-array}

```
const OFFLINE_CACHE_FILES =['/style.css','/script.js','/offline/index.html',];
```

Define files necessary for your offline page.

#### NOT\_FOUND\_CACHE\_FILES`{ array }` {#not-found-cache-files-array}

```
const NOT_FOUND_CACHE_FILES =['/style.css','/script.js','/404.html',];
```

Define files necessary for your 404 page.

#### OFFLINE\_PAGE`{ string }` {#offline-page-string}

```
const OFFLINE_PAGE ='/offline/index.html';
```

Deliver this page when the user is offline and the page is not cached already.

#### NOT\_FOUND\_PAGE`{ string }` {#not-found-page-string}

```
const NOT_FOUND_PAGE ='/404.html';
```

Deliver this page when the user lands on a page with`Status-Code >= 400`.

#### MAX\_TTL`{ object }` {#max-ttl-object}

```
const MAX_TTL ={'/':3600,
    html:3600,
    json:86400,
    js:86400,
    css:86400,};
```

This is a key-value mapping file extensions with a certain maximal time-to-live \(in secondsnot milliseconds\). This is how long the chache will be active until the page is getting refreshed from the network.

Not specified extensions with stay cached until a SW-Update.

```
// 60 = 1 minute// 3600 = 1 hour// 86400 = 1 day// 604800 = 1 week// 2592000 = 30 days (~ 1 month)// 31536000 = 1 year
```

#### CACHE\_BLACKLIST`{ array}` {#cache-blacklist-array}

```
const CACHE_BLACKLIST =[(str)=>{// str = URL of the resource// apply this rule when you do not want to cache external filesreturn!str.startsWith('https://yourwebsite.tld');},];
```

Adjust these parameters as they fit for your website \/ app.

### Register the Service Worker {#register-the-service-worker}

Add the following script at the end of the`<body>`in your content or put it in your custom javascript files:

```
<script>
    if('serviceWorker' in navigator) {
        navigator.serviceWorker
            .register('/sw.js', { scope: '/' })
            .then(function(registration) {
                console.log('Service Worker Registered');
            });

        navigator.serviceWorker
            .ready
            .then(function(registration) {
                console.log('Service Worker Ready');
            });
    }
</script>
```

This will register, install and activate your service worker.

Now you are done with all necessary steps. Enjoy a blazing fast Hugo page now :-\)

---

## Debugging your Service Worker {#debugging-your-service-worker}

Debugging a Service Worker in Google Chrome you can simply open the Console and go to the tab`Application`. There you find your registered service workers and caches.

Read more about[Debugging Service Workers](https://developers.google.com/web/fundamentals/getting-started/codelabs/debugging-service-workers/)at Google Developers.

If your preferred Browser is Firefox read more about[Debugging Service Workers and Push with Firefox Devtools](https://hacks.mozilla.org/2016/03/debugging-service-workers-and-push-with-firefox-devtools/)at hacks.mozilla.org.




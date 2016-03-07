---
layout: post
title:  "Push manifest and HTTP2"
date:   2016-03-05 08:20:21 +0000
author: slemgrim
summary: "With http2 a connection can serve multiple requests simultaneously. 
         A connection can also be used for a server to **push** a resource to a client."
---

- is not a generic push mechanism like websockets 
- it's designed for optimisation of http2 conversations
- server can guess associated resources for a given resource

With http2 a connection can serve multiple requests simultaneously. 
A connection can also be used for a server to **push** a resource to a client.

In the past we inlined and/or concatenated or assets to reduce the number of requests in order to increase the performance.
Http2 push replaces this at the protocol level. Yes this means no more inlining and concatenation, in fact this practice is harmful.

So is it ok to have multiple **script** and **link** tags in our documents? I know, we spent the last years to avoid this, but
now this comes with a couple of benefits:

- browser cache on module level
- simpler build tasks
- todo??

no more  spriting, inlining, domain sharding and concatenation. 
 
### http1.1
- client has to wait for first resource to know which resources he has to request next
- client can only download 6 resources at a time (TODO browser limit? server limit? why?)
- therefore client has to make many round trips until he has all resources needed to render a page

### http2

- server tells the client that he also pushes associated resources with the requested resource (e.g. index.html)
- client accepts the pushes or ignores them without extra round trips 
- application has to decide which resources are associated 

## What's a push?

You request **index.html** which requires app.js, app.css and a bunch of images to be fully rendered, so the server 
pushes these files in one go with the requests index.html.

Currently there is no way for the browser to tell the server which files are already in its cache. 
The browser can ignore the pushed files, but they are sent already.

This can be a huge benefit but can also lead to a waste of bandwidth if we get a lot of files we don't need.

## What's a push manifest?
   
*A manifest is not required by the HTTP2 protocol but is useful for telling your
server what resources to push with your main page.*

The push_manifest.json usually lies in the top level directory of your app and can 
contain push information for a single document or multiple documents.

Every associated resource has a type and a weight and is in the format *<URL>: <PUSH_PROPERTIES>*

``` json
{
    "/css/main.css": {
        "type": "style",
        "weight": 1
    },
} 
```

## type property

## weight property

Weight in HTTP/2 is a value used for distributing bandwidth between the streams. 
In case of CSS or JavaScript files, what you should do is to assign bandwidth exclusively to those files     
    
Todo: what is type? Why is it necessary?
Todo: what is weight? Why is it necessary? 

At the moment of writing no browser implements handling of the weight property. 

### single file manifest

In the manifest for a single ressource each key is the url to an associated file with its properties. 

    {
      "/css/main.css": {
        "type": "style",
        "weight": 1
      }, 
      "/js/main.js": {
        "type": "script",
        "weight": 1
      },
      ...
    }
    


### multi-file manifest

In the manifest for multiple files each key represents a document and its sub objects are associated resources

    {
      "index.html": {
        "/css/main.css": {
          "type": "style",
          "weight": 1
        },
        "/css/social.css": {
          "type": "style",
          "weight": 1
        },
        ...
      },
      "post.html": {
        "/css/main.css": {
          "type": "style",
          "weight": 1
        },
        ...
      }
    }
    
    
todo: weight based (chrome) vs. dependency based (firefox) approach    

## what does this mean to bundling and tooling like webpack?

## fallbacks


## Todo
https://webtide.com/http2-push-with-experimental-servlet-api/
https://github.com/GoogleChrome/http2-push-manifest
https://httpwg.github.io/specs/rfc7540.html#rfc.section.5.3.2
https://h2o.examp1e.net/configure/file_directives.html#file.mime.addtypes
http://blog.kazuhooku.com/2015/06/http2-and-h2o-improves-user-experience.html
https://w3c.github.io/preload/
https://github.com/GoogleChrome/http2-push-manifest/issues/1#issuecomment-150063151
https://http2.github.io/faq/

## Resources
https://rmurphey.com/blog/2015/11/25/building-for-http2
---
description: This page describes how the HTTP profile works and configuration options
---

# dynamicHTTP

## Overview

This profile uses HTTP Get and Post messages to communicate with the main Mythic server. Unlike the default HTTP though, this profile allows a lot of customization from both the client and server. There are two pieces to this as with most C2 profiles - Server side code and Agent side code. In general, the flow looks like:

![](<../.gitbook/assets/Screen Shot 2020-08-11 at 9.16.56 PM.png>)

## Agent Configuration

From the UI perspective, there are only two parameters - a JSON configuration and the auto-populated operation specific AES key. For the JSON configuration, there is a generic structure:

```
{
  "GET": {
    "ServerBody": [
    ],
    "ServerHeaders": {
      "Server": "NetDNA-cache/2.2",
    },
    "ServerCookies": {},
    "AgentMessage": []
  },
  "POST": {
    "ServerBody": [],
    "ServerCookies": {},
    "ServerHeaders": {
      "Server": "NetDNA-cache/2.2",
    },
    "AgentMessage": []
  },
  "jitter": 50,
  "interval": 10,
  "chunk_size": 5120000,
  "key_exchange": true,
  "kill_date": "2019-12-31"
}
```

There are some important pieces to notice here:

* `GET` - this block defines what "GET" messages look like. This section is used for requesting tasking.
  * `ServerBody` - this block defines what the body of the server's response will look like
  * `ServerHeaders` - this block defines what the server's headers will look like
  * `ServerCookies` - this block defines what the server's cookies will look like
  * `AgentMessage` - This block defines the different forms of agent messages for doing GET requests. This defines what query parameters are used, what cookies, headers, URLs, etc.The format here is generally the same as the "POST" messages and will be described in the next section.
* `POST` - this block defines what "POST" messages look like
  * `ServerBody` - this block defines what the body of the server's response will look like
  * `ServerHeaders` - this block defines what the server's headers will look like
  * `ServerCookies` - this block defines what the server's cookies will look like
  * `AgentMessage` - This block defines the different forms of agent messages for doing POST requests. This defines what query parameters are used, what cookies, headers, URLs, etc.
* `jitter` - this is the jitter percentage for callbacks
* `interval` - this is the interval in seconds between callbacks
* `chunk_size` - this is the chunk size used for uploading/downloading files
* `key_exchange` - this specifies if Apfell does an encrypted key exchange or just uses a static encryption key
* `kill_date` - this defines the date agents should stop checking in. This date is checked when the agent first starts and before each tasking request, if it is the specified date or later, the agent will automatically exit. This is in the YYYY-MM-DD format.

Now let's talk about the `AgentMessage` parameter. This is where you define all of the key components about how your GET and POST messages _from_ agent _to_ Mythic look, as well as indicating _where_ in those requests you want to put the actual message the agent is trying to send. The format in general looks like this:

```
{
  "urls": ["http://192.168.0.11:9001"],
  "uri": "/<test:string>",
  "urlFunctions": [
    {
      "name": "<test:string>",
      "value": "",
      "transforms": [
        {
          "function": "choose_random",
          "parameters": ["jquery-3.3.1.min.js","jquery-3.3.1.map"]
        }
      ]
    }
  ],
  "AgentHeaders": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Host": "code.jquery.com",
    "Referer": "http://code.jquery.com/",
    "Accept-Encoding": "gzip, deflate",
    "User-Agent": "Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko"
  },
  "QueryParameters": [
      {
        "name": "q",
        "value": "message",
        "transforms": [
        ]
      }
  ],
  "Cookies": [
     {
      "name": "__cfduid",
      "value": "",
      "transforms": [
        {
          "function": "random_alpha",
          "parameters": [30]
        },
        {
          "function": "base64",
          "parameters": []
        }
      ]
    }
  ],
  "Body": []
}
```

In the `AgentMessage` section for the "GET"s and "POST"s, you can have 1 or more instances of the above `AgentMessage` format (the above is an example of one instance). When the agent goes to make a GET or POST request, it randomly picks one of the formats listed and uses it. Let's look into what's actually being described in this `AgentMessage`:

* `urls`  - this is a list of URLs. One of these is randomly selected each time this overall `AgentMessage` is selected. This allows you to supply fallback mechanisms in case one IP or domain gets blocked.
* `uri` - This is the URI to be used at the end of each of the URLs specified. This can be a static value, like `/downloads.php`, or can be one that's changed for each request. For example, in the above scenario we supply `/<test:string>`. The meaning behind that format is explained in the [HTTP](dynamichttp.md#uri-formatting) for the server side configuration, but the point here to look at is the next piece - `urlFunctions`
* `urlFunctions` - This describes transforms for modifying the URI of the request. In the above example, we replace the `<test:string>` with a random selection from `["jquery-3.3.1.min.js", "jquery-3.3.1.map"]`.&#x20;
* `AgentHeaders` - This defines the different headers that the agent will set when making requests
  * Note: if you're doing domain fronting, this is where you'd set that value
* `QueryParameters` - This defines the query parameters (if any) that will be sent with the request. When doing transforms and dynamic modifications, there is a standard format that's described in the next section.

{% hint style="warning" %}
When doing query parameters, if you're going to do anything base64 encoded, make sure it's URL safe encoding. Specifically, `/`, `+`, `=`, and `\n` characters need to be URL encoded (i.e. with their %hexhex equivalents)&#x20;
{% endhint %}

* `Cookies` - This defines any cookies that are sent with the agent messages
* `Body` - This defines any modifications to the body of the request that should be made

### Transforms

The defining feature of the HTTP profile is being able to do transforms on the various elements of HTTP requests. What does this look like though?

```
"Cookies": [
     {
      "name": "__cfduid",
      "value": "",
      "transforms": [
        {
          "function": "random_alpha",
          "parameters": [30]
        },
        {
          "function": "base64",
          "parameters": []
        }
      ]
    }
  ]
```

These transforms have a few specific parts:

* `name` - this is the parameter name supplied in the request. For query parameters, this is the name in front of the `=` sign (ex:`/test.php?q=abc123`). For cookie parameters, this is the name of the cookie (ex: `q=abc123;id=abc1`).&#x20;
* `value` - this is the starting value before the transforms take place. You can set this to whatever you want, but if you set it to `message`, then the starting value for the transforms will be the message that the agent is trying to send to Apfell.
* `transforms` - this is a list of transforms that are executed. The value starts off as indicated in the `value` field, then each resulting value is passed on to the next parameter. In this case, the value starts as `""`, then gets 30 random alphabet letters, then those letters are base64 encoded.&#x20;
  * Transforms have 2 parameters: the name of the function to execute and an array of parameters to pass in to it.
  * The initial set of supported functions are:
    * base64
      * takes no parameters
    * prepend
      * takes a single parameter of the thing to prepend to the input value
    * append
      * takes a single parameter of the thing to append to the input value
    * random\_mixed
      * takes a single parameter of the number of elements to append to the input value. The elements are chosen from upper case, lower case, and numbers.
    * random\_number
      * takes a single parameter of the number of elements to append to the input value. The elements are chosen from numbers 0-9.
    * random\_alpha
      * takes a single parameter of the number of elements to append to the input value. The elements are chosen from upper case and lower case letters.
    * choose\_random
      * takes an array of elements to choose from
  * To add new transforms, a few things need to happen:
    * In the HTTP profile's `server` code, the function and a reverse of the function need to be added. The options need to be added to the `create_value` and `get_value` functions.
      * This allows the server to understand the new transforms
      * If you look in the `server` code, you'll see functions like `prepend` (which prepends the value) and `r_prepend` which does the reverse.
    * In the agent's HTTP profile code, the options for the functions need to also be added so that the agent understands the functions. Ideally, when you do this you add the new functions to all agents, otherwise you start to lose parity.

A final example of an agent configuration can be seen below:

```
{
  "GET": {
    "ServerBody": [
      {
        "function": "base64",
        "parameters": []
      },
      {
        "function": "prepend",
        "parameters": ["!function(e,t){\"use strict\";\"object\"==typeof module&&\"object\"==typeof module.exports?module.exports=e.document?t(e,!0):function(e){if(!e.document)throw new Error(\"jQuery requires a window with a document\");return t(e)}:t(e)}(\"undefined\"!=typeof window?window:this,function(e,t){\"use strict\";var n=[],r=e.document,i=Object.getPrototypeOf,o=n.slice,a=n.concat,s=n.push,u=n.indexOf,l={},c=l.toString,f=l.hasOwnProperty,p=f.toString,d=p.call(Object),h={},g=function e(t){return\"function\"==typeof t&&\"number\"!=typeof t.nodeType},y=function e(t){return null!=t&&t===t.window},v={type:!0,src:!0,noModule:!0};function m(e,t,n){var i,o=(t=t||r).createElement(\"script\");if(o.text=e,n)for(i in v)n[i]&&(o[i]=n[i]);t.head.appendChild(o).parentNode.removeChild(o)}function x(e){return null==e?e+\"\":\"object\"==typeof e||\"function\"==typeof e?l[c.call(e)]||\"object\":typeof e}var b=\"3.3.1\",w=function(e,t){return new w.fn.init(e,t)},T=/^[\\s\\uFEFF\\xA0]+|[\\s\\uFEFF\\xA0]+$/g;w.fn=w.prototype={jquery:\"3.3.1\",constructor:w,length:0,toArray:function(){return o.call(this)},get:function(e){return null==e?o.call(this):e<0?this[e+this.length]:this[e]},pushStack:function(e){var t=w.merge(this.constructor(),e);return t.prevObject=this,t},each:function(e){return w.each(this,e)},map:function(e){return this.pushStack(w.map(this,function(t,n){return e.call(t,n,t)}))},slice:function(){return this.pushStack(o.apply(this,arguments))},first:function(){return this.eq(0)},last:function(){return this.eq(-1)},eq:function(e){var t=this.length,n=+e+(e<0?t:0);return this.pushStack(n>=0&&n<t?[this[n]]:[])},end:function(){return this.prevObject||this.constructor()},push:s,sort:n.sort,splice:n.splice},w.extend=w.fn.extend=function(){var e,t,n,r,i,o,a=arguments[0]||{},s=1,u=arguments.length,l=!1;for(\"boolean\"==typeof a&&(l=a,a=arguments[s]||{},s++),\"object\"==typeof a||g(a)||(a={}),s===u&&(a=this,s--);s<u;s++)if(null!=(e=arguments[s]))for(t in e)n=a[t],a!==(r=e[t])&&(l&&r&&(w.isPlainObject(r)||(i=Array.isArray(r)))?(i?(i=!1,o=n&&Array.isArray(n)?n:[]):o=n&&w.isPlainObject(n)?n:{},a[t]=w.extend(l,o,r)):void 0!==r&&(a[t]=r));return a},w.extend({expando:\"jQuery\"+(\"3.3.1\"+Math.random()).replace(/\\D/g,\"\"),isReady:!0,error:function(e){throw new Error(e)},noop:function(){},isPlainObject:function(e){var t,n;return!(!e||\"[object Object]\"!==c.call(e))&&(!(t=i(e))||\"function\"==typeof(n=f.call(t,\"constructor\")&&t.constructor)&&p.call(n)===d)},isEmptyObject:function(e){var t;for(t in e)return!1;return!0},globalEval:function(e){m(e)},each:function(e,t){var n,r=0;if(C(e)){for(n=e.length;r<n;r++)if(!1===t.call(e[r],r,e[r]))break}else for(r in e)if(!1===t.call(e[r],r,e[r]))break;return e},trim:function(e){return null==e?\"\":(e+\"\").replace(T,\"\")},makeArray:function(e,t){var n=t||[];return null!=e&&(C(Object(e))?w.merge(n,\"string\"==typeof e?[e]:e):s.call(n,e)),n},inArray:function(e,t,n){return null==t?-1:u.call(t,e,n)},merge:function(e,t){for(var n=+t.length,r=0,i=e.length;r<n;r++)e[i++]=t[r];return e.length=i,e},grep:function(e,t,n){for(var r,i=[],o=0,a=e.length,s=!n;o<a;o++)(r=!t(e[o],o))!==s&&i.push(e[o]);return i},map:function(e,t,n){var r,i,o=0,s=[];if(C(e))for(r=e.length;o<r;o++)null!=(i=t(e[o],o,n))&&s.push(i);else for(o in e)null!=(i=t(e[o],o,n))&&s.push(i);return a.apply([],s)},guid:1,support:h}),\"function\"==typeof Symbol&&(w.fn[Symbol.iterator]=n[Symbol.iterator]),w.each(\"Boolean Number String Function Array Date RegExp Object Error Symbol\".split(\" \"),function(e,t){l[\"[object \"+t+\"]\"]=t.toLowerCase()});function C(e){var t=!!e&&\"length\"in e&&e.length,n=x(e);return!g(e)&&!y(e)&&(\"array\"===n||0===t||\"number\"==typeof t&&t>0&&t-1 in e)}var E=function(e){var t,n,r,i,o,a,s,u,l,c,f,p,d,h,g,y,v,m,x,b=\"sizzle\"+1*new Date,w=e.document,T=0,C=0,E=ae(),k=ae(),S=ae(),D=function(e,t){return e===t&&(f=!0),0},N={}.hasOwnProperty,A=[],j=A.pop,q=A.push,L=A.push,H=A.slice,O=function(e,t){for(var n=0,r=e.length;n<r;n++)if(e[n]===t)return n;return-1},P=\"\r"]
      },
      {
        "function": "prepend",
        "parameters": ["/*! jQuery v3.3.1 | (c) JS Foundation and other contributors | jquery.org/license */"]
      },
      {
        "function": "append",
        "parameters": ["\".(o=t.documentElement,Math.max(t.body[\"scroll\"+e],o[\"scroll\"+e],t.body[\"offset\"+e],o[\"offset\"+e],o[\"client\"+e])):void 0===i?w.css(t,n,s):w.style(t,n,i,s)},t,a?i:void 0,a)}})}),w.each(\"blur focus focusin focusout resize scroll click dblclick mousedown mouseup mousemove mouseover mouseout mouseenter mouseleave change select submit keydown keypress keyup contextmenu\".split(\" \"),function(e,t){w.fn[t]=function(e,n){return arguments.length>0?this.on(t,null,e,n):this.trigger(t)}}),w.fn.extend({hover:function(e,t){return this.mouseenter(e).mouseleave(t||e)}}),w.fn.extend({bind:function(e,t,n){return this.on(e,null,t,n)},unbind:function(e,t){return this.off(e,null,t)},delegate:function(e,t,n,r){return this.on(t,e,n,r)},undelegate:function(e,t,n){return 1===arguments.length?this.off(e,\"**\"):this.off(t,e||\"**\",n)}}),w.proxy=function(e,t){var n,r,i;if(\"string\"==typeof t&&(n=e[t],t=e,e=n),g(e))return r=o.call(arguments,2),i=function(){return e.apply(t||this,r.concat(o.call(arguments)))},i.guid=e.guid=e.guid||w.guid++,i},w.holdReady=function(e){e?w.readyWait++:w.ready(!0)},w.isArray=Array.isArray,w.parseJSON=JSON.parse,w.nodeName=N,w.isFunction=g,w.isWindow=y,w.camelCase=G,w.type=x,w.now=Date.now,w.isNumeric=function(e){var t=w.type(e);return(\"number\"===t||\"string\"===t)&&!isNaN(e-parseFloat(e))},\"function\"==typeof define&&define.amd&&define(\"jquery\",[],function(){return w});var Jt=e.jQuery,Kt=e.$;return w.noConflict=function(t){return e.$===w&&(e.$=Kt),t&&e.jQuery===w&&(e.jQuery=Jt),w},t||(e.jQuery=e.$=w),w});"]
      }
    ],
    "ServerHeaders": {
        "Server": "NetDNA-cache/2.2",
        "Cache-Control": "max-age=0, no-cache",
        "Pragma": "no-cache",
        "Connection": "keep-alive",
        "Content-Type": "application/javascript; charset=utf-8"
      },
    "ServerCookies": {},
    "AgentMessage": [{
      "urls": ["http://192.168.0.11:9001"],
      "uri": "/<test:string>",
      "urlFunctions": [
        {
          "name": "<test:string>",
          "value": "",
          "transforms": [
            {
              "function": "choose_random",
              "parameters": ["jquery-3.3.1.min.js","jquery-3.3.1.map"]
            }
          ]
        }
      ],
      "AgentHeaders": {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Host": "code.jquery.com",
        "Referer": "http://code.jquery.com/",
        "Accept-Encoding": "gzip, deflate",
        "User-Agent": "Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko"
      },
      "QueryParameters": [
          {
            "name": "q",
            "value": "message",
            "transforms": [
            ]
          }
      ],
      "Cookies": [
         {
          "name": "__cfduid",
          "value": "",
          "transforms": [
            {
              "function": "random_alpha",
              "parameters": [30]
            },
            {
              "function": "base64",
              "parameters": []
            }
          ]
        }
      ],
      "Body": []
    }]
  },
  "POST": {
    "ServerBody": [],
    "ServerCookies": {},
    "ServerHeaders": {
          "Server": "NetDNA-cache/2.2",
          "Cache-Control": "max-age=0, no-cache",
          "Pragma": "no-cache",
          "Connection": "keep-alive",
          "Content-Type": "application/javascript; charset=utf-8"
        },
    "AgentMessage": [{
      "urls": ["http://192.168.0.11:9001"],
      "uri": "/download.php",
      "urlFunctions": [],
      "QueryParameters": [
        {
          "name": "bob2",
          "value": "justforvalidation",
          "transforms": []
        }
      ],
      "AgentHeaders": {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Host": "code.jquery.com",
        "Referer": "http://code.jquery.com/",
        "Accept-Encoding": "gzip, deflate",
        "User-Agent": "Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko"
      },
      "Cookies": [
        {
          "name": "BobCookie",
          "value": "splat",
          "transforms": [
            {
              "function": "prepend",
              "parameters": [
                "splatity_"
              ]
            }
          ]
        }
      ],
      "Body": [
        {
          "function": "base64",
          "parameters": []
        },
        {
          "function": "prepend",
          "parameters": ["<html>"]
        },
          {
          "function": "append",
          "parameters": ["</html>"]
        }
      ]
    }]
  },
  "jitter": 50,
  "interval": 10,
  "chunk_size": 5120000,
  "key_exchange": true,
  "kill_date": "2020-01-20"
}
```

## Server Configuration

Like with all C2 profiles, the HTTP profile has its own docker container that handles connections. The purpose of this container is to accept connections from HTTP agents, undo all of the special configurations you specified for your agent to get the real message back out, then forward that message to the actual Mythic server. Upon getting a response from Mythic, the container performs more transforms on the message and sends it back to the Agent.

The docker container just abstracts all of the C2 features out from the actual Mythic server so that you're free to customize and configure the C2 as much as you want without having to actually adjust anything in the main server.&#x20;

There is only _ONE_ HTTP docker container per Mythic instance though, not one per operation. Because of this, the HTTP profile's server-side configuration will have to do that multiplexing for you. Below is an example of the setup:

![HTTP Comms from Agent to Server](<../.gitbook/assets/Screen Shot 2020-01-02 at 1.12.01 PM.png>)

Let's look into what this sort of configuration entails. We already discussed the agent side configuration above, so now let's look into what's going on in the HTTP C2 Docker container. The Server configuration has the following general format:

```
{
  "instances": [
    {
      "GET": {
    },
      "POST": {
    },
    "no_match": {
      "action": "return_file",
      "redirect": "http://example.com",
      "proxy_get": {
        "url": "https://www.google.com",
        "status": 200
      },
      "proxy_post": {
        "url": "https://www.example.com",
        "status": 200
      },
      "return_file": {
        "name": "fake.html",
        "status": 404
      }
    },
    "port": 9001,
    "key_path": "",
    "cert_path": "",
    "debug": true
    }
  ],
}
```

There are a couple key things to notice here:

* `instances` - You generally have one instance per operation, but there's no strict limit there. This is an array of configurations, so when the Docker server starts up, it loops through these instances and starts each one
* `no_match` - this allows you to specify what happens if there's an issue reaching the main apfell server or if you get a request to the docker container that doesn't match one of your specified endpoints. This is also what happens if you get a message to the right endpoint, but it's not properly encoded with the proper agent message (ex: fails to decrypt the agent message or fails to get the agent message from the url)
  * `action` - this allows you to specify which of the options you want to leverage
  * `redirect` - this simply returns a 302 redirect to the specified url
  * `proxy_get` - this proxies the request to the following url and returns that url's contents with the status code specified
  * `proxy_post` - this proxies the request to the following url and returns that url's contents with the specified status code
  * `return_file` - this allows you to return the contents of a specified file. This is useful to have a generic 404 page or saved static page for a sight you might be faking.
* `port` - which port this instance should listen on
* `key_path` - the path locally to where the key file is located for an SSL connection. If you upload the file through the web UI then the path here should simply be the name of the file.
* `cert_path` - the path locally to where the cert file is located for an SSL connection.If you upload the file through the web UI, then the path here should simply be the name of the file. Both this and the `key_path` must be specified and have valid files for the connection to the container to be SSL.
* `debug` - set this to true to allow debug messages to be printed. There can be a lot though, so once you have everything working, be sure to set this to `false` to speed things up a bit.
* `GET` and `POST` - simply take the `GET` and `POST` sections from your `agent configuration` mentioned above and paste those here. No changes necessary.&#x20;

### URI formatting

When it comes to the URIs you choose to use for this, there's an additional feature you can leverage. You can choose to keep them static (like `/download.php`) and specify/configure the query parameters/cookies/body, but you can also choose to register more dynamic URIs. There's an example of this above:

```
"uri": "/<test:string>"
```

What this does for the server side is register a pattern via [Sanic's Request Parameters](https://sanic.readthedocs.io/en/latest/sanic/routing.html#request-parameters). This allows you to register URIs that can change every time, but still be valid. For example:

```
"uri": "/downloads/category/<c:string>/page/<i:int>"
```

this would require you to have the URI of something like `/downloads/category/abcdefh/page/5` or the docker container will hit a `not_found` case and follow that action path. Combine this with the `urlFunctions` and you can have something to always generate unique URIs as follows:

```
"uri": "/downloads/category/<c:string>/page/<i:int>",
"urlFunctions": [
  {
    "name": "<c:string>",
    "value": "",
    "transforms": [
      {
        "function": "random_alpha",
        "parameters": [15]
      }
    ]
  },
  {
    "name": "<i:int>",
    "value": "",
    "transforms": [
    {
      "function": "random_number",
      "parameters": [3]
    }
    ]
  }
]
```

{% hint style="info" %}
Specifically for `urlFunctions`, the "name" must match the thing that'll be replaced. Unlike query parameters and cookie values where the name specifies the name for the value, the name here specifies which field is to be replaced with the result of the transforms
{% endhint %}

{% hint style="warning" %}
One last thing to note about this. You cannot have two URIs in a single instance within the GET or the POST that collide. For example, you can't have two URIs that are `/download.php` but vary in query parameters. As far as the docker container's service is concerned, the differentiation is between the URIs, not the query parameters, cookies, or body configuration. Now, in two different instances you can have overlapping URIs, but that's because they are different web servers bound to different ports.
{% endhint %}

### Special kinds of configuration

What if you want all of your messages to be `"POST"` requests or `"GET"` requests? Well, Apfell by default tries to do GET requests when getting tasking and POST requests for everything else; however, if there are no GET `agent_message` instances in that array (i.e. `{"GET":{"AgentMessage":[]}}`) then the agent should use POST messages instead and vise versa. This allows you to have yet another layer of customization in your profiles.

## Linting

Because this is a profile that offers a _LARGE_ amount of flexibility, it can get quite confusing. So, included with the dynamicHTTP is a linter (a program to check your configurations for errors). In `C2_Profiles/dynamicHTTP/c2_code/` is a `config_linter.py` file. Run this program with `./config_linter.py [path_to_agent_config.json]` where \[path\_to\_agent\_config.json] is a saved version of the configuration you'd supply your agent. The program will then check that file for errors, will check your server's `config.json` for errors, and then verify that there's a match between the agent config and server config (if there was no match, then your server wouldn't be able to understand your agent's messages). Here's an example:

```
its-a-feature@ubuntu:~/Desktop/Mythic/C2_Profiles/dynamicHTTP/c2_code$ ./config_linter.py agent_config.json 
[*] Checking server config for layout structure
[+] Server config layout structure is good
[*] Checking agent config for layout structure
[+] Agent config layout structure is good
[*] Looking into GET AgentMessages
[*] Current URLs: ['http://192.168.205.151:9000']
	Current URI: /<test:string>
[*] Found 'message' keyword in QueryParameter q
[*] Now checking server config for matching section
[*] Found matching URLs and URI, checking rest of AgentMessage
[+] FOUND MATCH
[*] Looking into POST AgentMessages
[*] Current URLs: ['http://192.168.205.151:9000']
	Current URI: /download.php
[*] Didn't find message keyword anywhere, assuming it to be the Body of the message
[*] Now checking server config for matching section
[*] Found matching URLs and URI, checking rest of AgentMessage
[*] Checking for matching Body messages
[+] FOUND MATCH

```

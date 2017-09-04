---
layout: 爬虫
title: PhantomJs学习心得
date: 2017-09-04 10:47:47
tags: [PhantomJS,爬虫]
---



## Quick Start

**下载地址:**http://phantomjs.org/download.html

下载完成之后都要加入Path中,这样才可以在命令行中进行调用.

### Hello,World

```js
console.log('Hello, world!');
phantom.exit();
```

直接看这个例子,log替换了一般语言的输出(print)

> 注意:这个地方的phantom.exit(),没有这条语句,它会永远在运行.

```js
phantomjs hello.js
```

通过这条语句我们进行运行.

The output is:

Hello,world!

### Page Loading

```js
var page = require('webpage').create();
page.open('http://example.com', function(status) {
  console.log("Status: " + status);
  if(status === "success") {
    page.render('example.png');
  }
  phantom.exit();
});
```

这里首先创建了`require('webpage').create()`对象,这个对象可以访问网站,通过`open(url,function函数)`方法.

这里直接用render(渲染)将页面转化成为一个图片.

```js
var page = require('webpage').create(),
  system = require('system'),
  t, address;

if (system.args.length === 1) {
  console.log('Usage: loadspeed.js <some URL>');
  phantom.exit();
}

t = Date.now();
address = system.args[1];
page.open(address, function(status) {
  if (status !== 'success') {
    console.log('FAIL to load the address');
  } else {
    t = Date.now() - t;
    console.log('Loading ' + system.args[1]);
    console.log('Loading time ' + t + ' msec');
  }
  phantom.exit();
});
```

这个地方可以写一个执行的脚本,通过`require('system')`可以获取到命令行其他的参数.

### Code Evaluation

```js
var page = require('webpage').create();
page.open(url, function(status) {
  var title = page.evaluate(function() {
    return document.title;
  });
  console.log('Page title is ' + title);
  phantom.exit();
});
```

```js
var page = require('webpage').create();
page.onConsoleMessage = function(msg) {
  console.log('Page title is ' + msg);
};
page.open(url, function(status) {
  page.evaluate(function() {
    console.log(document.title);
  });
  phantom.exit();
});
```

上面这个和下面这个输出的都一样,但是下面这个是通过`evaluate`方法进入的page的`onConsoleMessge`,现在我是这样理解的,就是调用evaluate结束后会调用onConsoleMessage.

### Network Requests and Responses

```js
var page = require('webpage').create();
page.onResourceRequested = function(request) {
  console.log('Request ' + JSON.stringify(request, undefined, 4));
};
page.onResourceReceived = function(response) {
  console.log('Receive ' + JSON.stringify(response, undefined, 4));
};
page.open(url);
```

重写的这两个方法:`onResourceRequested`与`onResourtceReceived`,返回的是他们的请求头,和一些状态.

**output:**

Receive {

```json
"contentType": "text/javascript",
"headers": [
    {
        "name": "Accept-Ranges",
        "value": "bytes"
    },
    {
        "name": "Vary",
        "value": "Accept-Encoding, Origin"
    },
    {
        "name": "Content-Encoding",
        "value": "gzip"
    },
    {
        "name": "Content-Type",
        "value": "text/javascript"
    },
    {
        "name": "Date",
        "value": "Tue, 29 Aug 2017 08:32:59 GMT"
    },
    {
        "name": "Expires",
        "value": "Wed, 29 Aug 2018 08:32:59 GMT"
    },
    {
        "name": "Last-Modified",
        "value": "Thu, 24 Aug 2017 05:15:00 GMT"
    },
    {
        "name": "X-Content-Type-Options",
        "value": "nosniff"
    },
    {
        "name": "Server",
        "value": "sffe"
    },
    {
        "name": "X-XSS-Protection",
        "value": "1; mode=block"
    },
    {
        "name": "Cache-Control",
        "value": "public, max-age=31536000"
    },
    {
        "name": "Age",
        "value": "259174"
    }
],
"id": 8,
"redirectURL": null,
"stage": "end",
"status": 200,
"statusText": "OK",
"time": "2017-09-01T08:32:33.206Z",
"url": "http://ssl.gstatic.com/gb/js/sem_458f6896768983bbac9b4b751040add8.js"
}
```

### Screen Capture

Since PhantomJS is using WebKit, a real layout and rendering engine, it can capture a web page as a screenshot. Because PhantomJS can render anything on the web page, it can be used to convert HTML content styled with CSS but also SVG, images and Canvas elements.

因为PhantomJs是适用WebKit,拥有real layout与渲染引擎,可以捕捉页面的屏幕快照.因为PhantomJs可以渲染任何的网页,它可以用来转换HTML格式的CSS,SVG,images和Canvas.

```js
var page = require('webpage').create();
page.open('http://github.com/', function() {
  page.render('github.png');
  phantom.exit();
});
```

将`http://github.com/`转换成为`github.png`.

在Github上面可以找到[rasterize.js](https://github.com/ariya/phantomjs/blob/master/examples/rasterize.js)源码,可以将SVG转换成为PNG,把页面转换成为PDF.


<!-- more -->


## Network Monitoring

PhantomJs允许检查网络流量,对网络行为和情况可以进行许多分析.

```js
var page = require('webpage').create();
page.onResourceRequested = function(request) {
  console.log('Request ' + JSON.stringify(request, undefined, 4));
};
page.onResourceReceived = function(response) {
  console.log('Receive ' + JSON.stringify(response, undefined, 4));
};
page.open(url);
```

这个在Quick Start那里也有,就是用来检查网络流量的.

我们可以使用[netsniff.js](https://github.com/ariya/phantomjs/blob/master/examples/netsniff.js)来收集数据并将其重新整理,网络流量的传播格式为[HAR format](http://www.softwareishard.com/blog/har-12-spec),我们使用[HAR viewer](http://www.softwareishard.com/blog/har-viewer)可以将结果显示出来,并以waterfall diagram形式出现.

## Page Automation

因为PhantomJS可以加载并且操作Web pager,而且对Web Pager的操作都是可以实现的.

### DOM Manipulation

```js
var page = require('webpage').create();
console.log('The default user agent is ' + page.settings.userAgent);
page.settings.userAgent = 'SpecialAgent';
page.open('http://www.httpuseragent.org', function(status) {
  if (status !== 'success') {
    console.log('Unable to access network');
  } else {
    var ua = page.evaluate(function() {
      return document.getElementById('myagent').textContent;
    });
    console.log(ua);
  }
  phantom.exit();
});
```

上面这个就是一个获取id为`myagent`的例子,我们在evalute中创建一个function,然后在这个函数中获取.

### Use jQuery and Other Libraries

```js
var page = require('webpage').create();
page.open('http://www.sample.com', function() {
  page.includeJs("http://ajax.googleapis.com/ajax/libs/jquery/1.6.1/jquery.min.js", function() {
    page.evaluate(function() {
      $("button").click();
    });
    phantom.exit()
  });
});
```

在1.6版本中,你可以直接包含jQuery在代码中,通过`page.includeJS`来实现.

### The Webpage instance

Suppose you have an instance of the webpage:

```
var page = require('webpage').create();
```

What can be extracted and executed on it?

#### Attributes

- page.canGoForward -> boolean

If window.history.forward would be a valid action

- page.canGoBack -> boolean

If window.history.back would be a valid action

- page.clipRect -> object

Can be set to an object of the following form:

```
{ top: 0, left: 0, width: 1024, height: 768 }
```

It specifies which part of the screen will be taken in the screenshot

- page.content -> string

The whole HTML content of the page

- page.cookies -> object

The cookies. They have this form:

```
{
    'name' : 'Valid-Cookie-Name',
    'value' : 'Valid-Cookie-Value',
    'domain' : 'localhost',
    'path' : '/foo',
    'httponly' : true,
    'secure' : false
}
```

- page.customHeaders -> object

TODO

- page.event -> object

Contains modifiers and keys TODO

- page.libraryPath -> string

The current library path, usually it’s the directory where the script is executed from

- page.loading -> boolean

If the page is loading or not

- page.loadingProgress -> number

The percentage that has been loaded. 100 means that the page is loaded.

- page.navigationLocked -> boolean

TODO

- page.offlineStoragePath -> string Where the sqlite3 localstorage and other offline data are stored.
- page.offlineStorageQuota, ‘number

The quota in bytes that can be stored offline

- page.paperSize -> object

Similar to clipRect but takes real paper sizes such as A4. For an in depth example check  [this](https://github.com/ariya/phantomjs/blob/d10b8dc5832797be434f43fa2cbd4f1110d035fb/examples/printheaderfooter.js).

- page.plainText -> string

The elements that are plain text in the page

- page.scrollPosition -> object

The current scrolling position as an object of the following form:

```
{
	left: 0
	top: 0
}
```

- page.settings -> object

The settings which currently only has the useragent string page.settings.userAgent = ‘SpecialAgent’;

- page.title -> string

The page title

- page.url -> string

The page url

- page.viewportSize -> object

The browser size which is in the following form:

```
{
	width: 1024,
	height: 768
}
```

- page.windowName -> string

The name of the browser window that is assigned by the WM.

- page.zoomFactor -> number

The zoom factor. 1 is the normal zoom.

## Inter Process Communication

这是通过多种方式来处理PhantomJS与其他进程之间的进程交流.

### File I / O

- [files](http://phantomjs.org/api/fs/)

### HTTP

- outgoing HTTP requests (to other processes/services)
  - GET/POST data to a server endpoint
  - GET/POST data to a server endpoint, parse resulting JSON/XML/HTML/etc.
  - Create a XMLHttpRequest object and use it as you would in a normal web page
  - Cross-domain requests are restricted by default (PhantomJS code is considered to be in file:// scope). Use [Access-Control-Allow-Origin](http://www.w3.org/TR/cors/#access-control-allow-origin-response-header) on your server to enable access.
- incoming HTTP requests (server). Use the [WebServer module](http://phantomjs.org/api/webserver/)
- route PhantomJS traffic through an HTTPS proxy like [mitmproxy/libmprox](http://mitmproxy.org/) or [fiddler](http://fiddler2.com/)

### Websockets inside a WebPage context

- [hixie-76](http://tools.ietf.org/html/draft-hixie-thewebsocketprotocol-76) websockets only
- PhantomJS 2.x will eventually get a WebKit upgrade that has [RFC 6455](http://tools.ietf.org/html/rfc6455) websockets.

### stdin, stdout, and stderr

See the [stdin-stdout-stderr.js](https://github.com/ariya/phantomjs/blob/master/examples/stdin-stdout-stderr.js) example script for details.

## Command Line Interface

**currently applies to latest PhantomJS release:**PhantomJS 2.1.1

### Command-line Options

Supported command-line options are:

- `--help` or `-h` lists all possible command-line options. *Halts immediately, will not run a script passed as argument.*
- `--version` or `-v` prints out the version of PhantomJS. *Halts immediately, will not run a script passed as argument.*
- `--debug=[true|false]` prints additional warning and debug message, default is `false`. Also accepted: `[yes|no]`.
- `--config` specifies JSON-formatted configuration file (see below).
- `--cookies-file=/path/to/cookies.txt` specifies the file name to store the persistent Cookies.
- `--disk-cache=[true|false]` enables disk cache (at desktop services cache storage location, default is `false`). Also accepted: `[yes|no]`.
- `--disk-cache-path` specifies the location for the disk cache.
- `--ignore-ssl-errors=[true|false]` ignores SSL errors, such as expired or self-signed certificate errors (default is `false`). Also accepted: `[yes|no]`.
- `--load-images=[true|false]` load all inlined images (default is `true`). Also accepted: `[yes|no]`.
- `--local-storage-path=/some/path` path to save LocalStorage content and WebSQL content.
- `--local-storage-quota=number` maximum size to allow for data.
- `--local-url-access` allows use of ‘file:///’ URLs (default is ‘true’).
- `--local-to-remote-url-access=[true|false]` allows local content to access remote URL (default is `false`). Also accepted: `[yes|no]`.
- `--max-disk-cache-size=size` limits the size of disk cache (in KB).
- `--offline-storage-path` specifies the location for offline storage.
- `--offline-storage-quota` sets the maximum size of the offline storage (in KB).
- `--output-encoding=encoding` sets the encoding used for terminal output (default is `utf8`).
- `--remote-debugger-port` starts the script in a debug harness and listens on the specified port
- `--remote-debugger-autorun` runs the script in the debugger immediately: ‘yes’ or ‘no’ (default)
- `--proxy=address:port` specifies the proxy server to use (e.g. `--proxy=192.168.1.42:8080`).
- `--proxy-type=[http|socks5|none]` specifies the type of the proxy server (default is `http`).
- `--proxy-auth` specifies the authentication information for the proxy, e.g. `--proxy-auth=username:password`).
- `--script-encoding=encoding` sets the encoding used for the starting script (default is `utf8`).
- `--script-language` sets the script language instead of detecting it: ‘javascript’.
- `--ssl-protocol=[sslv3|sslv2|tlsv1|tlsv1.1|tlsv1.2|any']` sets the SSL protocol for secure connections (default is `SSLv3`). Not all values may be supported, depending on the system OpenSSL library.
- `--ssl-certificates-path=<val>` sets the location for custom CA certificates (if none set, uses system default).
- `--ssl-client-certificate-file` sets the location of a client certificate.
- `--ssl-client-key-file` sets the location of a clients’ private key.
- `--ssl-client-key-passphrase` sets the passphrase for the clients’ private key.
- `--ssl-ciphers` sets supported TLS/SSL ciphers. Argument is a colon-separated list of OpenSSL cipher names (macros like ALL, kRSA, etc. may not be used). Default matches modern browsers.
- `--web-security=[true|false]` enables web security and forbids cross-domain XHR (default is `true`). Also accepted: `[yes|no]`.
- `--webdriver` starts in ‘Remote WebDriver mode’ (embedded GhostDriver): ‘[[:]]' (default '127.0.0.1:8910')
- `--webdriver-selenium-grid-hub` URL to the Selenium Grid HUB: ‘URL_TO_HUB’ (default ‘none’) (NOTE: works only together with ‘–webdriver’)
- `--webdriver-logfile` file where to write the WebDriver’s Log (default ‘none’) (NOTE: needs ‘–webdriver’)
- `--webdriver-loglevel` WebDriver Logging Level: (supported: ‘ERROR’, ‘WARN’, ‘INFO’, ‘DEBUG’) (default ‘INFO’) (NOTE: needs ‘–webdriver’)

Alternatively, since PhantomJS 1.3, you can also utilize a JavaScript Object Notation (JSON) configuration file instead of passing in multiple command-line options:

```
--config=/path/to/config.json
```

The contents of `config.json` should be a standalone JavaScript object. Keys are de-dashed, camel-cased equivalents of the other supported command-line options (excluding `--version`/`-v` and `--help`/`-h`). Values are their JavaScript equivalents: ‘true’/’false’ (or ‘yes’/’no’) values translate into `true`/`false` Boolean values, numbers remain numbers, strings remain strings. For example:

```js
{
  /* Same as: --ignore-ssl-errors=true */
  "ignoreSslErrors": true,

  /* Same as: --max-disk-cache-size=1000 */
  "maxDiskCacheSize": 1000,

  /* Same as: --output-encoding=utf8 */
  "outputEncoding": "utf8"

  /* etc. */
}

There are some keys that do not translate directly:

 * --disk-cache => diskCacheEnabled
 * --load-images => autoLoadImages
 * --local-storage-path => offlineStoragePath
 * --local-storage-quota => offlineStorageDefaultQuota
 * --local-to-remote-url-access => localToRemoteUrlAccessEnabled
 * --web-security => webSecurityEnabled
 * --debug => printDebugMessages
```

## Summary

PhantomJS拥有WebKit,WebKit是一个排版引擎也称浏览器内核,它具备rendering engine.同时有一系列操作DOM,同时还可以写JQuery来操作元素.
---
layout: post
comments: true
title: Connect Module 学习笔记
category: Nodejs
tag: nodejs
---

![Middleware](http://mforever78.qiniudn.com/middleware.jpg "Connect Middleware")

##简介

- Connect 可以理解为中间件（`Middleware`）的栈。
- 中间件有两种，分别是`filter`和`provider`：
	- `filter` 是对请求的处理，并不对请求作出回应。
	- `provider` 负责对请求进行回应。

##使用方法

在终端中切换至工作目录下，安装 Connect：

```bash	
npm install connect
```

调用 Connect：

```js
var connect = require("connect");
```

此时变量 connect 是一个 function ，返回值是一个新的 Connect 实例。接下来建立一个名为 `app` 的 Conncect 实例：

```js
var app = connect();
```

有时候并不需要这样一个 `app` 变量，可以直接使用 `use()` 语句将 `connect()` 链起来，如：

```js
connect()
	.use(/* middleware */)
	.use(/* middleware */)
	.use(/* middleware */)
	.listen(3000);
```

##编写中间件

中间件的基本语法为

```js
.use(function (req, res, next) {
	// do something
})
```
其中 `req` 和 `res` 分别是请求和响应对象，而通过调用 `next()` 可以使请求被下一个中间件捕捉到。

**注意**：如果没有对请求作出响应，则必须使用 `next()` 将请求传递至下一个中间件，否则会使页面处于空白无响应状态。

一个例子：

```js
connect()
    .use(function (req, res, next) {
        if (req.method === 'POST') {
            res.end("This is a POST request");
        } else {
            next();
        }
    })
    .use(function (req, res) {
        res.end("This is not a POST request (probably a GET request)");
    }).listen(3000);
```

##Request Object & Response Object

`request` 对象实际上是一个 `http.IncomingMessage` 对象，它有如下重要的属性：

- `req.method` 是 HTTP 请求 method。
- `req.url` 是请求的 url。
- `req.header` 是 hearder 对象。
- `req.query` 是查询字符串中的数据组成的对象。需在合适的地方对 query string 使用 `connect.query()` 进行 parse 操作。
- `req.body` 是表单对象。需进行 body parse 操作。
- `req.cookies` 是 cookies 对象。需进行 cookies parse 操作。
- `req.session` 是session 对象。需要 cookies parse 和 session 中间件。

一个完整的例子：

```js
connect()
    .use(connect.query()) // gives us req.query
    .use(connect.bodyParser())  // gives us req.body
    .use(connect.cookieParser()) // for session
    .use(connect.session({ secret: "asdf" }))     // gives us req.session
    .use(function (req, res) {
        res.write("req.url: " + req.url + "\n\n");
        res.write("req.method: " + req.method + "\n\n");
        res.write("req.headers: " + JSON.stringify(req.headers) + "\n\n");
        res.write("req.query: " + JSON.stringify(req.query) + "\n\n");
        res.write("req.body: " + JSON.stringify(req.body) + "\n\n");
        res.write("req.cookies: " + JSON.stringify(req.cookies) + "\n\n");
        res.write("req.session: " + JSON.stringify(req.session));
        res.end();
    }).listen(3000);
```
没有 Express 这样的框架的支持，不能直接对 `respose` 进行类似于 `render` 这样的操作，必须手动写入所有信息。

首先，使用 `writeHead()` 写入中响应头：

```js
    var body = 'hello world';
    response.writeHead(200, {
        'Content-Length': body.length,
        'Content-Type': 'text/plain'
    });
```
也可以使用 `setHeader()` 方法进行单项修改：

```js
connect()
    .use(function (req, res) {
        var accept = req.headers.accept.split(","),
            body, type;
        console.log(accept);
        if (accept.indexOf("application/json") &gt; -1) {
            type = "application/json";
            body = JSON.stringify({ message: "hello" });
        } else if (accept.indexOf("text/html") &gt; -1) {
            type = "text/html";
            body = "<h1> Hello! </h1>";
        } else {
            type = "text/plain";
            body = "hello!";
        }
        res.statusCode = 200;
        res.setHeader("Content-Type", type);
        res.end(body);
    }).listen(3000);
```

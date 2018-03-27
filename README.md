# route
## node模仿express封装路由
```js
var http = require('http');
var url = require('url');

var R = {}; //R用来接收参数
R._get = {}, R._post = {},
methods = ["get", "post"];
var app = function (req, res) {
    var pathname = url.parse(req.url).pathname;
    if (!pathname.endsWith("/")) pathname += "/";
    var method = req.method.toLowerCase();
    // R["_"+method][pathname] ? R["_"+method][pathname](req, res) : res.end("网址不存在");
    if (R["_" + method][pathname]) {
        if (method == "get") {
            R["_" + method][pathname](req, res)
        } else if (method == "post") {
            var data = "";
            req.on("data", function (chunk) {
                data += chunk;
            })
            req.on("end", function () {
                res.body = data;
                R["_" + method][pathname](req, res);
            })
        }
    } else {
        res.end("Not Found");
    }
}
methods.forEach(function(v) {
    app[v] = function (string, fun) {
        if (!string.startsWith("/")) string = "/" + string;
        if (!string.endsWith("/")) string += "/";
        R["_"+v][string] = fun
    }
})
http.createServer(app).listen(9001);    // 把app提出去
app.get("/", function (req, res) {    //测试
    res.writeHead(200, { "Content-Type": "text/html;charset='utf-8'" });
    res.end(`
        <form action='/form' method='POST'>
            <input type='text' name='text'>
            <input type='submit'>  
        </form>
    `)
})
app.post("/form", function (req, res) {
    res.end(res.body)
})
```

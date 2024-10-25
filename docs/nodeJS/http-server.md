---
title: "[Node.js]HTTP/1.1 GET, POST 요청 처리하기(1) - Server"
last_modified_date: "2023-11-29 17:20:38"
parent: "JavaScript & NodeJS"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">[Node.js]HTTP/1.1 GET, POST 요청 처리하기(1) - Server</div>

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

### HTTP 1.1?

![img](https://blog.kakaocdn.net/dn/l5faU/btsA4VXa0Ys/8vMrIWwvIMgPqNxuk7Eoq0/img.png)

Client가 Server에게 HTTP request를 보내면, Server에서 Client로 HTTP response를 보내게 된다. 

Client가 Server에게 request를 보내기 위해서는 URL(Uniform Resource Locator)를 사용해 정적/동적인 컨텐츠의 위치를 설명하고, method 라는 것을 사용한다.



![img](https://blog.kakaocdn.net/dn/pbLBw/btsBbFrJeLs/yPnCO7cxtKT1kODDzG6KS0/img.png)

HTTP의 주요 method로는 GET, POST, PUT, DELETE가 존재한다. 이번에는 NodeJS를 활용해 GET, POST를 구현해 보았다. 



---

## GET 요청

GET은 client가 server에게 리소스를 달라고 요청하여 서버로부터 리소스를 받는 것을 의미한다. Request에서 메서드를 확인한 뒤 GET이면 해당 요청을 처리한다. 



Node.js 기반의 서버 소켓을 활용해 서버 코드를 작성하였다. 

![img](https://blog.kakaocdn.net/dn/c1dmJB/btsA5gNZbXQ/MjP8oQWglHb16ej2P2pob1/img.png)

![img](https://blog.kakaocdn.net/dn/bwGF1R/btsA7NY3FP0/NzbsvcODcKNcqTlyLTINCk/img.png)

![img](https://blog.kakaocdn.net/dn/tchPl/btsA7NSel9y/AF0hV00ghllSrNKDUgOcXk/img.png)



8080포트를 통해 서버가 실행되면 URL 파싱을 통해 query가 존재할 시 쿼리 스트링을 각각 변수화한다. req.method를 통해 method를 확인한 뒤, GET일 시 헤더에 응답코드 200을 전달하고, query 유무를 확인하고 이에 맞는 기능을 수행한다. 



![img](https://blog.kakaocdn.net/dn/bgiIX9/btsA7qCMUtr/f57kRVDWWeYB8zaX7ZJTf0/img.png)

터미널에서 실행한 뒤 postman이나 일반 웹 서버에서 요청을 보내면 확인이 가능하다. 

다음 포스트에서 client 코드를 작성하는 과정을 소개하겠다. 



---

## POST 요청

Text를 서버로 보내 로직 서버가 작업을 하도록 한다. 

![img](https://blog.kakaocdn.net/dn/bo1FwN/btsBb5c2q0F/13QZ9nCZHVNunxaucwqY8K/img.png)

request의 메서드가 POST 일때 req.on() 명령어를 통해 데이터를 받는다. 

**req.on('data', function())**은 데이터를 수신할 때마다 function을 실행한다.

**req.on('end', function())**은 데이터 수신이 끝났을 때 function을 실행한다. 

client 코드를 통해 테스트를 확인하는 것은 다음 글에서 소개하겠다. 



### 전체 코드

```javascript
const http = require('http');
const url = require('url');
const queryString = require('querystring');

function simple_calc(var1, var2) {
    return var1 * var2;
};

const server = http.createServer((req,res) => {
    const reqUrl = new URL(req.url, `http://${req.headers.host}`);
    console.log('::Client address   :', req.socket.remoteAddress);
    console.log('::Client port      :', req.socket.remotePort);
    console.log('::Request command  :', req.method);
    console.log('::Request line     :', req.method + ' ' + req.url + ' HTTP/' + req.httpVersion);
    console.log('::Request path     :', reqUrl.pathname);
    console.log('::Request version  : HTTP/', req.httpVersion);

    const parsedUrl = url.parse(req.url);
    const parsedQuery = queryString.parse(parsedUrl.query, '&', '=');
    res.writeHead(200);

    if(req.method == "GET") {
        console.log("## GET activated");

        if(parsedUrl.query) {
            const result = simple_calc(parseInt(parsedQuery.var1), parseInt(parsedQuery.var2));
            res.write("<html>");
            res.write(`GET request for calculation => ${parsedQuery.var1} x ${parsedQuery.var2} = ${result}!` );
            res.write("</html>");
            console.log(`## GET request for calculation => ${parsedQuery.var1} x ${parsedQuery.var2} = ${result}!`);
        }
        else{
            res.write('<html>');
            res.write(`<p>HTTP Request GET for path: ${parsedUrl.path}</p>`);
            res.write('</html>');
            console.log(`## GET request for directory => ${parsedUrl.path}`);
        }
    }
    else if(req.method == "POST") {
        console.log('## POST activated');
        let body = '';
        
        req.on('data', function(data) {
            body += data;
        });

        
        req.on('end', function() {
            const post = JSON.parse(body);
            const result = simple_calc(parseInt(post.var1), parseInt(post.var2));
            res.write(`POST request for calculation => ${post.var1} x ${post.var2} = ${result}!`);
            console.log(`## POST request data => ${body}`);
            console.log(`## POST request for calculation => ${post.var1} x ${post.var2} = ${result}!`);
        });
    }

    res.end();
});

server.listen(8080, () => {
    const server_name = "localhost";
    console.log(`## HTTP Server started at http://${server_name}:8080`);
})
```






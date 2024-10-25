---
title: "[Node.js] HTTP/1.1 GET, POST 요청 처리하기(2) - Client(Feat. Axios)"
last_modified_date: "2023-12-04 18:31:38"
parent: "JavaScript & NodeJS"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">[Node.js] HTTP/1.1 GET, POST 요청 처리하기(2) - Client(Feat. Axios)</div>

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

저번 시간에 HTTP/1.1의 GET, POST 요청 처리하는 서버 코드를 작성해 보았다. 이번 시간에는 서버에게 응답을 요구하는 클라이언트 코드에 대해 알아보도록 하겠다.



## Client code란?

 어떤 기능을 제공하는 쪽을 서버(Server), 제공받는 쪽을 클라이언트(Client)라고 한다. 코드도 마찬가지로 기능을 제공하는 서버 코드, 기능을 제공받는 클라이언트 코드가 존재한다. 특정 기능을 구현해 호출되는 코드를 서버 코드, 이 코드를 호출하는 모든 코드(객체, 클래스)가 클라이언트 코드라고 한다.

간단히 말하자면 "클라이언트 = 호출, 서버 = 응답" 이 되겠다. 



## Axios

![img](https://blog.kakaocdn.net/dn/vV6ni/btsBlcKPQCS/1Mtobbs067FTEuYXkJk8B0/img.jpg)

여기서는 Client Code를 짜기 위해 Node.js환경에서 Axios를 활용해보려고 한다. 

Axios란 Nodejs와 http를 위한 promise기반 HTTP 클라이언트이다. 이는 비동기 http통신을 지원한다. 

 

대표적으로 이렇게 설치가 가능하다. 

![img](https://blog.kakaocdn.net/dn/sXXJi/btsBjdjnrIw/K2hRFoVQGaXa6IRGktSGKK/img.png)



## Client Code

axios get/post로 쉽게 client code를 짤 수 있다. 

```javascript
const axios = require("axios");

console.log("## HTTP client started.");

console.log("## GET request for http://localhost:8080/temp/");
axios.get('http://localhost:8080/temp/')
    .then(function(response) {
        console.log("## GET response [start]");
        console.log(response.data);
        console.log("## GET response [end]");
    })
    .catch(function(error) {
        console.log(error);
    });

console.log("## GET request for http://localhost:8080/?var1=9&var2=9");
axios.get('http://localhost:8080/?var1=9&var2=9')
    .then(function(response) {
        console.log("## GET response [start]");
        console.log(response.data);
        console.log("## GET response [end]");
    })
    .catch(function(error) {
        console.log(error);
    });

console.log("## POST request for http://localhost:8080/ with var1 is 9 and var2 is 9");
axios.post(`http://localhost:8080`, { var1: '9', var2: '9' })
    .then(function(response) {
        console.log("## POST response [start]");
        console.log(response.data);
        console.log("## POST response [end]");
    })
    .catch(function(error){
        console.log(error); 
    });

console.log("## HTTP client completed.")
```



## 실행 화면

서버를 실행시키고, client 코드를 실행시킨다. 



### Client 터미널

![img](https://blog.kakaocdn.net/dn/b7uF9A/btsBl4Z50f0/sWttRjglnZxkeIf87j5t9k/img.png)



### 서버 터미널

클라이언트의 주소와 포트번호, API, 버전 등등을 가져올 수 있다.

![img](https://blog.kakaocdn.net/dn/qi5WO/btsBqDHp1ow/Z13WVFnHbMRYvBwSjmzYbK/img.png)




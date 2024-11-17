---
title: "[AWS] 포트포워딩 & HTTP Redirect"
last_modified_date: "2024-11-16 14:33:38"
parent: "Cloud"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">[AWS] 포트포워딩 & HTTP Redirect</div>

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

![bootstrap](../../../assets/images/cloud/port-forwarding/bootstrap.png)

> 진행했던 프로젝트의 서버는 8080포트를 사용한다. 하지만 client 단에서 서버에 요청을 보낼 때는 HTTP(S) 프토토콜을 통해 API를 호출해주기 때문에 해당 요청을 정상적으로 처리하기 위해선 해당 요청들을 8080포트로 바꿔주는 작업이 필요했다. 
>
> Nginx로도 포트포워딩과 리다이렉션이 가능하지만 AWS에서 제공하는 서비스들로 아키텍처를 구축해 보았다. 



## 기본 개념

**- 포트포워딩 vs 리다이렉션**

서버는 모든 요청을 8080 포트로 받아들이기 때문에, 만일 다른 포트로 요청이 들어온다면 해당 포트로 변경해주는 작업이 필요하다. 

이러한 작업을 '**포트포워딩**'이라고 부른다. 포트포워딩의 사전적 정의는 **"특정 포트로 들어오는 데이터 패킷을 다른 포트로 변경하여 다시 전송하는 작업"**이라고 할 수 있다. 일반적으로 인터넷 공유기 또는 방화벽이 네트워크에서 내부 IP로 들어오는 트래픽을 특정 컴퓨터로 전달하기 위해 사용된다.   

**리다이렉션**은 클라이언트에게 다른 URL로 이동하라는 지시를 내려주는 것을 말한다. 주로 웹 브라우저를 이동시키거나, HTTP를 HTTPS 프로토콜로 변경시키는 작업에 많이 사용된다. 

> HTTPS를 사용하기 위해서는 SSL/TLS 인증서가 필요하다. 해당 인증서는 웹사이트와 사용자 간의 통신을 암호화하여 보안성을 강화하는 데 필요한 요소이다. 

**리다이렉트와 포워딩의 차이점**

- 리다이렉트는 url 주소가 달라지고, 포워딩은 url 주소가 달라지지 않는다.

  - 리다이렉트는 클라이언트가 서버에게 요청을 보내면 서버는 어떠한 일을 처리하고 클라이언트에게 새롭게 요청할 곳을 알려준다.
- 포워딩은 클라이언트가 요청을 보냈을 때 또 다른 서버에게 일을 넘긴다. 
  
  - 즉 클라이언트는 서버에서 진행하는 일을 알 필요가 없다. 따라서 url 주소가 변경되지 않는다.
- 리다이렉트는 request, response 객체가 여러 번 생성되고, 포워드는 한 번만 생성되어 재사용(공유)할 수 있다.


​    

**- HTTP vs HTTPS**

**HTTP(Hypertext Transfer Protocol)**는 클라이언트와 서버 간 통신을 위한 통신 규칙 세트 또는 프로토콜이다. 사용자가 웹 사이트를 방문하면 사용자 브라우저가 웹 서버에 HTTP 요청을 전송하고, 웹 서버는 HTTP 응답으로 응답한다. 웹 서버와 사용자 브라우저는 데이터를 일반 텍스트로 교환한다. 간단히 말해 HTTP 프로토콜은 네트워크 통신을 작동하게 하는 기본 기술이다. 이름에서 알 수 있듯이 **HTTPS(Hypertext Transfer Protocol Secure)**는 HTTP의 확장 버전 또는 더 안전한 버전이다. HTTPS에서는 브라우저와 서버가 데이터를 전송하기 전에 안전하고 암호화된 연결을 설정한다.

HTTP는 암호화되지 않은 데이터를 전송하기 때문에 브라우저에서 전송된 정보를 제3자가 가로채고 읽을 수 있다. 따라서 통신에 또 다른 보안 계층을 추가하기 위해 HTTPS로 확장되었다. HTTPS는 HTTP 요청 및 응답을 SSL 및 TLS 기술에 결합한다. 이러한 웹 사이트는 독립된 인증기관에서 SSL/TLS 인증서를 획득해야 하고, 데이터를 교환하기 전에 브라우저와 인증서를 공유하게 된다. 

![http](../../../assets/images/cloud/port-forwarding/https.png)

## AWS ELB(Elastic Load Balancing)

ELB(Elastic Load Balancer)란 애플리케이션 트래픽을 여러 대상에 자동 분산시켜 안정적인 AWS서버 환경을 운용하는데 도움을 주는 서비스다. EC2 뿐 아니라 컨테이너, AWS Lamda 등으로 다양한 서비스와 연계하여 부하를 분배할 수 있다.

<img src="../../../assets/images/cloud/port-forwarding/elb.png" alt="elb" width="400">

부하분산뿐만 아니라 부하 분산 대상에 대한 **헬스 체크(Health Check), 고정 세션, SSL 암복호화, 헬스 체크를 통한 다운서버 제외** 등 다양한 기능이 가능하다.

본 글에서는 포트포워딩과 HTTPS 리다이렉션에 관해 다루므로, 오토스케일링을 통한 로드밸런싱 보다는 해당 내용에 대해 자세히 기술하겠다. 

> 기회가 된다면 ELB의 메인 기능인 로드밸런싱 및 오토스케일링에 관해 다뤄보겠다.





프로젝트에서 HTTPS/로드밸런서를 합쳐서 운영되어야 하는 과정을 살펴보면 다음과 같았다. 

**(1) Client가 Request를 보낸다.**

**(2) 해당 요청을 ELB가 HTTPS(443 port) 요청인지 일반적인 HTTP(80 port) 요청인지 판단한다.** 

**(3) HTTP 요청이면 이 요청을 HTTPS 요청으로 redirection한다. HTTPS요청이면 Target Group(EC2 인스턴스)의 8080 포트로 포워딩한다.**

![port-forward](../../../assets/images/cloud/port-forwarding/port.png)

### SSL 인증서 발급 및 적용

우선 HTTPS를 사용하기 위해서는 SSL 인증서가 필요하다. SSL 인증서는 인증 기관(CA, Certificate Authority)가 웹사이트의 신원을 확인하고 발급하기 때문에, 서버의 신뢰성을 보장한다. 

Route53 세팅을 하며 발급받았던 SSL 인증서를 사용했다.



### ELB 생성

![alb](../../../assets/images/cloud/port-forwarding/alb.png)

ALB를 생성한다. 

> **ALB(Application Load Balancer)는 OSI 7계층 중 최상단 일곱번째 계층에 해당하는 Application Layer를 다루는 로드밸런서이다. 사용자와 직접 접하는 계층으로, HTTP/HTTPS 트래픽을 이용하는 애플리케이션에 적합한 로드밸런서이다.**
>
> **즉 특정 path(/shop, /user) 또는 port에 따라 대상 그룹을 나눠서 보내줄 수 있다.** 



![network](../../../assets/images/cloud/port-forwarding/network.png)

네트워크 매핑은 EC2가 존재하는 서브넷을 선택해준다. 그래야 EC2와 매핑되어 정확히 트래픽을 라우팅할 수 있기 때문이다. 



![secure](../../../assets/images/cloud/port-forwarding/secure.png)

보안 그룹은 ELB를 위해 새로 하나를 생성한다. HTTPS 및 HTTP 포트만 통과시키면 되므로, 두 포트에 대한 모든 요청을 허용한다. 

또한 EC2에 대한 보안 그룹 세팅을 수정한다. 모든 요청은 ELB로만 받아들이므로 EC2의 보안 그룹은 ELB의 보안 그룹만 허용한다. 





![listener](../../../assets/images/cloud/port-forwarding/listener.png)

리스너 설정은 HTTPS, HTTP 두 개를 등록해주면 된다. HTTPS는 대상 그룹으로, HTTP는 HTTPS로 리다이렉션되도록 설정한다. 

HTTPS는 SSL 인증서가 필요하므로 사전에 발급받은 인증서를 사용한다.

여기서 **대상 그룹**이라는 것이 필요한데, 우리의 ELB는 EC2 인스턴스로 라우팅되도록 연결하면 되므로 대상 그룹을 생성해 8080포트로 라우팅되도록 설정한다. 



![target-group](../../../assets/images/cloud/port-forwarding/target-group.png)

상태 검사는 해당 대상 그룹 내부의 인스턴스들이 건강한(Healthy) 상태인지 체크해, 정상적으로 작동하는 인스턴스에만 트래픽을 분배한다. 이 헬스체크를 하기 위해, 원하는 감시 규칙을 추가해준다. 나 같은 경우는 root page에 404 코드가 전달되면 서버가 healthy 상태임을 판단하도록 설정했다.



![dns](../../../assets/images/cloud/port-forwarding/dns.png)

로드밸런서가 생성되면, ALB(Application Load Balancer)가 생성되면 IP주소가 변동되기 때문에 Client에서 Access할 ELB의 DNS name을 제공해준다. 

만약 Route53을 사용한다면, 해당 ELB를 A레코드로 생성해주면 도메인 연결까지 할 수 있다. 



이렇게 ELB를 활용한 HTTP redirection&Port Forwarding을 마치겠다.



## REF

- [[Web] Nginx HTTP redirect 및 포트포워딩](https://ekdms5566.tistory.com/29)
- [HTTP와 HTTPS의 차이점은 무엇인가요?- AWS](https://aws.amazon.com/ko/compare/the-difference-between-https-and-http/)
- [ELB(Elastic Load Balancer) 구성 & 사용법 가이드-Inpa Dev](https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-ELB-Elastic-Load-Balancer-%EA%B0%9C%EB%85%90-%EC%9B%90%EB%A6%AC-%EA%B5%AC%EC%B6%95-%EC%84%B8%ED%8C%85-CLB-ALB-NLB-GLB)


















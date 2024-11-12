---
title: "[AWS] Route53 적용하기"
last_modified_date: "2024-10-27 18:53:38"
parent: "Cloud"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">[AWS] Route53 적용하기</div>

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

### Route53 도입 계기

> **프로젝트에서 ip 주소를 노출하지 않기 위해 내가 원하는 도메인 이름으로 호스팅하기 위한 DNS서비스를 활용하기 위해 route53을 도입하게 되었다.  .** 



# Route53

Amazon Route 53은 가용과 확장성이 뛰어난 Domain Name System(DNS) 웹 서비스이다.

Route53은 일반적인 DNS기능에, 모니터링 기능, L4 기능(Server Load Balancing), GSLB(Global Server Load Balancing) 기능까지 제공한다.

또한 사용자의 요청을 EC2, S3 버킷 등의 인프라 직접 연결이 가능하고, 외부의 인프라로도 라우팅이 가능하다.



## DNS?

![DNS](../../../assets/images/cloud/route53/DNS.png)

- 사용자 디바이스에서 도메인 이름을 요청하면, 해당 요청은 Cache DNS(Local DNS) 서버로 전달된다. 만일 예전에 접속했던 이력이 있다면 Local DNS에 캐싱된 정보가 저장되어 있어 바로 접속이 가능하다. 

- 만약 캐시에 저장된 정보가 없다면 상위 DNS 서버에 요청을 보내게 되고, 제일 처음으로 ROOT DNS 서버로 요청을 보낸다.

  - Root DNS?

    - Root DNS는 인터넷의 도메인 네임 시스템의 루트 존이다. 
      ICANN이 직접 관리하는 절대 존엄 서버로, TLD DNS 서버 IP들을 저장해두고 안내하는 역할을 한다.
      전세계에 961개의 루트 DNS가 운영되고 있다.
- 만일 ROOT DNS에서 찾을 수 없다면, Cache DNS에게 com 도메인을 관리하는 .com ROOT DNS(TLD DNS)서버의 주소를 반환하며 다시금 요청하라고 전달한다.
- 만약 여기에도 없다면, Local(Cahce) DNS 서버는 Authoritative DNS 서버인 route53의 네임서버에게 IP주소를 요청하고, 해당 도메인에 상응하는 IP 주소를 반환한다. 
- 이를 수신한 local DNS 서버는 IP주소를 캐싱한 뒤, 다른 요청이 올 때 응답할 수 있도록 IP 정보를 단말에 전달한다. 



### Route53 사용법

- 먼저 AWS 콘솔에서 Route53을 들어간 뒤, 이미 구매한 도메인이 있다면 호스팅 영역 > 호스팅 영역 생성에 들어간다.

![hosting](../../../assets/images/cloud/route53/hosting.png)



- 생성된 레코드의 네임서버를 도메인을 구매한 곳에 등록해준다. 

  ![hosting](../../../assets/images/cloud/route53/record.png)

  TTL은 원하는 대로 바꿔주면 된다. 



- 해당 도메인을 Ec2 인스턴스와 연결하기 위해 레코드를 생성해준다.

  ![record](../../../assets/images/cloud/route53/create_record.png)

  라우팅 정책에서 트래픽에 대한 라우팅 방법을 선택할 수 있다. 각 라우팅 정책에 대한 기능은 다음과 같다.
  
  > 단순 - 표준 DNS기능 사용
  >
  > 가중치 기반 - 각 리소스에 보낼 트래픽을 지정 가능
  >
  > 지리적 위치 - 지정한 위치의 가까운 사용자의 트래픽을 라우팅
  >
  > 지연 시간 - 지연시간이 가장 짧은 리전으로 트래픽을 라우팅
  >
  > 장애 조치 - 정상 상태일때만 트래픽을 라우팅
  
  레코드 이름에 라우팅할 이름을 지정해주고, 값 부분에 ec2 퍼블릭 IP 주소를 적어주면 된다. 



이러한 방법으로 내가 구매한 domain과 EC2를 연결할 수 있다. 

 




# REF

[🌐 DNS 개념 & 동작 ★ 알기 쉽게 정리](https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-DNS-%EA%B0%9C%EB%85%90-%EB%8F%99%EC%9E%91-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4-%E2%98%85-%EC%95%8C%EA%B8%B0-%EC%89%BD%EA%B2%8C-%EC%A0%95%EB%A6%AC)

[Route53 개념 원리 & 사용 세팅 💯 정리](https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-Route-53-%EA%B0%9C%EB%85%90-%EC%9B%90%EB%A6%AC-%EC%82%AC%EC%9A%A9-%EC%84%B8%ED%8C%85-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC)

[Route 53이란?](https://brunch.co.kr/@topasvga/85)










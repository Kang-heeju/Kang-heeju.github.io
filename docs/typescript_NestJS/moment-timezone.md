---
title: "[Typescript]Moment-timezone 라이브러리에 관해"
last_modified_date: "2024-10-14 19:14:38"
parent: "TypeScript & NestJS"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">[Typescript] Moment-timezone 라이브러리</div>

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



### **Moment-timezone 라이브러리를 쓰게 된 계기**

> **우리 BBIP 서비스에서는 주차별 스터디 시간을 db에 저장하고, 사용해야 했기 때문에 시간대를 db에 저장하는 날짜 type을 제대로 지정해줘야 했다.**

> **프론트가 API 호출할 때 정확한 type(date or string)과 시간 type이 아니면, 제대로 받아오지 못하는 상황이 발생하면서 어떤 시간 type을 사용해야 할 지 고민 중, moment-timezone을 활용하여 시간을 계산하고 처리하면 쉽다는 것을 알고, js 기본 date 함수에서 모두 moment-timezone 라이브러리로 바꾸었다.**



- 프론트엔드와 주고받기 위한 dto를 만들던 중, 프론트에게 주는 날짜 type에 따라 decoding 하는 방법이 달랐고, 모든 날짜 type을 통일하기 위해 moment-timezone을 도입했다.
- 또한 사용했던 mysql 데이터베이스가 utc type의 시간만 처리할 수 있었기에, seoul time을 사용해야 했던 우리는 모든 시간대를 utc 형태로 고쳐서 사용해야 했다. 그리고 모든 날짜 type은 ISO8601을 따르기로 했다. 
- 이러한 처리를 쉽게 하기 위해 기본 date 함수 말고 외부 라이브러리인 moment-timezone을 사용하기로 했다.



### **Moment-timezone 사용법**

```typescript
import * as moment from 'moment-timezone';
```



- utc timezone 찍어내기
```typescript
const now = moment(); // 현재 컴퓨터의 timezone을 받아온다. 
const seoulTime = moment().tz('Asia/Seoul') //서울 시간
const formattedTime = seoultime.format(); //string type으로 format해서 반환
const formattedTime2 = seoultime.foramt('YYYY/MM/DD') //년/월/일 으로 반환
```



- 이러한 moment는 기본적으로 컴퓨터의 timezone을 받아오기 때문에, **`.tz()`**로 시간대를 지정해주는 것이 편리하다.
- 또한 **`.setDefault()`**를 통해 해당 moment가 불러올 시간대를 강제로 지정해줄 수도 있다.
- .clone, .add(), .isAfter() 등 다양한 내장 함수가 존재하고, 이름도 직관적이므로 편리하게 사용할 수 있다.



### **하지만 해당 라이브러리는 레거시 프로젝트로 공식적 선언이 되었다.**

라이브러리가 가벼운 편도 아니고, 현재 개발이 멈춘 프로젝트이므로 최근에는 다른 date 라이브러리로 많이 넘어가는 추세이다.

해당 라이브러리를 대체하는 라이브러리로는 Luxon이 있다.

가장 moment 스럽지만, moment-timezone의 단점을 많이 보완한 라이브러리이다.

다음 번 프로젝트를 진행한다면 moment-timezone 대신 luxon을 사용할 것 같다.


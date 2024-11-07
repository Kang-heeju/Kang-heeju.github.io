---
title: "TypeScript에 대한 심층 탐구"
last_modified_date: "2024-11-05 19:14:38"
parent: "TypeScript & NestJS"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">TypeScript에 대한 심층 탐구</div>

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

> **내가 최근에 주로 쓰는 언어 및 프레임워크는 TypeScript와 NestJS이다. 올해 초에 교내에서 진행하는 NestJS 스터디를 통해 typescript와 NestJS를 접하게 되었고, 기회가 되어 프로젝트 및 업무를 진행하면서 배운 TypeScript 및 NestJS의 장단점과 탄생배경 등에 대해 소개해보고자 한다.  또한 TypeScript 언어를 사용해보면서 느꼈던 의문점들에 관해서도 이번 기회에 정리해보려고 한다.**
>
> 어떤 개발 언어를 배울 때는, 해당 언어가 어떤 이유로 세상에 등장했는지, 기존 언어들의 어떤 단점을 보완하였는지에 대해 알고 있는 것이 중요하다고 생각한다. 그래야 다양한 언어를 다룰 줄 아는 상황에서 또는 어떤 프로젝트를 진행하는 데 있어 프로젝트 수준에 어울리는 언어를 골라서 제대로 사용할 수 있다고 생각하기 때문이다. 

TypeScript를 기존에 사용하던 사람이라면 본문을 조금 더 쉽게 이해할 수 있을 것이다.



## TypeScript?

<img src="../../../assets/images/typescript/nestjs/chart.png" alt="rank" width="500" />

Github에서 제공하는 [세계적으로 유명한 개발 언어 top 30](https://innovationgraph.github.com/global-metrics/programming-languages#programming-languages-rankings)이다. 잘 보이진 않지만 자세히 보면 TypeScript가 2020년도부터 치고 올라오면서 현재 Java를 체지고 4위를 기록하고 있는 것을 볼 수 있다. 그만큼 최근에 가장 뜨는 언어이고, 웹 프론트엔드 개발, 서버 개발, 어플리케이션 개발 등 다양한 분야에서 사용되고 있다. 

TypeScript는 Microsoft에서 개발하고 유지/관리하는 Apache 라이센스가 부여된 오픈소스이다. 일반 JavaScript로 컴파일되는 자바스크립트 superset(상위호환)으로 2021년 10월에 처음 릴리스 되었다고 한다.



## 그럼 왜 TypeScript를 사용해야 하는가?

TypeScript의 등장 배경을 알기 위해서는 우선 JavaScript라는 언어의 동작 원리부터 이해하고 있어야 한다. JavaScript는 초기에 웹 브라우저에서의 간단한 스크립트 작성용 언어로 시작했으나, 현재는 Node.js덕분에 서버 측 프로그래밍까지 확장되어 전 세계적으로 가장 널리 쓰이는 프로그래밍 언어 중 하나로 자리잡았다. JavaScript의 가장 큰 장점 중 하나는 JavaScript는 동적 언어기 때문에 개발자의 머릿속에서 구현하고 싶은 프로그램이 빠르게 프로토타이핑 된다는 것이다. 

**동적(dynamic) 언어란?**

> 런타임에 타입(자료형)이 결정되는 언어를 의미한다. 즉 dynamic type checking을 통해 명시적인 타입 annotation이 존재하지 않는다. 

**짚고 넘어가는 김에 정적(static) 언어도 짚고 넘어가자면,**

> 컴파일시에 타입(자료형)이 결정되는 언어를 의미한다. 코드 작성 시 명시적으로 자료형을 지정해줘야 한다. **우리가 이야기하는 TypeScript가 정적 언어에 속한다.**

또한 다양한 플랫폼(web, app, server)에서 사용할 수 있는 생태계가 갖추어져 있는 언어라는 점, 단일 스레드 기반의 이벤트 루프이기 때문에 비동기 작업을 통한 자연스러운 동시성을 지원한다는 점을 중요한 시스템적 특성으로 뽑을 수 있다. 

이러한 JavaScript의 한계는 "**거대한 프로젝트에서는 프로그램이 개발자의 의도대로 동작하는지 확인하기 힘들다**" 라는 것이다. 이에 대한 원인는 크게

1. Static type checking의 부재
2. 1번의 문제로 인한 값이 연산 과정에서 계속 바뀐다는 점, 비동기 함수 호출과 독특한 런타임 

으로 뽑을 수 있다. 

TypeScript가 등장한 근본적인 원인은 1에서 찾을 수 있다. 이러한 문제는 동적 언어의 특성에서부터 기원하였다. Dynamic(동적) 언어의 특성상 type annotation을 요구하지 않기 때문에, 런타임 환경에서 잦은 버그가 발생한다. 즉 언제 나도 모르게 형변환이 되어 있을 수도 있고, 이러한 버그가 발생했을 때 인터프리터 언어 특성상 그런 버그들을 찾는 것 조차 쉽지 않다. 컴파일 과정이 없기 때문에 에러를 출력하지 않고 바로 실행되기 때문이다. 

**TypeScript는 이런 단점들을 보완하기 위해 JavaScript 위에 static type checking을 얹었다.** 프로그래밍 언어에 type system이 있음으로써 개발자는 자연스럽게 프로그램에 구조가 생기고, 문서화하는 효과가 생긴다. 또한 프로그램에 존재하는 타입 에러를 런타임 전에 검출하여 디버깅할 수 있게 된다.

**즉 TypeScript를 써야 하는 이유를 한줄로 정리하자면 다음과 같다.**

1. Static type checking을 지원하는 정적 언어이기 때문에 타입 안정성이 높아 type으로 발생하는 버그를 예방할 수 있다.
2. 객체의 필드나 함수의 매개변수로 들어오는 값들의 type을 지정해 주기 때문에 더 나은 개발자 경험과 코드 퀄리티 향상을 기대할 수 있다.

3. TypeScript는 JavaScript의 superset이기 때문에, 100% 호환이 가능하고 그 외의 OOP 패턴을 제공한다.

![typescript](../../../assets/images/typescript/nestjs/typescript.png)



## TypeScript의 단점

장점이 있으면 단점 또한 명확히 존재한다. 

첫째, **초반 환경설정의 어려움이 존재한다**. TypeScript는 high-level 언어기 때문에 해당 코드를 low-level로 변환하는 과정이 필요하다. 이러한 성질 때문에 TypeScript를 '**컴파일 언어(Compiled Langugage)**'라고 부른다. 즉 TypeScript는 tsc 컴파일러를 통해 JavaScript로변환되어 사용한다. 

> 대표적으로 JS는 인터프리터 언어로 별도의 컴파일 과정이 존재하지 않는다고 알려져 있으나, 엄밀히 말해 JS에서도 컴파일 단계가 존재하며 JS를 해석하고 실행하는 V8 엔진은 JS코드를 최적화하기 위해 컴파일 단계를 거친다. 

> TypeScript의 컴파일 단계는 다음과 같다. 
>
> 1. 타입스크립트 소스코드를 타입스크립트 AST로 만든다. (tsc)
> 2. 타입 검사기가 AST를 확인하여 타입을 확인한다.(tsc)
> 3. 타입스크립트 AST를 자바스크립트 소스로 변환한다. (tsc)
> 4. 자바스크립트 소스코드를 자바스크립트 AST로 만든다.(런타임)
> 5. AST가 바이트 코드로 변환된다.(런타임)
> 6. 런타임에서 바이트코드가 평가되어 프로그램이 실행된다.(런타임)
>
> 보통 나는 프로젝트 파일을 실행할 때, tsc로 컴파일한 뒤 node.js 런타임으로 실행한다.

TypeScript는 다음과 같은 과정을 거쳐야 하기 때문에 컴파일 시간이 길고, 이런 컴파일을 위한 환경 세팅(tsconfig)이 비교적 복잡하다. 

추가적으로 TypeScript는 주로 타입 설정을 위해 탄생한 언어인데, 인터넷에서는 종종 “타입 설정이 복잡하다”, “코드의 가독성이 떨어진다” 등과 같은 단점이 함께 언급되곤 한다. 하지만 이런 단점들은 나에게는 쉽게 이해되지 않는다. TypeScript라는 언어의 탄생 배경이 이러한 타입 설정들이 코드의 안정성과 예측성을 높여주기 위함인데, 타입 설정이 복잡해 보일 수 있어도 그로 인한 코드의 안정성과 명확성이 주는 장점이 더 높다고 생각하기 때문이다. 



다음 번에는 TypeScript 기반의 서버 개발 프레임워크인 NestJS에 대해 깊이있게 다뤄보도록 하겠다.



## REF

- [Typescript-kor docs](https://typescript-kr.github.io/pages/tutorials/ts-for-the-new-programmer.html)
- [우리가 TypeScript로 갈아탄 이유](https://brunch.co.kr/@redwit/1)
- [[Typescript] 타입스크립트의 컴파일 과정](https://velog.io/@kimmainsain/Typescript-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-%EC%BB%B4%ED%8C%8C%EC%9D%BC-%EA%B3%BC%EC%A0%95)

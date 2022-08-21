---
title: "내가 추구하는(추구하고싶은) 코드와 방향성"
date: 2022-08-21 12:00:00 +1830
categories: develop
---

무언가 인사이트를 제공하고자 하는 글이 아니라 생각을 정리하기 위한 글이다.

프론트엔드 직군에서 처음 개발을 시작한지 약 2년 정도가 되었다.\
소프트웨어를 길들이기 어렵다고 느낀다. 아무리 단순한 클라이언트 어플리케이션이라도 다음과 같은 부분이 필연적으로 발생한다.\
현재까지 내가 개발한 소프트웨어는 대부분 아래와 같은 구조로 도식화 되어있다.

1. 인프라 계층에서 데이터를 받아오고/변경하고 
2. 그 데이터의 스냅샷을 어디엔가 보관한다.
3. 현재의 스냅샷을 통해 사용자 UI 를 제공한다.
4. 사용자가 UI 를 동작하여 이벤트를 일으키면 1 번을 수행한다. 

하나 하나가 어렵고 무거운 주제라고 느낀다.
한가지 주제를 성숙하게 구현해내는것도 쉬운 일이 아니며 각 주제가 어우러지며 소프트웨어의 복잡성이 늘어나며,
잘 길들이지 못한 복잡성을 괴물이 되어 내 발목을 잡는다. 나를 기민하게 움직이지 못하게 한다.
무엇이 문제이고 무엇을 추구해야하는지, 무엇을 공부해야 하는지 글로 한번 정리해야할 필요가 있다고 느낀다.

## 단위를 작게 만들고, 이름을 붙이고, 하나의 단위가 많은 일을 하지 못하게 하기

코드를 목적에 맞게 작동하게 만드는건 쉽다. 요구사항을 읽고, 요구사항 대로 데이터를 분기하고, 반복하고, 출력한다.
요구사항은 글로 존재하고, 저마다의 히스토리가 있고, 하고자 하는 목적이 있다.
요구사항은 실행가능한 코드가 되는 순간 주의를 기울이지 않으면 순식간에 암호화 된다.
분기하고, 반복하고, 출력하는 코드를 읽고 복호화하여 요구사항을 떠올리는것은 쉽지 않다.
때문에 분석하기 어려운 코드는 다음과 같은 끝없는 의문을 갖게한다.

* 이 변수는 무엇일까? 무슨 목적으로 저장할까?
* 이 부분에서 왜 분기할까? 이 조건의 의미는 무엇일까?
* 이 반복문의 목적은 뭘까?
* 이 부분을 고치면 다른 로직의 영향도가 없다고 확신할 수 있을까?
* ... 더 이해하기 쉽게 구현 할 수는 없었을까?

그렇다면 왜 코드를 해석하기 어려워지는가?

* 한번에 읽고 해석하여야 하는 단위가 크다.
* 이 기능/이 변수가 무엇인지 네이밍으로 설명해주지 못한다.
* 추상화 수준이 낮고 해당 코드가 무엇을 하는지 하나하나 떠올려가며 머릿속으로 연산해야한다.
* 코드가 입/출력 으로 한정 지어져 있지않고 부수 효과가 많다.

다행인 점은 이러한 문제를 해결하는 방법은 여러 도서와 훌륭한 아티클 등을 통해 어느정도 가닥을 잡아나갈 수 있다는 것이다.
네이밍을 좀 더 세심하게 작성할 수 있을것이고, 단위를 작게 관리하는 방법에 대해 공부해볼 수 있겠다.
실천은 어렵지만 다음과 같은 키워드들을 통해 공부해나가다보면 개선될 수 있겠다고 느낀다. 
공부할 수 있는 책도 많다. 리팩토링2, 클린코드 등을 통해 좋은 코드의 지표에 대해 배우고, 이를 실천하는데 도움을 줄 수 있는
여러 프로그래밍 페러다임들과 우수사례들을 공부하다보면 두려움이 사라질 것으로 느낀다.

슬프게도 길들일 방법도 아직 가닥을 잡지못한 무서운 괴물들이 많이 존재한다.
큰 단위와 큰 단위가 서로 결합될 때가 그 때 이다.

## 공룡 간의 결합은 어떻게 다루어야 하는가?

우리는 특정한 기준에 따라 소프트웨어를 분리하고 나누고, 다른 계층을 모르게 하고 저마다의 역할을 수행하게 한다.\
가령 프론트엔드 진영에는 `MVC`, `MVVM`, `Flux`, `BLoC` 등 수많은 분리하고 나누는 주제에 대한 방법론이 있다.
추구하는 공통점은 분명하다 생각한다.

* 각 계층을 나누어 각 단위에 맞는 역할을 하도록 구현하여 전체 구조를 파악하기 쉽게한다.
* 각 계층을 서로를 깊게 알지 않는다는 전제하에 협력한다. 서로의 논리가 엃히지 않는다.
* 보편적인 방법론을 채택한다면 타인에게 이해시키기 쉽다. "이 앱은 MVC 패턴으로 구성된 앱입니다."
* 동료들이 이러한 규칙을 준수하며 작성한다. 모두의 코드가 동일한 기준점을 갖기에 파악하기 쉬워진다.

내가 두렵다고 느끼는 점을 이러한 패턴들이 실무의 영역으로 오게되면 단순히 개념으로 존재한다는게 아니라 
수많은 도구들을 수반하는 거대한 공룡 처럼 보인다는 것이다.

가령 `Flux 패턴` 을 구글에 검색해보면 프론트엔드 관련된 아티클이 많이 나온다.
눈여겨 볼 점은 redux, react 등 키워드들을 수반한다는 점이다.

![google-flux](/assets/img/google-flux.png)

하나하나 읽다보면 이런 생각이 든다.

* redux 는 전역 상태 관리 도구이다. 전역 상태 관리는 곧 Flux 인가?
* 도구 없이 달성 할 수는 없는가?
* Flux 패턴으로 어플리케이션을 구현하고 그에 따른 도구를 수반하는것이 보편적인 프론트엔드 개발인가?

가령 redux-saga 를 통해 우아하게 action - dispatch - store 계층을 만들었다 가정해보자.\
해당 계층에는 자연스레 비즈니스 로직들이 모일 것이다.\
가령 커머스 서비스라면 장바구니에 아이템 추가, 아이템 제거, 현재 장바구니에 추가된 아이템의 금액 합계 계산 등 주제들이 모일 것이다.

이러한 주제들은 핵심 비즈니스 로직이다! 가장 소중한 것이고 가장 오류가 있어서는 안되는 계층이다.\
비즈니스 로직은 여러 어플리케이션에서 반복된다. 우리가 다루는 도메인에 특화되어있고 우리의 코어 로직이다.

위화감은 우리의 핵심 비즈니스 로직이 redux 등 도구에 결합된단 점이며 위화감은 테스트를 작성하면서부터 발생한다.

테스트 코드를 작성할때 모킹을 하는 이유는 자명하다. 테스트하고자 하는 코드가 **무언가에 의존하고 있으며 해당 의존성을 테스트가 가능한 형태로 주입해줘야 테스트가 성립하기 때문이다.**
우리의 비즈니스 로직은 도구와 결합되었다. 테스트 코드 또한 도구와 결합된다.\

내가 두렵다고 느끼는 점은 우리가 이 도구를 더이상 사용하지 않겠다는 결정을 하게될때 이다.\
그 날이 왔을때 비즈니스 로직을 도구로 부터 문제없이 구해낼 수 있는가? 테스트 코드는 처음부터 다시 작성되어야하나?
애초에 이런일이 벌어지기전에 비즈니스 로직과 도구의 커플링을 억제할수는 없었을까?

## React 의 참을 수 없는 존재의 무거움

react 는 무엇일까. 사용자 인터페이스를 만들기 위한 JavaScript **라이브러리** 이다.\
내가 생각하는 리액트의 핵심 가치는 **현재 데이터의 상태에 맞게 개발자가 큰 노력을 안들이고도 DOM 을 업데이트 해준단 것** 이다.\
이 도구는 현재 프론트엔드 개발의 큰 비중을 차지 하며 이 도구를 익히는 것은 2022 년 현재 필수적인 소양 중 하나이다.

다만 내가 두려움을 느끼는 점은 리액트가 단순히 "사용자 인터페이스를 만들기 위한 JavaScript **라이브러리**" 로 쓰이고 존재하는가 이다.\
내가 바라보는 리액트는 라이브러리가 아닌 하나의 거대한 공룡 프레임워크 이다. 이 공룡으로 대형 어플리케이션을 구현하려면 redux, react-router-dom, react-query 등 수많은 친구들을 수반하며
자연스레 우리가 지켜야하는 핵심 비즈니스 로직들을 흡수하고 결합한다.

그리고 자기 소개를 한다. "나는 react 기반 어플리케이션 입니다."

컴포넌트와 비즈니스 로직의 결합도를 낮추려면 custom hook 등을 사용하라고 한다. 커스텀 훅에는 `useState` 등 수많은 리액트 의존성 훅들이 들어간다. 커스텀 훅이기에 이름도 `use*` 로 짓는다.
커스텀 훅으로 작성된 비즈니스 로직은 곧 리액트이다. 테스트하려면 모킹을 해줘야하며, 리액트가 없으면 동작할 수도 없고 성립할 수도 없다.

내가 의구심을 가지는 점은 이게 미래의 우리 어플리케이션을 위해 옳은 방법인가 하는것이다.

## 어디서든 재사용 가능한 형태로 코드를 보호하고 싶다.

행복한 상상을 해보자면 다음과 같은 주제들이 언어 이외의 어떠한 의존성도 지니지않고 독립적인 개체로 존재하는 어플리케이션을 만들고 싶다.\
여기서부터 서술할것은 명확한 방법이 떠오르지않았고 아직 공상으로서만 두루뭉술하게 존재한다.

* 장바구니 담기 등 핵심 비즈니스 로직
* 서버 api 를 호출하거나 하는 외부세계와 소통하는 액션 코드들
* 데이터의 스냅샷을 UI 계층에서 바로 렌더링할 수 있도록 관리하는 레포지토리
* 위 역할을 잘 수행하도록 도와주는 잘 추상화 된 계산 관련 함수

어떠한 외부 의존성도 지니지않는 순진무구한 클래스가 있고, 
다른 계층에서 이 클래스를 사용할때 몇가지 꼭 필요한 의존성을 내장하는것이 아닌 외부로 부터 건네받아 인스턴스를 만든 후
사용부에서는 public 으로 공개된 메서드를 호출하는것이다.
인스턴스의 상태변화는 구독 패턴 등 모종의 방법을 통해 react 등에 알려준다.

현재 이러한 목표에 현재 가장 근접하다고 보는 키워드는 `클린아키텍처`와, `BLoC 패턴`이다.
BLoC 패턴 은 아직 깊게 공부해보지 못했고, 클린 아키텍처란 책에서 강조하는 어니언 아키텍처는 아직 이해하지 못했다.
엔터티, 유스케이스 등은 아직 대단히 모호한 개념으로 내 머릿속에 존재한다.

객체지향 언어나 프로그래밍을 배워보면 어떨까도 생각한다. 객체 지향 언어 진형에서는 이러한 주제에 대한 고민과 논의가 보다 많이 이루어지는것 같다고 느낀다.\
현재로서 바라는 것이 있다면 이러한 목표에 어느정도 도달한 코드를 지닌 프론트엔드 어플리케이션을 뜯어보고 분석해보고 설명을 듣고 싶다.\
어느정도 뒤 시점에 다시 이 글을 읽었을 때 "그땐 이랬었지. 지금은 과거의 나에게 이에 대해 잘 설명해줄 수 있을텐데" 하는 시점이 왔으면 한다. 


















---
title: "웹 쇼핑몰 검색 구현 하기 1. : 요구사항 분석편"
date: 2021-04-29 22:00:00 +0900
categories: frontend
---

## 쇼핑몰 검색 기능을 구현해보자.

쇼핑몰 검색 폼을 구현해보자. 이를 통해 검색 기능의 구성요소들이 어떠한 역할을 해야하고 어떠한 방향으로 구현하는 것이 보다 확장성이 있을지
정리 해보고 싶다. 심화된 기능은 최대한 배재하고 핵심적인 기능을 프론트엔드 구현하는 것에만 집중해볼 것이다.

다음과 같은 순서로 진행하여 볼 것이다.

1. 어떠한 기능이 필요할 지 요구 사항을 분석한다. 
2. 어떠한 구조로 구현하는 것이 적합한지 숙고한다.
3. 자바스크립트 프레임워크를 사용하여 검색 기능을 구현한다.
4. 결과물을 회고하며 결과에 대해 정리해본다.

이 포스트 에서는 위 단계 중 1단계에 집중하여 어떠한 요구사항이 적합할 지 요구사항을 만들어보도록 하자.

### 어떠한 검색 기능이 쇼핑몰 상품 검색에 있어 적합한가?

내가 생각하는 상품 검색 기능이 제공하여야 하는 필수적인 기능은 아래와 같다.

* 키워드 우선으로 원하는 상품을 검색해낼 수 있어야 한다.
* 사이즈 별, 색상 별, 브랜드 별 로 찾아볼 수 필터 기능을 제공한다.
* 검색 조건에 해당하는 결과들을 최대한 빠르게 노출한다.

우선 이 기능을 충족하며 훌륭히 수행하고 있는 롤 모델을 찾아보자. 필터 기능을 적극적으로 활용하고 있는 서비스가 적합할 것이다.
내가 참고한 것은 현재 서비스 되고 있는 네이버 쇼핑의 모바일 웹 검색 기능이다.

<!-- 네이버 쇼핑 캡쳐 이미지 첨부 -->

네이버 쇼핑의 검색 기능을 살펴보자. 내가 필요하다고 느낀 사항을 충족하며 검색 조건이 변경될때마다 즉시 검색 결과를 반영한다.
또한 아래와 같은 힌트를 얻어 볼 수 있다.

### 서비스 중인 기능을 분석하여 보자.

네이버 쇼핑에서 검색기능이 어떻게 동작하고 있는지 살펴보자.
가령 '코트' 를 검색하였을 때 코트에 대한 검색 결과가 즉시 노출된다. 또한 코트에 해당하는 필터들이 하단에 노출된다.

화면적인 요소말고도 눈여겨보아야 할것은 어떻게 이 기능이 실제 동작하고 있는가이다.

* 필터 된 항목들은 URL 에 쿼리 문자열로 포함되게 된다.
* 필터가 변경되면 HTTP GET 요청을 통해 해당하는 결과들을 즉시 요청하여 응답 받는다.
* 새로 고침을 하거나 하여 동일한 URL 로 접근하면 동일한 검색 조건으로 검색이 가능하다.

여기까지는 내가 생각하는 검색 기능의 틀에서 크게 벗어나지 않는 것 같다.
 
눈에 띄는 한가지 편의적인 기능이 포함되어 있는데 검색 키워드에 따라 필터가 바뀌는 기능이 포함되어 있다.

'아름다운 의류' 등 추상적인 키워드로 검색하면 필터 기능이 노출되지 않는데, '코트' 등 필요한 상품군을 키워드에 정확히 제시하면
'코트' 에 맞는 필터가 등장한다. 
이 기능은 아마도 다음과 같이 동작할 것이다.

1. 클라이언트에서 검색 키워드를 통해 GET 요청을 보낸다.
2. 서버측에서 검색 키워드를 분석하여 어떠한 필터가 적합할지 클라이언트에 필터 데이터 응답을 내려준다.
3. 클라이언트에서 이를 화면에 렌더링하여 사용자에게 필터 기능을 제공한다.

이러한 방식으로 동작할 것임을 염두해둔다면 이렇게 똑똑한 API 가 현재 없더라도, 어느정도 이러한 API가 있다는 전제하에 구현이 가능 할 것이다.

### 최종 정리

위의 분석을 통해 우리가 구현해야할 기능은 다음과 같다.

* 사용자는 키워드를 통해 상품을 검색한다.
  * 키워드에 맞는 적합한 필터가 있다면 이를 사용자에게 표시한다.
  * 필터가 변경되면 즉시 이에대한 검색 결과를 사용자에게 표시한다.
  * 필터된 검색 조건은 URL 에 쿼리 문자열로 저장한다.
* 해당하는 상품에 대한 검색 결과를 노출한다.
  * 키워드 기반으로 검색한다.
  * 검색 결과는 검색 조건이 변경될 때마다 즉시 반영한다.
* 검색 조건을 공유할 경우를 가정하여야 한다.
  * 쿼리 문자열이 포함된 URL 로 접근하였다면, 해당하는 검색 조건을 표시한다.
  * 가능한한 쿼리 문자열에 포함된 필터의 조건 또한 그대로 보여준다. 

요구사항 정리가 완료되었다. URL 을 통해 특정 검색 조건을 이미 가진채로 접근하였을 때를 처리하는것이 조금 복잡할 것으로 예상되나 
우선 분석된 스팩 그대로 구현해보도록 하자.

 

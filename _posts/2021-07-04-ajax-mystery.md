---
title: "Synchronous ajax 요청에 대해"
date: 2021-07-04 22:00:00 +0900
categories: frontend
---

## Asynchronous call

서버와 클라이언트와 통신은 모두 비동기 요청으로 이루어진다.
서버가 언제 응답을 줄지 알 수 없기 때문이다.

때문에 개발을 할때에서는 서버로 부터 응답이 온 후에(비동기) 다음 로직이 실행될 수 있도록 콜백, 프로미스 등 여러가지 처리를 한다.
가령 다음 코드처럼 말이다.

``` javascript
async getData() {
  const data = await api.fetchData() // await. 응답이 올때까지 기다려.
  console.log(data); // 응답이 오면 data를 표시해줘.
}
```

이러한 비동기 요청의 응답을 기다린 이후, 다음 코드를 실행하는 패턴이 나에게는 당연하다고 여겨지는 패턴이다.
이러한 http call 을 처리해주는 jQuery.ajax 에 대해 간단히 살펴보자.

## jQuery.ajax

[jQuery.ajax : zerocho](https://www.zerocho.com/category/jQuery/post/57b1a48f432b8e586ae4a973) 

## jQuery.ajax.async = false

레거시 프로젝트에서 api 요청을 할때에는 jQuery.ajax 를 사용하곤 한다.

이 프로젝트는 fetch, axios 등을 사용하지않고 jQuery.ajax 를 통해 api 요청을 웹핑한다.\
사용부에서 이해가 안가는 부분을 발견하였는데 다음과 같다.

``` javascript
getData() {
  const data = api.fetchData() // 어..? 비동기 처리를 안하네..?
  console.log(data); // 아니 비동기 처리를 안했는데 어떻게 이 시점에 response 를 받을 수 있지??
}
```

이는 여태까지 써왔던 패턴과 사뭇 다른데, 이러한 패턴이 가능한 이유를 풀어보면 다음과 같았다.\
async 란 $.ajax 옵션을 사용하는것이다.

``` javascript
plainSyncCall() {
  const res =  $.ajax({
    method: 'GET',
    url: 'http://localhost:3004/posts/1',
    async: false // 이 녀석이 범인이다.
  })
  console.log(res.responseJSON)
}
```

### 이게 어떻게 가능한데?

[Synchronous and asynchronous requests : MDN](https://developer.mozilla.org/ko/docs/Web/API/XMLHttpRequest/Synchronous_and_Asynchronous_Requests)

`XMLHttpRequest` 는 요청을 동기로 처리할지 비동기로 처리할지 정할 수 있는 패터미터가 있**었**다.
세번째 인자가 그것인데 아래와 같이 xhr 을 동기식으로 요청할 수 있다.

``` javascript
var request = new XMLHttpRequest();
request.open('GET', '/bar/foo.txt', false); // 세번째 인자 async: false
```

서버의 응답이 올때까지 javascript 의 다음 행을 실행하지 않는것이다.
MDN 에서는 이를 이렇게 표현하고 있다. 

> Synchronous requests block the execution of code which causes "freezing" on the screen and an unresponsive user experience

> 동기 요청은 코드 실행을 차단하여 화면이 “얼어 붙어” 버리고 응답하지없는 듯한 사용자 경험을 만듭니다.

### 이는 안티 패턴인가?

모든 ajax 요청을 동기적으로 처리한다면 개발자는 더이상 비동기 요청을 처리하는 케이스에 대해서 생각하지 않아도 될지 모른다.

이 방식으로 개발하게 된다면 마치 내가 구현하는 서비스에 비동기적 요청이 일체 없는듯한 착각을 불러일으킬것이다.
 
허나 이러한 xhr 요청을 하거나, `jQuery.ajax.async = false` 를 사용하면 브라우저 콘솔에서 다음과 같은 경고문을 볼 수 있다.

> [Deprecation] Synchronous XMLHttpRequest on the main thread is deprecated because of its detrimental effects to the end user's experience. For more help, check https://xhr.spec.whatwg.org/.

> [Deprecation] 메인 스레드의 동기 XMLHttpRequest 는 최종 사용자의 경험에 해로운 영향을 미치기 때문에 더 이상 사용되지 않습니다. 자세한 도움말은 https://xhr.spec.whatwg.org/를 확인하세요.

그럼 왜 동기적 요청 패턴을 사용하면 안되는가??

'사용하면 브라우저 콘솔에 Deprecation warning 이 뜨니깐요..' 라는 대답은 근본에 비켜나간 대답인것같고 내가 느끼기엔 조금 멍청해보인다.



그래서 나는 이것에 대해 타당한 근거를 제시하는 여러 아티클을 찾아보았으나, 
안타깝게도 '이러이러해서 안티패턴이구나!' 라고 느껴지는 아티클을 찾지못하였다.  

대부분 다음 코드 실행을 의도치않게 중지시킬 수 있기때문에 사용하면 안된다는 내가 느끼기에는 조금 모호해보이는 대답이 부지기수였다.

### 나의 생각

다시 처음으로 돌아가서 맨 처음 살펴보았던 async await 패턴을 보자.

``` javascript
async getData() {
  const data = await api.fetchData() // await. 응답이 올때까지 기다려.
  console.log(data); // 응답이 오면 data를 표시해줘.
}
``` 

이 코드의 목적성은 `jQuery.ajax.async = false` 나 `Synchronous XMLHttpRequest` 와 근본적으로 동일하다.
 응답이 온 이후에 응답값을 표시하기를 바라기 때문이다.
 
다만 위 동기적 api 콜들과 한가지 다른점은 다음과 같다.

1. 개발자가 이 코드가 비동기 요청이라는것을 명확하게 인지하고 동기적으로 처리하고있디.
2. 다른 개발자가 이 코드를 보더라도 비동기요청을 동기적으로 처리한다는 것을 명확하게 알 수 있다.

만일 jQuery.ajax api 웹핑을 할때 `async = false` 를 기본값으로 두고 이 이름을 api 가 아닌 **전혀 비동기임이 연상되지않는 네이밍**으로 지었다고 생각해보자.

가령 웹핑시 이름을 `data` 라고 지었다면 다음과 같은 코드가 될것이다.

``` javascript
// 사용자의 프로필을 서버요청을 통해 받아와 렌더링 하는 코드 

displayProfile() {
  const id = data.getId() // 'id: foo'
  const profile = data.getProfile({ id })
  this.renderProfile(profile)
}
```

`data.getId`, `data.getProfile` 이 비동기 요청임에도 코드상에 전혀 들어나지 않는다.

의도에 맞게 실행될 것이지만, 데이터를 서버에서 받아오는지, 스토리지에서 가져오는지, 인스턴스에서 가져오는지가 코드상에 전혀 들어나지않는다.

이는 추후 특정 응답에 문제가 있거나, 코드를 해석하고자 할때 큰 혼란을 불러일으킬 것이다.

어느 부분이 비동기이고, 어느 부분에 브라우저가 `await` 를 하고있는지를 혼탁하게 만든다.

이러한 이유때문에 목적성은 같더라도 callback, promise, async, await 등으로 비동기 처리를 하는 것이라고 생각한다.

이러한 일반적인 비동기 요청 핸들링은 동기적 코드와 비동기적 코드를 구분지어주며 **브라우저가 기다릴 부분과 기다리지 않아도 될 부분을 명확히 분간하여 개발자에게 인지시켜준다.**

때문에 Synchronous XMLHttpRequest 가 안티패턴으로 정의되었고, 이러한 패턴이 지양되어야한다는 의견이 지배적인것이 아닌지 생각해본다.

### 추가로 알아보고 싶은 주제

jQuery.ajax 에서 병렬 요청이 가능한가요? 

https://sharepoint.stackexchange.com/questions/263675/how-to-use-promise-all-and-foreach-to-make-ajax-call-per-array-item



 



 



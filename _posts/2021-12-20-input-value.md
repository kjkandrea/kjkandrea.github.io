---
title: "폼 데이터 양방향 바인딩은 항상 필요한가?"
date: 2021-12-20 22:00:00 +0900
categories: frontend
---

# 폼 데이터 양방향 바인딩은 항상 필요한가?
 
우리는 유저에게 입력값을 받아 입력된 값을 저장하거나 수정하고자 할때 Form 을 사용한다.

React, Vue 등으로 Form 을 다루고자 할때에는 해당값을 바인딩해서 제어하고 싶어하는 경향이 있다.
프레임워크의 방식대로 사고하다보면 자연스레 [해당 값을 상태로서 제어하고 싶어하기 때문](https://ko.reactjs.org/docs/forms.html#controlled-components) 이다.

가령 `nickname` 을 입력받는 Form 을  React 로 Controlled 하게 작성해보자.

## Controlled Form

```jsx
const ReactForm = () => {
  const [nickname, setNickname] = React.useState() // 1. nickname 의 상태

  return (
    <form>
      <label>
        nickname:
        <input 
          type="text" 
          name="nickname"
          value={nickname}
          onChange= {e => setNickname(e.target.value)} // 2. change 마다 username 동기화
          />
      </label>
      <button type="submit">Join the room</button>
      
      {<!-- 3. username 상태가 동기화 되어 업데이트 된다. -->}
      <p>nickname is : {nickname}</p>
    </form>
  )
}

ReactDOM.render(<ReactForm />, document.querySelector("#app"))
```

1. 1번에서 nickname 의 상태를 생성하였고
2. 2번에서 입력값과 상태의 동기화를 위하여 이벤트를 생성하였다.
3. 3번은 불필요하나, 상태가 실시간으로 동기화된다는것을 보여주고자 추가하였다.

`nickname` 상태에 입력값이 항상 동기화 되므로 우리는 아래와 같이 submit 이벤트를 처리하여,
유저가 submit 을 발생시켰을 때에 nickname 을 원하는 바 대로 처리할 수 있다.
가령 다음과 같이 말이다.

``` jsx
const ReactForm = () => {
  const [nickname, setNickname] = React.useState()
  
  const handleSubmit = event => {
  	event.preventDefault()
  	alert(nickname) // 입력받은 nickname 으로 하고싶은 일을 한다.
  }

  return (
    // ...
  )
}
```

`handleSubmit` 을 통해 입력받은 nickname 값을 원하는 시점에 처리할 수 있다.

이제 반문해보자.

*`onChange` 를 왜 사용하였는가?*\
`nickname` 과 `input.nickname.value` 의 value 를 동기화 하기 위해서이다.

*`input.nickname.value`가 변경되는 시점에 `nickname` 이 왜 매번 동기화 되어야하는가?*\
`submit` 이벤트가 발생하는 시점에 `nickname` 이 준비되어 있어야 하기 때문이다.

위와 같은 이유 때문에 양방향 바인딩이 필요하다. 

위와 같은 양방향 바인딩은 React 의 세계에서는 자주쓰이나, 다음과 같은 비용을 지불해야 할 수 있다.

가령 입력을 받아야하는 필드가 추가되었고, 그에 따라 `nickname` 과 유사한 `phoneNo`, `age` 등 여러 상태들이 생겨났다고 가정해보자.
Form 하위에 `TextField` 등 서브컴포넌트가 생겨났다고 가정해보자.

`ReactForm` 는 특정 상태가 업데이트 될때에 해당 상태를 양방향 바인딩하기 위해서 리렌더를 시도할것이다.
리렌더가 필요하다면 React 는 diffing check 를 해가며 변경되어야 하는 요소만 DOM 에 업데이트 시킬 것이다.

성능을 챙기고 싶다면 개발자는 이를 도와주기 위해서 `useMemo` 등을 이용하여 각 `TextField` 가 
해당 `TextField` 에 해당하는 상태가 업데이트 될때만 리렌더링 하도록 하는 코드를 작성할 것이다.

위와 같은 작업은 필요한가? Submit 과 같이 사용자가 값을 제출하였을때만 각 필드들의 값이 필요할때에는 필요 없을 수 있다.

우리가 애써 React 의 상태로 관리하려 한 값은 이미 DOM 의  `input.nickname.value` 에 실시간으로 바인딩되어있다.

그렇다면 이제는 `uncontrolled` 하게 관리해보자.

## UnControlled Form

React 에서 소개하는 [비제어 컴포넌트](https://ko.reactjs.org/docs/uncontrolled-components.html) 는 요약해보면 상태를 만들지도 않고 관리하지도 않는것이다.
단지 해당 데이터가 필요할때에 해당 값을 읽어오면 된다. 

값이 바인딩된 nickname Form 을 아무런 상태도 없는 순진무구한 상태로 되돌려보자.

```jsx
const ReactForm = () => {
return (
  <form>
    <label>
      nickname:
      <input
        type="text"
        name="nickname"
      />
    </label>
    <button type="submit">Join the room</button>
  </form>
)}
```

그리고 useCallback 을 이용하여 submit 이벤트 핸들러를 작성한다.

```jsx
const ReactForm = () => {
  
const handleSubmit = React.useCallback(event => {
  event.preventDefault()
  alert('submit')
}, [])

return (
  <form onSubmit={handleSubmit}>
  // ...
  <form>
)}
```

그리고 `handleSubmit` 에서 input 에 접근하여 value 를 가져오면 된다.

현재 공식문서에는 `useRef` 를 사용하여 `input.nickname.value` 를 가져오나,
나는 훌륭한 Web API 인 [FormData](https://developer.mozilla.org/ko/docs/Web/API/FormData) 를 이용하여 value 를 가져올 것이다.

```jsx
const getFormData = formEl => {
  const formData = new FormData(formEl);
  
  const result = {};
  for (const [key, value] of formData.entries()) 
    Object.assign(result, { [key]: value });
  
  return result;
};

const ReactForm = () => {
  
const handleSubmit = React.useCallback(event => {
  const { nickname } = getFormData(event.currentTarget)
  event.preventDefault()
  alert(nickname)
}, [])

return (
  <form onSubmit={handleSubmit}>
  // ...
  <form>
)}
```

이제 nickname 을 입력 후 submit 을 동작하여보면 alert 에 입력한 nickname 값이 담겨서 출력되는것을 볼 수 있다.

## 정리

UnControlled Form 은 상태를 양방향 바인딩 하는 방식에 비해서 성능상의 이점과, 작성의 편의를 제공하나
입력 즉시 validation 을 실행하여 잘못된 입력값이 있음을 알려주고자하는 기능을 구현하고자 할때에는 적절 치 않을 수 있다.

하지만 무작정 모든 값을 프레임워크의 상태 값으로 관리하는것보다는 더 나은 선택 일 수 있다는것을 기억하자.

* 상태를 양방향 바인딩할때는, 양방향 바인딩이 꼭 필요한 행위인지 반문하여 보자.
* 양방향 바인딩이 필요없다면 양방향 바인딩을 하지 않으면 된다.
* UnControlled 하게 input 에 기본 값을 바인딩하고자 할때에는 `defaultValue` 를 사용하자.
* 복잡한 Form 을 UnControlled 하게 관리하고자 한다면 [react-hook-form](https://react-hook-form.com/) 모듈을 활용하는것도 좋은 대안이 될 수 있다. 
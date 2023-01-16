---
title: "최소단위의 컴포넌트"
date: 2023-01-16 12:00:00 +0900
categories: frontend
---

[아토믹 디자인](https://bradfrost.com/blog/post/atomic-web-design/) 이란 원자(atom)-분자(molecule)-유기체(organism) 순으로
최소 단위의 atom 컴포넌트를 조합하여 단계별로 나타내는 설계 기준이 있다.

최근 팀 내 프로젝트에서 이런 패턴으로 컴포넌트를 단계별로 구현하는 경험을 하였는데, 어려웠던 점은 최소단위인 atom 컴포넌트를 구현하는 방법이었다.\
최소단위의 컴포넌트는 HTML 엘리먼트와 닮아있다. 가령 버튼을 만든다고 가정해보자.

이 버튼은 특정 자바스크립트 이벤트를 트리거 하기 위한 버튼일 수도 있고, 양식을 제출하기 위한 버튼일 수도 있다. 특정 페이지로 이동시키는 링크를 제공하는 버튼일수도 있다.\
최소단위를 이루는 컴포넌트는 위와 같이 넒은 범위의 쓰임새에 대한 많은 가정을 해야하며 그 때문에 구현하기 쉽지않다.

특정 문제를 해결하는 도메인에 집중하는 서비스 개발과는 달리 라이브러리를 만들듯이 범용성을 갖추어야 한다.
React UI 라이브러리인 [머티리얼 UI](https://mui.com/)의 Button 컴포넌트의 API 를 살펴보자. clickable 한 요소가 수용할 수 있는 넒은 범위의 수많은 옵션(Prop)을 제공한다. 

![MUI Button Prop Options](../assets/img/mui-props.png)

대략 위와 같은 옵션이 존재한다. '어떻게 보여질 것인가' 를 정의하는 옵션이외에 대략 다음과 같은 옵션을 확인할 수 있다.

* component(as) : 어떤 ElementNode 로 렌더링할 것인가
* disabled : 클릭 이벤트를 비활성화
* href : 링크 기능으로 쓰일때에 이동 주소

문서내에 언급되진 않았지만 이 외 onClick, type 등 다양한 기본 옵션들이 존재한다. 이제 여러 사용케이스를 포괄하는 다형성을 지닌 버튼을 만들어보자.

## Button 을 만들어보자.

'어떻게 보여질 것인가' 를 제외한 몇가지 옵션이 구현된 버튼을 만들어보자.

```tsx
import React from 'react';

interface ButtonProps {
    children: React.ReactNode;
    onClick?: React.MouseEventHandler<HTMLButtonElement>;
}

export default function Button({onClick, children}: ButtonProps) {
    return (
        <button type="button" onClick={onClick}>
            {children}
        </button>
    );
}   
```

현재 상태로는 클릭 이벤트를 바인딩 할 수 있지만 type, disabled 등 `<button>` 이 지니는 기본 어트리뷰트를 제어할 수 없다.

`ComponentPropsWithoutRef`([참고](https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase#wrappingmirroring-a-html-element)) 를 이용하여 조금 더 범용적으로 타입을 정의해보자.

```tsx
interface ButtonProps extends React.ComponentPropsWithoutRef<'button'> {
  children: React.ReactNode;
}

export default function Button({children, ...rest}: ButtonProps) {
  return <button {...rest}>{children}</button>;
}
```

## reference

* [evan-moon : 타입스크립트와 함께 컴포넌트를 단계 별로 추상화해보자](https://evan-moon.github.io/2020/11/28/making-your-components-extensible-with-typescript/)
* [Create a Polymorphic Component with Typescript and React](https://scottbolinger.com/create-a-polymorphic-component-with-typescript-and-react/)
* [typescript-cheatsheets/react : Useful Patterns by Use Case](https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase)

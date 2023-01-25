---
title: "Tagged Union : 태그로 데이터들의 조합을 안전하게 구분하기"
date: 2023-01-25 12:00:00 +0900
categories: frontend
---

## 장바구니 클라이언트 만들기

클라이언트에서 데이터들을 다루다보면 특정 상태일때에만 유효한 필드, 유효하지 않은 필드가 존재한다.\
[태그된 유니온(Tagged Union)](https://en.wikipedia.org/wiki/Tagged_union) 을 사용하여 이를 구분하는 방법을 간단히 정리해보자. 

전자상거래에서의 상품을 타입으로 나타낸다면 대략 다음과 같을 것이다.

```ts
interface Product {
  id: string,
  name: string,
  featured_image: {
    url: string,
    alt: string
  },
  price: Money,
}
```

위와 같은 상품 데이터를 토대로 사용자에게 장바구니 리스트, 장바구니 상품의 구매가능 여부등을 제공하는 장바구니 클라이언트를 개발한다고 생각해보자.

![Cart Product List](/assets/img/cart_example.jpeg)

개발을 하다보니 **구매불가한 상품**에 대한 처리가 필요해졌다. Product 인터페이스에 해당 프로퍼티를 추가한다.

```ts
interface Product {
  id: string,
  name: string,
  featured_image: {
    url: string,
    alt: string
  },
  price: Money,
  status: 'ONSALE' | 'UNAVAILABLE',
  /** saleStatus UNAVAILABLE 일때의 구매 불가 사유 */ 
  unavailableReason?: string
}
```

status 를 추가하고 선택 속성(optional property) 을 통해 unavailableReason 를 추가했다.\
이에 따라 썸네일에 '구매 불가' 텍스트를 씌우고, 구매불가 사유를 표기해야한다. JSX 탬플릿을 다음과 같이 업데이트한다.

```tsx
<CartProduct>
  <CartProduct.FeaturedImage 
    image={product.featured_image}
    unavailable={product.status === 'UNAVAILABLE'} 
  />
  // 📌 problem.
  {
    product.unavailableReason !== undefined
    && <CartProduct.GuideText text={product.unavailableReason} color="warn" />
  }
  <CartProduct.Title text={product.title} />
  <CartProduct.Price price={product.price} />
</CartProduct>
```

어색한 부분이 보인다. `Product.unavailableReason` 은 `product.status === 'UNAVAILABLE'` 일 경우에만 존재하는 값이지만, `Product` 인터페이스는 `status` 와 `unavailableReason` 의 **연관성을 나타내지 못한다.**

때문에 주석상의 📌부분에서는 `product.status === 'UNAVAILABLE'` 과 별도로 `unavailableReason` 의 존재 유무를 검사하고 있다.

현재 상태로 특정 `status` 에 대한 프로퍼티가  더 추가되거나 한다면 `Product` 의 이러한 구분은 더욱 모호해질 것이다.\

## 불가능한 상태는 불가능하게

이러한 데이터의 경우 태그된 유니온 을 통해 속성과 속성간의 관계를 다음과 같이 명시적으로 나타내줄 수 있다.

```ts
interface BaseProduct {
  id: string,
  name: string,
  featured_image: {
    url: string,
    alt: string
  },
  price: Money
}

interface OnSaleProduct extends BaseProduct {
  status: 'ONSALE'
}

interface UnavailableProduct extends BaseProduct {
  status: 'UNAVAILABLE',
  unavailableReason: string
}

type Product = OnSaleProduct | UnavailableProduct 
```

이제 `status` 필드는 해당 Product 의 유형을 설명하는 태그 역할을 한다.\
이를 통해 status 와 unavailableReason 의 관계가 명확해졌기에 JSX 도 다음과 같이 status 중심으로 작성될 수 있다.

```tsx
<CartProduct>
  {() => {
    switch (product.status) {
      case 'ONSALE':
        return (
          <CartProduct.FeaturedImage image={product.featured_image} />
        );
      case 'UNAVAILABLE':
        return (
          <>
            <CartProduct.FeaturedImage image={product.featured_image} unavailable={true} />
            <CartProduct.GuideText text={product.unavailableReason} color="warn" />
          </>
        )
    }
  }}
  <CartProduct.Title text={product.title} />
  <CartProduct.Price price={product.price} />
</CartProduct>
```

## 정리

위와 같이 태그된 유니온을 통해 선언된 타입은 가능한 상태와 불가능한 상태를 나타내는 일종의 문서 역할을 한다.\
이처럼 태그만을 추가하는 간단한 방법을 통해 데이터를 보다 명시적으로 모델링하고 안전하게 다룰 수 있다.

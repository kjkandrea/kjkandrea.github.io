---
title: "상표 기법 : 매개변수에 제약 조건 만들기"
date: 2023-01-26 20:00:00 +0900
categories: frontend
---

프로그래밍을 하다보면 계산 함수들의 실행 순서에 따른 제약 혹은 특정 의미를 지닌 원시 타입만 허용하고 싶을 때가 있다.

첫번째 예시로 이진 탐색을 통해 특정 값의 존재여부를 리턴하는 binarySearch 함수를 만든다고 생각해보자.
이진 탐색은 선결 조건으로 정렬이 이루어진 목록 을 요구한다. 

```ts
function binarySearch<T>(sortedArray: T[], findItem: T): boolean {
  // 구현 생략
}
```

'정렬이 이루어진' 이라는 선결조건을 명시적으로 나타내고자 매개변수명을 sortedArray 로 힌트를 두었으나 실수로 인해 정렬되지 않은 리스트를 인자로 넘길 경우가 존재한다.

두번째 예시로 간단한 환율 계산 앱을 만든다고 가정해보자. 
이 앱이 수행하는것은 사용자에게 계산하고자하는 원 단위를 입력받아 현재 기준 환율에 맞게 달러로 변환하여 화면에 보여주는 것이다.
현재 기준 원을 달러로 반환해주는 currencyCalculator.wonToDollar 메서드가 존재하고, renderDollar 라는 달러를 인자로 받아 화면에 렌더링하는 함수가 있다고 가정해보자.

```ts
class CurrencyCalculator {
  //...
  wonToDollar(won: number): number {
    return this.getExchangeRate() * won
  }
}

function renderDollar(dollar: number): void {
  //...
}

const currencyCalculator = new CurrencyCalculator()

const dollar = currencyCalculator.wonToDollar(1_000)
renderDollar(dollar)
```

마찬가지로 renderDollar 의 dollar 매개변수를 두었으나, number 타입으로는 할당되는 값이 won 인지, 계산이 완료된 dollar 인지 제약을 둘 수 없다.
마찬가지로 개발자의 실수로 인해 실행 문맥이 변경되면 잘못된 결과가 계산될 여지가 있는 것이다.

## 상표(brand) 기법 을 통한 명목적(nominal) 타이핑

타입스크립트는 구조적 타이핑을 차용하지만 특성 사례에 이러한 제약을 두고싶은 경우 상표 기법으로 명목적 타이핑을 흉내낼 수 있다.

```ts
type Brand<T, K> = T & { __brand: K }
```

Brand 타입에 제너릭 매개변수 T, K 를 통해 구조적 타입에 `__brand` 를 부여함으로 명목적 타입으로 사용할 수 있다.

**여기에서 교차 타입으로 선언된 `__brand` 는 말그대로 명목적 타이핑을 위한 이름이다.** 런타임에서는 실제로 존재하지않으며 타입 시스템에서만 쓰이기에 런타임 오버헤드를 발생시키지 않는다. 

### binarySearch 매개변수에 제약 두기

이 타입을 사용하여 binarySearch 의 매개변수에 제약을 부여해보자. 

```ts
type SortedArray<T> = Brand<T[], 'sorted'> // T[] & { __brand: 'sorted' } 

function binarySearch<T>(sortedArray: SortedArray<T>, findItem: T): boolean {
  // 구현 생략
}
```

제약이 부여된 sortedArray 에 배열 리터럴로 배열을 넘기면 타입 에러가 발생하는 것을 확인할 수 있다.

```ts
binarySearch([3, 1, 2], 3);
/** 
 * 🚨 TS2345: 
 * Argument of type 'number[]' is not assignable to parameter of type 'SortedArray<number>'. 
 * Property '__brand' is missing in type 'number[]' but required in type '{ __brand: "sorted"; }' 
 */
```

이제 binarySearch 에 sortedArray 인자를 넘기기 위해서는 정렬된 배열임을 증명해야 한다.\
정렬함수를 거쳐 `SortedArray` 를 만들어내보자.

```ts
type SortedArray<T> = Brand<T[], 'sorted'> // T[] & { __brand: 'sorted' } 

function sortArray<T>(array: T[]): SortedArray<T> {
  return [...array].sort() as SortedArray<T>
}

const sortedNumbers = sortArray([3, 1, 2]) // number[] & { __brand: 'sorted' } 
console.log(sortedNumbers) // [1, 2, 3] 
```

sortArray 함수를 통해 `SortedArray` 상표가 부여된 sortedNumbers 는 비소로 안전하게 binarySearch 의 인자로 할당될 수 있다. 

### renderDollar 매개변수에 제약 두기

마찬가지로 Brand 를 통해 원시 타입 number 에 상표를 부여해보자.

```ts
type Brand<T, K> = T & { __brand: K }
type Dollar = Brand<number, 'dollar'> // number & { __brand: 'dollar' } 
```

wonToDollar 메서드가 `Dollar` 타입을 리턴하도록 변경하고,  renderDollar 함수가 `Dollar` 타입을 매개변수로 받도록 변경한다.

```ts
function renderDollar(dollar: Dollar): void {
  //...
}

class CurrencyCalculator {
  //...
  wonToDollar(won: number): Dollar {
    return this.getExchangeRate() * won as Dollar
  }
}

const currencyCalculator = new CurrencyCalculator()

const dollar = currencyCalculator.wonToDollar(1_000)
renderDollar(dollar)
```

이를 통해 currencyCalculator.wonToDollar 메서드의 리턴값만 인자로 할당 할 수 있도록 제약을 둘 수 있다.  

## 한계점 & 정리

상표 기법은 산술 연산을 거치면 상표가 없어지기에 수많은 산술 연산을 거치는 과정에 사용하기엔 무리가 있을 수 있다.

```ts
type Brand<T, K> = T & { __brand: K }
type Dollar = Brand<number, 'dollar'> // number & { __brand: 'dollar' } 

const dollar: Dollar = currencyCalculator.wonToDollar(1_000)
const millionDollar = dollar * 1_000_000 // number

renderDollar(millionDollar)
/**
 * 🚨 TS2345:
 * Argument of type 'number' is not assignable to parameter of type 'Dollar'.
 * Type 'number' is not assignable to type '{ __brand: "dollar"; }'.
 */
```

이와 같은 한계를 인지하고 상표 기법을 사용해야 한다.

이팩티브 타입스크립트는 상표 기법을 다음과 같이 요약한다.

* 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에, 값을 세밀하게 구분하지 못하는 경우가 있다. 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는것을 고려해야 한다.
* 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.

## reference

* [\[댄 밴더캄\] 이팩티브 타입스크립트 : 37. 공식 명칭에는 상표를 붙이기](https://effectivetypescript.com/)
* [\[toss.tech\] TypeScript 타입 시스템 뜯어보기: 타입 호환성](https://toss.tech/article/typescript-type-compatibility)
* [\[Michal Zalecki\] Nominal typing techniques in TypeScript](https://michalzalecki.com/nominal-typing-in-typescript/)

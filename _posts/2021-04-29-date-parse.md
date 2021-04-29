---
title: "new Date(s: DateFromISOString)"
date: 2021-04-29 22:00:00 +0900
categories: javascript, diary
---

최근 new Date() 관련 문제를 경험했는데, 버그의 내용은 서비스 내에 사용하는 캘린더 라이브러리가 Safari 브라우저에서 죽어있다는 것이였다.

당시 사용하던 [Tui Date Picker](https://ui.toast.com/tui-date-picker) 라이브러리를 래핑하고 있는 코드를 의심했고 과연 그 부분의 문제가 있었다. 문제는 해당 코드 뿐만아닌  코드베이스 전역에 문제 있었다.      

### Date.parse())

문제는 표준을 지키지 않은 코드에 있었다.
일자 데이터를 `YYYY-MM-DD HH:MM:SS` 형식으로 관리하고 있었고, 저 형식으로 전송된 응답을 아래와 같이 쓴 것이다.

``` javascript
const selectDate = '2021-04-29 21:00:00' // api response
const date = new Date(selectDate);

new TuiDatePicker({ date })
```

이것이 문제였다. 당장 chrome 이나 node 에 `new Date('2021-04-29 21:00:00')` 때려보면 `Date` 를 잘 반환 한다.

하지만 safari 는 Invalid Date 를 반환한다.  RFC2822 또는 ISO 8601 날짜를 나타내는 문자열 이나 UTC 기준의 숫자값 을 삽입해야 정상적인 Date 객체를 반환한다.
아래 처럼.

``` javascript
new Date(2011, 0, 1, 0, 0, 0, 0); // Sat Jan 01 2011 00:00:00 GMT+0900 (KST)
new Date('2012-01-26T13:51:50.417') // Thu Jan 26 2012 13:51:50 GMT+0900 (KST)
new Date('2012-01-26T13:51:50.417Z') // Thu Jan 26 2012 22:51:50 GMT+0900 (KST)
```

`YYYY-MM-DD HH:MM:SS` 를 라이브러리에 의존하지 않고 네이티브 new Date() 로 가져오고자 한다면 아래와 같이 함수를 만들어 두고 사용해 볼 수 있을것 같다.

``` typescript
function parseStringDateFormat(ymdhms: string): Date {
  const [_, y, month, d, h, m, s] = ymdhms.match(/^(\d+)-(\d+)-(\d+) (\d+)\:(\d+)\:(\d+)$/)
  return new Date(y, month - 1, d, h, m, s)
}

parseStringDateFormat('2011-07-15 13:18:52') // Fri Jul 15 2011 13:18:52 GMT+0900 (KST)
```

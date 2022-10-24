# 소개

RxJS는 observable sequence를 사용하여 비동기 및 event 기반 프로그램을 제작하기 위한 라이브러리입니다. 핵심 유형 중 하나인 [Observable](./guide/observable), satellite types (Observer, Schedulers, Subjects), `Array` 메서드 (`map`, `filter`, `reduce`, `every` 등) 에 의해 영감을 받아 비동기 event를 collection로 처리할 수 있게 합니다.

<span class="informal">RxJS를 이벤트용 Lodash 처럼 생각하세요.</span>

ReactiveX는 [Observer 패턴](https://en.wikipedia.org/wiki/Observer_pattern)과 [Iterator 패턴](https://en.wikipedia.org/wiki/Iterator_pattern)과 [컬렉션을 이용한 함수형 프로그래밍](http://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions)을 결합하여 event sequence를 관리하기 위한 아이디어를 제공합니다.

비동기 event 관리를 위한 RxJS의 본질적인 개념:

- **Observable:**  변경될 수 있는 value나 이벤트를 다루는 collection.
- **Observer:** Observable에서 전달하는 value를 처리하는 callback collection.
- **Subscription:** Observable의 실행. 주로 실행을 취소하는데 유용하다.
- **Operators:** `map`, `filter`, `concat`, `reduce` 등의 함수형 프로그래밍 표현을 가능하게 해주는 순수 함수.
- **Subject:** EventEmitter와 동등하다. value나 event를 다수의 Observer로 멀티캐스팅 할 수 있는 유일한 방법.
- **Schedulers:** 동시성을 제어하는 중앙집중식 dispatcher. `setTimeout`이나 `requestAnimationFrame`등의 연산이 일어날 때 작동함.

## 예시

일반적으로 event listener를 등록하는 코드:

```ts
document.addEventListener('click', () => console.log('Clicked!'));
```

RxJS를 이용해 observable을 만드는 코드:

```ts
import { fromEvent } from 'rxjs';

fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
```

### Purity

RxJS를 강력하게 만드는 것은 순수한 함수를 사용하여 value를 생성하는 기능입니다. 코드에서 에러가 덜 발생합니다.

일반적으로, state를 엉망으로 만드는 불순한 함수:

```ts
let count = 0;
document.addEventListener('click', () => console.log(`Clicked ${++count} times`));
```

RxJS를 이용해 state를 격리하는 코드:

```ts
import { fromEvent, scan } from 'rxjs';

fromEvent(document, 'click')
  .pipe(scan((count) => count + 1, 0))
  .subscribe((count) => console.log(`Clicked ${count} times`));
```

**scan**은 배열의 **reduce**와 동일하게 작동합니다. callback에는 시작 value가 필요하고, callback이 리턴한 value는 다음 callback의 value가 됩니다.

### Flow

RxJS에는 event가 observable에 의해 제어되도록 도와주는 전역 operator가 있습니다.

순수 JavaScript를 이용하여 초당 한 번만 클릭을 허용하는 코드:

```ts
let count = 0;
let rate = 1000;
let lastClick = Date.now() - rate;
document.addEventListener('click', () => {
  if (Date.now() - lastClick >= rate) {
    console.log(`Clicked ${++count} times`);
    lastClick = Date.now();
  }
});
```

RxJS 이용한 코드:

```ts
import { fromEvent, throttleTime, scan } from 'rxjs';

fromEvent(document, 'click')
  .pipe(
    throttleTime(1000),
    scan((count) => count + 1, 0)
  )
  .subscribe((count) => console.log(`Clicked ${count} times`));
```

다른 흐름 제어 operator는 [**filter**](../api/operators/filter), [**delay**](../api/operators/delay), [**debounceTime**](../api/operators/debounceTime), [**take**](../api/operators/take), [**takeUntil**](../api/operators/takeUntil), [**distinct**](../api/operators/distinct), [**distinctUntilChanged**](../api/operators/distinctUntilChanged) 등이 있습니다.

### Values

observable을 이용해 value를 변경할 수 있습니다.

순수 JavaScript를 이용해 클릭할 때 마다 마우스의 x좌표를 더하는 코드:

```ts
let count = 0;
const rate = 1000;
let lastClick = Date.now() - rate;
document.addEventListener('click', (event) => {
  if (Date.now() - lastClick >= rate) {
    count += event.clientX;
    console.log(count);
    lastClick = Date.now();
  }
});
```

RxJS를 이용한 코드:

```ts
import { fromEvent, throttleTime, map, scan } from 'rxjs';

fromEvent(document, 'click')
  .pipe(
    throttleTime(1000),
    map((event) => event.clientX),
    scan((count, clientX) => count + clientX, 0)
  )
  .subscribe((count) => console.log(count));
```

value를 다루는 다른 operator로는 [**pluck**](../api/operators/pluck), [**pairwise**](../api/operators/pairwise), [**sample**](../api/operators/sample) 등이 있습니다.

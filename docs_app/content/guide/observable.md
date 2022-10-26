# Observable

Observable은 여러 value를 지연된 push를 할 수 있는 collection 입니다. 다음은 누락된 부분의 보충설명을 위한 테이블:

|          | Single                                                                                                | Multiple                                                                                            |
| -------- | ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Pull** | [`Function`](https://developer.mozilla.org/en-US/docs/Glossary/Function)                              | [`Iterator`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) |
| **Push** | [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) | [`Observable`](/api/index/class/Observable)                                                         |

**예시:** 다음은 Observable이 subscribe된 직후(동기적으로) `1`, `2`, `3` 을 push하고, 1초 후 `4`를 push하고 완료되는 코드:

```ts
import { Observable } from 'rxjs';

const observable = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
```

Observable을 호출 후 value를 확인하기 위해, _subscribe_ 하는 코드:

```ts
import { Observable } from 'rxjs';

const observable = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

console.log('just before subscribe');
observable.subscribe({
  next(x) {
    console.log('got value ' + x);
  },
  error(err) {
    console.error('something wrong occurred: ' + err);
  },
  complete() {
    console.log('done');
  },
});
console.log('just after subscribe');
```

실행 후 콘솔에 출력되는 결과:

```none
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```

## Pull vs Push

_Pull_ 과 _Push_ 는 데이터의 _(Producer)생산자_ 와 _(Consumer)소비자_ 가 통신하는 서로 다른방법의 프로토콜 입니다.

**Pull?** Pull은 Consumer(소비자)가 언제 Producer(생산자)에게 데이터를 받을지 결정하는 시스템 입니다. Producer(생산자)는 언제 Consumer(생산자)에게 데이터가 전달 될 지 알 수 없습니다.

JavaScript의 함수는 Pull 시스템 입니다. 함수는 Producer(생산자)이고, 함수를 호출하는 코드가 "pulling"하여 _single_ value를 consume(소비) 합니다.

ES2015에서 다른 타입의 Pull 시스템인 [generator functions and iterators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) (`function*`)이 도입되었습니다. `iterator.next()`는 호출 시 iterator(Producer(생산자))로 부터 여러 value를 가져오는 Consumer(소비자) 입니다.

|          | Producer                                   | Consumer                                    |
| -------- | ------------------------------------------ | ------------------------------------------- |
| **Pull** | **Passive:** produces data when requested. | **Active:** decides when data is requested. |
| **Push** | **Active:** produces data at its own pace. | **Passive:** reacts to received data.       |

**Push?** In Push systems, the Producer determines when to send data to the Consumer. The Consumer is unaware of when it will receive that data.

Promises are the most common type of Push system in JavaScript today. A Promise (the Producer) delivers a resolved value to registered callbacks (the Consumers), but unlike functions, it is the Promise which is in charge of determining precisely when that value is "pushed" to the callbacks.

RxJS introduces Observables, a new Push system for JavaScript. An Observable is a Producer of multiple values, "pushing" them to Observers (Consumers).

- A **Function** is a lazily evaluated computation that synchronously returns a single value on invocation.
- A **generator** is a lazily evaluated computation that synchronously returns zero to (potentially) infinite values on iteration.
- A **Promise** is a computation that may (or may not) eventually return a single value.
- An **Observable** is a lazily evaluated computation that can synchronously or asynchronously return zero to (potentially) infinite values from the time it's invoked onwards.

<span class="informal">For more info about what to use when converting Observables to Promises, please refer to [this guide](/deprecations/to-promise).</span>

## Observables as generalizations of functions

Contrary to popular claims, Observables are not like EventEmitters nor are they like Promises for multiple values. Observables _may act_ like EventEmitters in some cases, namely when they are multicasted using RxJS Subjects, but usually they don't act like EventEmitters.

<span class="informal">Observables are like functions with zero arguments, but generalize those to allow multiple values.</span>

Consider the following:

```ts
function foo() {
  console.log('Hello');
  return 42;
}

const x = foo.call(); // same as foo()
console.log(x);
const y = foo.call(); // same as foo()
console.log(y);
```

We expect to see as output:

```none
"Hello"
42
"Hello"
42
```

You can write the same behavior above, but with Observables:

```ts
import { Observable } from 'rxjs';

const foo = new Observable((subscriber) => {
  console.log('Hello');
  subscriber.next(42);
});

foo.subscribe((x) => {
  console.log(x);
});
foo.subscribe((y) => {
  console.log(y);
});
```

And the output is the same:

```none
"Hello"
42
"Hello"
42
```

This happens because both functions and Observables are lazy computations. If you don't call the function, the `console.log('Hello')` won't happen. Also with Observables, if you don't "call" it (with `subscribe`), the `console.log('Hello')` won't happen. Plus, "calling" or "subscribing" is an isolated operation: two function calls trigger two separate side effects, and two Observable subscribes trigger two separate side effects. As opposed to EventEmitters which share the side effects and have eager execution regardless of the existence of subscribers, Observables have no shared execution and are lazy.

<span class="informal">Subscribing to an Observable is analogous to calling a Function.</span>

Some people claim that Observables are asynchronous. That is not true. If you surround a function call with logs, like this:

```ts
console.log('before');
console.log(foo.call());
console.log('after');
```

You will see the output:

```none
"before"
"Hello"
42
"after"
```

And this is the same behavior with Observables:

```ts
console.log('before');
foo.subscribe((x) => {
  console.log(x);
});
console.log('after');
```

And the output is:

```none
"before"
"Hello"
42
"after"
```

Which proves the subscription of `foo` was entirely synchronous, just like a function.

<span class="informal">Observables are able to deliver values either synchronously or asynchronously.</span>

What is the difference between an Observable and a function? **Observables can "return" multiple values over time**, something which functions cannot. You can't do this:

```ts
function foo() {
  console.log('Hello');
  return 42;
  return 100; // dead code. will never happen
}
```

Functions can only return one value. Observables, however, can do this:

```ts
import { Observable } from 'rxjs';

const foo = new Observable((subscriber) => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100); // "return" another value
  subscriber.next(200); // "return" yet another
});

console.log('before');
foo.subscribe((x) => {
  console.log(x);
});
console.log('after');
```

With synchronous output:

```none
"before"
"Hello"
42
100
200
"after"
```

But you can also "return" values asynchronously:

```ts
import { Observable } from 'rxjs';

const foo = new Observable((subscriber) => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100);
  subscriber.next(200);
  setTimeout(() => {
    subscriber.next(300); // happens asynchronously
  }, 1000);
});

console.log('before');
foo.subscribe((x) => {
  console.log(x);
});
console.log('after');
```

With output:

```none
"before"
"Hello"
42
100
200
"after"
300
```

Conclusion:

- `func.call()` means "_give me one value synchronously_"
- `observable.subscribe()` means "_give me any amount of values, either synchronously or asynchronously_"

## Anatomy of an Observable

Observables are **created** using `new Observable` or a creation operator, are **subscribed** to with an Observer, **execute** to deliver `next` / `error` / `complete` notifications to the Observer, and their execution may be **disposed**. These four aspects are all encoded in an Observable instance, but some of these aspects are related to other types, like Observer and Subscription.

Core Observable concerns:

- **Creating** Observables
- **Subscribing** to Observables
- **Executing** the Observable
- **Disposing** Observables

### Creating Observables

The `Observable` constructor takes one argument: the `subscribe` function.

The following example creates an Observable to emit the string `'hi'` every second to a subscriber.

```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  const id = setInterval(() => {
    subscriber.next('hi');
  }, 1000);
});
```

<span class="informal">Observables can be created with `new Observable`. Most commonly, observables are created using creation functions, like `of`, `from`, `interval`, etc.</span>

In the example above, the `subscribe` function is the most important piece to describe the Observable. Let's look at what subscribing means.

### Subscribing to Observables

The Observable `observable` in the example can be _subscribed_ to, like this:

```ts
observable.subscribe((x) => console.log(x));
```

It is not a coincidence that `observable.subscribe` and `subscribe` in `new Observable(function subscribe(subscriber) {...})` have the same name. In the library, they are different, but for practical purposes you can consider them conceptually equal.

This shows how `subscribe` calls are not shared among multiple Observers of the same Observable. When calling `observable.subscribe` with an Observer, the function `subscribe` in `new Observable(function subscribe(subscriber) {...})` is run for that given subscriber. Each call to `observable.subscribe` triggers its own independent setup for that given subscriber.

<span class="informal">Subscribing to an Observable is like calling a function, providing callbacks where the data will be delivered to.</span>

This is drastically different to event handler APIs like `addEventListener` / `removeEventListener`. With `observable.subscribe`, the given Observer is not registered as a listener in the Observable. The Observable does not even maintain a list of attached Observers.

A `subscribe` call is simply a way to start an "Observable execution" and deliver values or events to an Observer of that execution.

### Executing Observables

The code inside `new Observable(function subscribe(subscriber) {...})` represents an "Observable execution", a lazy computation that only happens for each Observer that subscribes. The execution produces multiple values over time, either synchronously or asynchronously.

There are three types of values an Observable Execution can deliver:

- "Next" notification: sends a value such as a Number, a String, an Object, etc.
- "Error" notification: sends a JavaScript Error or exception.
- "Complete" notification: does not send a value.

"Next" notifications are the most important and most common type: they represent actual data being delivered to a subscriber. "Error" and "Complete" notifications may happen only once during the Observable Execution, and there can only be either one of them.

These constraints are expressed best in the so-called _Observable Grammar_ or _Contract_, written as a regular expression:

```none
next*(error|complete)?
```

<span class="informal">In an Observable Execution, zero to infinite Next notifications may be delivered. If either an Error or Complete notification is delivered, then nothing else can be delivered afterwards.</span>

The following is an example of an Observable execution that delivers three Next notifications, then completes:

```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});
```

Observables strictly adhere to the Observable Contract, so the following code would not deliver the Next notification `4`:

```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
  subscriber.next(4); // Is not delivered because it would violate the contract
});
```

It is a good idea to wrap any code in `subscribe` with `try`/`catch` block that will deliver an Error notification if it catches an exception:

```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  try {
    subscriber.next(1);
    subscriber.next(2);
    subscriber.next(3);
    subscriber.complete();
  } catch (err) {
    subscriber.error(err); // delivers an error if it caught one
  }
});
```

### Disposing Observable Executions

Because Observable Executions may be infinite, and it's common for an Observer to want to abort execution in finite time, we need an API for canceling an execution. Since each execution is exclusive to one Observer only, once the Observer is done receiving values, it has to have a way to stop the execution, in order to avoid wasting computation power or memory resources.

When `observable.subscribe` is called, the Observer gets attached to the newly created Observable execution. This call also returns an object, the `Subscription`:

```ts
const subscription = observable.subscribe((x) => console.log(x));
```

The Subscription represents the ongoing execution, and has a minimal API which allows you to cancel that execution. Read more about the [`Subscription` type here](./guide/subscription). With `subscription.unsubscribe()` you can cancel the ongoing execution:

```ts
import { from } from 'rxjs';

const observable = from([10, 20, 30]);
const subscription = observable.subscribe((x) => console.log(x));
// Later:
subscription.unsubscribe();
```

<span class="informal">When you subscribe, you get back a Subscription, which represents the ongoing execution. Just call `unsubscribe()` to cancel the execution.</span>

Each Observable must define how to dispose resources of that execution when we create the Observable using `create()`. You can do that by returning a custom `unsubscribe` function from within `function subscribe()`.

For instance, this is how we clear an interval execution set with `setInterval`:

```ts
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  // Keep track of the interval resource
  const intervalId = setInterval(() => {
    subscriber.next('hi');
  }, 1000);

  // Provide a way of canceling and disposing the interval resource
  return function unsubscribe() {
    clearInterval(intervalId);
  };
});
```

Just like `observable.subscribe` resembles `new Observable(function subscribe() {...})`, the `unsubscribe` we return from `subscribe` is conceptually equal to `subscription.unsubscribe`. In fact, if we remove the ReactiveX types surrounding these concepts, we're left with rather straightforward JavaScript.

```ts
function subscribe(subscriber) {
  const intervalId = setInterval(() => {
    subscriber.next('hi');
  }, 1000);

  return function unsubscribe() {
    clearInterval(intervalId);
  };
}

const unsubscribe = subscribe({ next: (x) => console.log(x) });

// Later:
unsubscribe(); // dispose the resources
```

The reason why we use Rx types like Observable, Observer, and Subscription is to get safety (such as the Observable Contract) and composability with Operators.

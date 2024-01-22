# 8. 비동기프로그래밍 동시성과 병렬성

### 비동기 프로그래밍

- callback, promise, stream 등의 비동기 API가 있다.
- 자바스크립튼느 스레드 하나로 비동기 처리를 한다
- 어떻게? => **이벤트루프**를 이용해서!  
  ![이벤트루프](https://developer.mozilla.org/ko/docs/Web/JavaScript/Event_loop/the_javascript_runtime_environment_example.svg)

### Event Emitter

- 장점:
- 키의 철자가 틀리거나 인수타입을 잘못사용하거나 인수 전달을 빼먹는 실수를 방지.
- 이벤트와 콜백 매개변수 타입을 제시할 수 있다.

> QUIZ  
> [1] 이벤트 루프란 무엇인가요?

```ts
// [2] 해당 코드의 출력은 어떻게 되는가요? 이유는?
setTimeout(() => console.info('A'), 1);
setTimeout(() => console.info('B'), 2);
console.info('C');

// [3] 아래는?
setTimeout(() => console.info('A'), 0);
Promise.resolve()
  .then(() => console.log(2))
  .then(() => console.log(3));
```

> [3] Promise reject 타입을 extends Error로 만들지 않은 이유는?

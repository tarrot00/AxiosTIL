## 프로미스(promise)

- 프로미스는 '제작 코드'와 '소비 코드'를 연결해주는 특별한 자바스크립트 객체이다.

- 프로미스 객체의 문법

```js
let promise = new Promise(function(resolve, reject) {
  // executor (제작 코드, '가수')
});
```

- new Promise에 전달되는함수는 executor(실행자, 실행함수) 라고 하며, executor의 인수 resolve와 reject는 자바스크립트에서 자체 제공하는 콜백이다. 개발자는 resolve와 reject를 신경 쓰지 않고 executor 안 코드만 작성하면 된다.

- executor는 결과를 즉시 얻든 늦게 얻든 상관없이 상황에 따라 인수로 넘겨준 콜백 중 하나를 반드시 호출해야 한다.
  
  - resolve(value): 일이 성공적으로 끝난 경우 그 결과를 나타내는 value와 함께 호출
  
  - reject(error): 에러 발생 시 에러 객체를 나타내는 error와 함께 호출

- 한편, new Promise 생성자가 반환하는 promise 객체는 다음과 같은 내부 프로퍼티를 갖는다.
  
  - state - 처음엔 "pending"(보류)이었다 resolve 가 호출되면 "fulfilled", reject가 호출되면 "rejected"로 변환한다.
  
  - result - 처음엔 undefined 이었다가 resolv(value)가 호출되면 value로, reject(error)가 호출되면 error로 변한다.

```js
let promise = new Promise(function(resolve, reject) {
  // 프라미스가 만들어지면 executor 함수는 자동으로 실행됩니다.

  // 1초 뒤에 일이 성공적으로 끝났다는 신호가 전달되면서 result는 '완료'가 됩니다.
  setTimeout(() => resolve("완료"), 1000);
});
```

1. executor는 new Promise에 의해 자동으로 그리고 즉각적으로 호출된다.

2. executor는 인자로 resolve 와 reject 함수를 받는다. executor '처리' 가 시작 된 지 1초후, resolve(done)이 호출되고 결과가 만들어진다.

> 이처럼 일이 성공적으로 처리되었을 때의 프로미스는 'fulfilled promise'(약속이 이행된 프로미스) 라고 불리운다.



## 프로미스 요약

## 프라미스는 성공 또는 실패만 합니다.

- executor는 `resolve`나 `reject` 중 하나를 반드시 호출해야 합니다. 이때 변경된 상태는 더 이상 변하지 않습니다.

> 처리가 끝난 프라미스에 `resolve`와 `reject`를 호출하면 무시되죠.

```js
let promise = new Promise(function(resolve, reject) {
  resolve("완료");

  reject(new Error("…")); // 무시됨
  setTimeout(() => resolve("…")); // 무시됨
});
```

> 이렇게 executor에 의해 처리가 끝난 일은 결과 혹은 에러만 가질 수 있습니다. 여기에 더하여 `resolve`나 `reject`는 인수를 하나만 받고(혹은 아무것도 받지 않음) 그 이외의 인수는 무시한다는 특성도 있습니다.



- `Error` 객체와 함께 거부하기
  
  - 무언가 잘못된 경우, executor는 `reject`를 호출해야 합니다. 이때 인수는 `resolve`와 마찬가지로 어떤 타입도 가능하지만 `Error` 객체 또는 `Error`를 상속받은 객체를 사용할 것을 추천합니다. 이유는 뒤에서 설명하겠습니다.

- `resolve`·`reject` 함수 즉시 호출하기

> executor는 대개 무언가를 비동기적으로 수행하고, 약간의 시간이 지난 후에 `resolve`, `reject`를 호출하는데, 꼭 이렇게 할 필요는 없습니다. 아래와 같이 `resolve`나 `reject`를 즉시 호출할 수도 있습니다.

```js
let promise = new Promise(function(resolve, reject) {
  // 일을 끝마치는 데 시간이 들지 않음
  resolve(123); // 결과(123)를 즉시 resolve에 전달함
});
```

> 어떤 일을 시작했는데 알고 보니 일이 이미 끝나 저장까지 되어있는 경우, 이렇게 `resolve`나 `reject`를 즉시 호출하는 방식을 사용할 수 있습니다. 이렇게 하면 프라미스는 즉시 이행 상태가 된다.

- `state`와 `result`는 내부에 있습니다.

- 프라미스 객체의 `state`, `result` 프로퍼티는 내부 프로퍼티이므로 개발자가 직접 접근할 수 없습니다. `.then`/`.catch`/`.finally` 메서드를 사용하면 접근 가능한데, 자세한 내용은 아래에서 살펴보겠습니다.



## 소비자: then, catch, finally

### then

```js
promise.then(
  function(result) { /* 결과(result)를 다룹니다 */ },
  function(error) { /* 에러(error)를 다룹니다 */ }
);
```

- then의 첫 번째 인수는 프라미스가 이행되었을때, 실행되는 함수이고, 여기서 실행 결과를 받는다.

- then의 두 번째 인수는 프라미스가 거부되었을 때 실행되는함수이고, 여기서 에러를 받는다.

```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("완료!"), 1000);
});

// resolve 함수는 .then의 첫 번째 함수(인수)를 실행합니다.
promise.then(
  result => alert(result), // 1초 후 "완료!"를 출력
  error => alert(error) // 실행되지 않음
);
```

- 성공적으로 처리된 경우만 다루고 싶다면 then에 인수를 하나만 전달하면 된다.



### catch

- 에러가 발생한 경우만 다루고 싶다면 `.then(null, errorHandlingFunction)`같이 `null`을 첫 번째 인수로 전달하면 됩니다. `.catch(errorHandlingFunction)`를 써도 되는데, `.catch`는 `.then`에 `null`을 전달하는 것과 동일하게 작동한다.



### finally

- 프라미스가 처리되면(이행이나 거부) `f`가 항상 실행된다는 점에서 `.finally(f)` 호출은 `.then(f, f)`과 유사하다.

```js
new Promise((resolve, reject) => {
  /* 시간이 걸리는 어떤 일을 수행하고, 그 후 resolve, reject를 호출함 */
})
  // 성공·실패 여부와 상관없이 프라미스가 처리되면 실행됨
  .finally(() => 로딩 인디케이터 중지)
  .then(result => result와 err 보여줌 => error 보여줌)
```

- Finally와 .then(f,f)의 차이점

> 1. `finally` 핸들러엔 인수가 없습니다. `finally`에선 프라미스가 이행되었는지, 거부되었는지 알 수 없습니다. `finally`에선 절차를 마무리하는 ‘보편적’ 동작을 수행하기 때문에 성공·실패 여부를 몰라도 됩니다.
> 
> 2. `finally` 핸들러는 자동으로 다음 핸들러에 결과와 에러를 전달합니다.
> 
> 3. `.finally(f)`는 함수 `f`를 중복해서 쓸 필요가 없기 때문에 `.then(f, f)`보다 문법 측면에서 더 편리합니다.



## 예시 LoadScript

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`${src}를 불러오는 도중에 에러가 발생함`));

  document.head.append(script);
}
```

- 프라미스를 이용해 다시 작성한 함수

```js
function loadScript(src) {
  return new Promise(function(resolve, reject) {
    let script = document.createElement('script');
    script.src = src;

    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error(`${src}를 불러오는 도중에 에러가 발생함`));

    document.head.append(script);
  });
}
```

- 사용법

```js
let promise = loadScript("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js");

promise.then(
  script => alert(`${script.src}을 불러왔습니다!`),
  error => alert(`Error: ${error.message}`)
);

promise.then(script => alert('또다른 핸들러...'));
```

- 프로미스 코드와 콜백함수의 비교
  
  - 프라미스를 이용하면 흐름이 자연스럽습니다. `loadScript(script)`로 스크립트를 읽고, 결과에 따라 그다음(`.then`)에 무엇을 할지에 대한 코드를 작성하면 된다.
  
  - 콜백함수의 경우 `loadScript(script, callback)`를 호출할 때, 함께 호출할 `callback` 함수가 준비되어 있어야 합니다. `loadScript`를 호출하기 *이전*에 호출 결과로 무엇을 할지 미리 알고 있어야 합니다.



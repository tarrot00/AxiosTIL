## 프라미스 API

- Promise 클래스의 5가지 정적 메서드

### Promise.all

- 여러 개의 프라미스를 동시에 실행시키고 모든 프라미스가 준비될 때까지 기다린다고 해봅시다. 복수의 URL에 동시에 요청을 보내고, 다운로드가 모두 완료된 후에 콘텐츠를 처리할 때 이런 상황이 발생한다.

- Promise.all은 이럴 때 사용할 수 있습니다.

```js
let promise = Promise.all([...promises...])
```

- Promise.all은 요소 전체가 프라미스인 배열을 받고 새로운 프라미스를 반환한다.

- 배열 안 프라미스가 모두 처리되면 새로운 프라미스 이행되는데, 배열 안 프라미스의 결괏값을 담은 배열이 새로운 프라미스의 resul가 됩니다. 

- 아래 Promise.all은 3초 후에 처리되고, 반환되는 프라미스의 result 는 배열 [1,2,3]이 된다.

```js
Promise.all([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
]).then(alert); // 프라미스 전체가 처리되면 1, 2, 3이 반환됩니다. 각 프라미스는 배열을 구성하는 요소가 됩니다.
```

- result 배열의 요소 순서는 Promise.all에 전달되는 프라미스 순서와 상응한다. 
  
  = Promise.all의 첫 번째 프라미스는 가장 늦게 이행되더라도 처리 결과는 배열의 첫 번째 요소에 저장됩니다. 

- 이를 활용해서 작업해야 할 데이터가 담긴 배열을 프라미스 배열로 매핑하고, 이 배열을 Promise.all 로 감싸는 트릭은 자주 사용됩니다. 

```js
let names = ['iliakan', 'Violet-Bora-Lee', 'jeresig'];

let requests = names.map(name => fetch(`https://api.github.com/users/${name}`));

Promise.all(requests)
  .then(responses => {
    // 모든 응답이 성공적으로 이행되었습니다.
    for(let response of responses) {
      alert(`${response.url}: ${response.status}`); // 모든 url의 응답코드가 200입니다.
    }

    return responses;
  })
  // 응답 메시지가 담긴 배열을 response.json()로 매핑해, 내용을 읽습니다.
  .then(responses => Promise.all(responses.map(r => r.json())))
  // JSON 형태의 응답 메시지는 파싱 되어 배열 'users'에 저장됩니다.
  .then(users => users.forEach(user => alert(user.name)));
```

- Promise.all 에 전달되는 프라미스 중 하나라도 거부되면, Promise.all 이 반환한느 프라미스는 에러와 함께 바로 거부된다.

```js
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("에러 발생!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).catch(alert); // Error: 에러 발생!
```

- 2초 후 두 번째 프라미스가 거부되면서 `Promise.all` 전체가 거부되고, `.catch`가 실행됩니다. 거부 에러는 `Promise.all` 전체의 결과가 됩니다.

> ## 에러가 발생하면 다른 프라미스는 무시됩니다.
> 
> 프라미스가 하나라도 거부되면 `Promise.all`은 즉시 거부되고 배열에 저장된 다른 프라미스의 결과는 완전히 무시됩니다. 이행된 프라미스의 결과도 무시되죠.
> 
> `fetch`를 사용해 호출 여러 개를 만들면, 그중 하나가 실패하더라도 호출은 계속 일어납니다. 그렇더라도 `Promise.all`은 다른 호출을 더는 신경 쓰지 않습니다. 프라미스가 처리되긴 하겠지만 그 결과는 무시됩니다.
> 
> 프라미스에는 '취소’라는 개념이 없어서 `Promise.all`도 프라미스를 취소하지 않습니다. [또 다른 챕터](https://ko.javascript.info/fetch-abort)에서 배울 `AbortController`를 사용하면 프라미스 취소가 가능하긴 하지만, `AbortController`는 프라미스 API는 아닙니다.



## Promise.allSettled

- Promise.all 은 프라미스가 하나라도 거절되면 전체를 거절하지만, Promise.allSettled는 모든 프라미스가 처리될 때까지 기다린다. 그리고 반환되는 배열은 
  
  - 응답이 성공한 경우 - `{status:"fulfilled", value:result}`
  
  - 에러가 발생한 경우 - `{status:"rejected", reason:error}`

- `fetch`를 사용해 여러 사람의 정보를 가져오고 있다고 해봅시다. 여러 요청 중 하나가 실패해도 다른 요청 결과는 여전히 필요하다. 이러한 경우 Promise.allSettled를 사용할 수 있다. 



## Promise.race

- `Promise.race`는 `Promise.all`과 비슷하다. 다만 가장 먼저 처리되는 프라미스의 결과(혹은 에러)를 반환한다. 

```js
let promise = Promise.race(iterable);
```

```js
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("에러 발생!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); //  결과 1
```



## Promise.resolve 와 Promise.reject

- 최근 많이 사용하지 않는다. 

# Testing Asynchronous Code

JavaScript에서 코드가 비동기적으로 실행되는 것은 일반적입니다. 코드가 비동기적으로 실행될 때 Jest는 테스트 중인 코드가 완료되기를 기다려야 다른 테스트로 넘어갈 수 있습니다. Jest에는 이를 처리하기 위한 여러 가지 방법이 있습니다.

## Promises
테스트에서 `Promise`를 반환하면 Jest는 해당 `Promise`가 해결될 때까지 기다립니다. `Promise`가 거부되면 테스트가 실패합니다.

예를 들어, `fetchData`가 `'peanut butter'`라는 문자열로 해결되는 `Promise`를 반환하는 것으로 가정해봅시다. 이를 다음과 같이 테스트할 수 있습니다:

```ts
test('데이터는 peanut butter입니다', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```

## Async/Await
또는 테스트에 `async`와 `await`을 사용할 수 있습니다. `async` 키워드를 사용하여 비동기 테스트를 작성합니다. 예를 들어, 동일한 `fetchData` 시나리오를 다음과 같이 테스트할 수 있습니다:

```ts
test('데이터는 peanut butter입니다', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('fetch가 에러로 실패합니다', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch('error');
  }
});
```

`async`와 `await`를 `.resolves` 또는 `.rejects`와 함께 사용할 수도 있습니다.

```ts
test('데이터는 peanut butter입니다', async () => {
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('fetch가 에러로 실패합니다', async () => {
  await expect(fetchData()).rejects.toMatch('error');
});
```

이러한 경우, `async`와 `await`는 사실상 `Promise` 예제에서 사용하는 로직과 동일한 동작을 하는 구문적 설탕입니다.

주의: 반드시 `Promise`를 반환하거나 `await` 문을 사용하세요. 반환문이나 `await` 문을 생략하면 `fetchData`에서 반환된 `Promise`가 해결되기 전에 테스트가 완료되어 버립니다.

`Promise`가 거부되는 것을 기대한다면 `.catch` 메서드를 사용하세요. 특정한 수의 어서션(assertion)이 호출되는지 확인하기 위해 `expect.assertions`를 추가해야 합니다. 그렇지 않으면 이행된 `Promise`는 테스트를 실패시키지 않습니다.

```ts
test('fetch가 에러로 실패합니다', () => {
  expect.assertions(1);
  return fetchData().catch(e => expect(e).toMatch('error'));
});
```

## 콜백(callbacks)
`Promise`를 사용하지 않는 경우, 콜백(callback)을 사용할 수 있습니다. 예를 들어, `fetchData`가 `Promise` 대신 콜백을 기대하는 형태라고 가정해봅시다. 즉, 어떤 데이터를 가져오고 완료되면 `callback(null, data)`를 호출합니다. 이 반환된 데이터가 `'peanut butter'`라는 문자열인지를 테스트하고 싶습니다.

기본적으로 Jest 테스트는 실행이 끝나면 완료됩니다. 이 테스트는 의도한대로 작동하지 않습니다:

```ts
// 이렇게 하지 마세요!
test('데이터는 peanut butter입니다', () => {
  function callback(error, data) {
    if (error) {
      throw error;
    }
    expect(data).toBe('peanut butter');
  }

  fetchData(callback);
});
```

문제는 `fetchData`가 완료될 때마다 콜백을 호출하기 전에 테스트가 완료된다는 것입니다.

이를 해결하는 대체형 test가 있습니다. 비어있는 인수로 함수를 함수 내에 넣는 대신 `done`이라는 단일 인수를 사용합니다. Jest는 `done` 콜백이 호출될 때까지 기다립니다.

```ts
test('데이터는 peanut butter입니다', done => {
  function callback(error, data) {
    if (error) {
      done(error);
      return;
    }
    try {
      expect(data).toBe('peanut butter');
      done();
    } catch (error) {
      done(error);
    }
  }

  fetchData(callback);
});
```

`done()`이 호출되지 않으면 테스트는 실패합니다(시간 초과 오류). 이렇게 동작하길 원하는 바입니다.

`expect` 문이 실패하면 오류를 `throw`하고 `done()`이 호출되지 않습니다. 테스트 로그에서 실패한 이유를 보려면 `expect`를 `try` 블록으로 래핑하고 `catch` 블록에서 오류를 `done`에 전달해야 합니다. 그렇지 않으면 어떤 값이 `expect(data)`에 의해 수신되었는지 표시되지 않는 불투명한 시간 초과 오류가 발생합니다.

주의: Jest는 동일한 테스트 함수에 `done()` 콜백이 전달되고 프로미스가 반환된 경우 오류를 throw합니다. 이는 테스트에서 메모리 누수를 방지하기 위한 예방 조치입니다.

## .resolves / .rejects

`.resolves` 매처를 `expect` 문에 사용할 수도 있으며, Jest는 해당 `Promise`가 해결될 때까지 기다립니다. `Promise`가 거부되면 테스트가 자동으로 실패합니다.

```ts
test('데이터는 peanut butter입니다', () => {
  return expect(fetchData()).resolves.toBe('peanut butter');
});
```
확인문을 반환하는 것을 잊지 마세요. 이 `return` 문을 생략하면 `fetchData`에서 반환된 `Promise`가 해결되고 `then()`이 콜백을 실행할 기회가 생기기 전에 테스트가 완료됩니다.

`Promise`가 거부될 것으로 예상하는 경우 `.rejects` 매처를 사용하세요. 이는 `.resolves` 매처와 유사하게 동작합니다. `Promise`가 이행된 경우 테스트가 자동으로 실패합니다.

```ts
test('fetch가 에러로 실패합니다', () => {
  return expect(fetchData()).rejects.toMatch('error');
});
```

이러한 형태 중 어느 것이 다른 형태보다 특별히 우수한 것은 아니며, 코드베이스 전체 또는 하나의 파일에서 혼합하여 사용할 수 있습니다. 단지 테스트를 더 간단하게 만드는 데 어떤 스타일을 선호하는지에 달려있습니다.
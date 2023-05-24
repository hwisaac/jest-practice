# An Async Example

먼저, Jest에서 Babel 지원을 활성화해야 합니다. Getting Started 가이드에서 설명된대로 작업하세요.

사용자 데이터를 API에서 가져와 사용자 이름을 반환하는 모듈을 구현해 보겠습니다.

```ts
// user.js
import request from './request';

export function getUserName(userID) {
  return request(`/users/${userID}`).then(user => user.name);
}
```
위의 구현에서 `request.js` 모듈이 프로미스를 반환하도록 예상합니다. 우리는 사용자 이름을 받기 위해 `then`으로 체인을 이어갑니다.

이제 네트워크에 연결하여 일부 사용자 데이터를 가져오는 `request.js`의 구현을 상상해보십시오.

```ts
// request.js
const http = require('http');

export default function request(url) {
  return new Promise(resolve => {
    // 이것은 예시로 API에서 사용자 데이터를 가져오기 위한 http 요청입니다.
    // 이 모듈은 __mocks__/request.js에서 모의(mock)되고 있습니다.
    http.get({path: url}, response => {
      let data = '';
      response.on('data', _data => (data += _data));
      response.on('end', () => resolve(data));
    });
  });
}
```
테스트에서 네트워크에 연결하지 않기 위해 `request.js` 모듈에 대한 수동 모의(mock)를 `__mocks__` 폴더에 만들 것입니다 (`__MOCKS__` 폴더는 작동하지 않습니다). 다음과 같이 보일 수 있습니다.

```ts
// __mocks__/request.js
const users = {
  4: {name: 'Mark'},
  5: {name: 'Paul'},
};

export default function request(url) {
  return new Promise((resolve, reject) => {
    const userID = parseInt(url.substr('/users/'.length), 10);
    process.nextTick(() =>
      users[userID]
        ? resolve(users[userID])
        : reject({
            error: `User with ${userID} not found.`,
          }),
    );
  });
}
```
이제 비동기 기능에 대한 테스트를 작성해 보겠습니다.

```ts
// __tests__/user-test.js
jest.mock('../request');

import * as user from '../user';

// Promise를 반환하는 어설션은 반환되어야 합니다.
it('works with promises', () => {
  expect.assertions(1);
  return user.getUserName(4).then(data => expect(data).toBe('Mark'));
});
```

`jest.mock('../request')`를 호출하여 Jest에 수동 모의(mock)를 사용하도록 알립니다. `it`은 반환 값이 해결될 Promise로 예상됩니다. 원하는 만큼 많은 Promise를 연결하고 언제든지 `expect`를 호출할 수 있지만, 마지막에 Promise를 반환해야 합니다.

## .resolves
`.resolves`를 사용하여 이행된 Promise의 값을 다른 matcher와 함께 추출하는 더 간결한 방법도 있습니다. Promise가 거부되면 어설션이 실패합니다.

ts
Copy code
it('works with resolves', () => {
  expect.assertions(1);
  return expect(user.getUserName(5)).resolves.toBe('Paul');
});

## `async`/`await`
`async`/`await` 구문을 사용하여 테스트를 작성하는 것도 가능합니다. 이전 예제를 `async`/`await` 구문을 사용하여 작성하는 방법은 다음과 같습니다.

```ts
// async/await를 사용할 수 있습니다.
it('works with async/await', async () => {
  expect.assertions(1);
  const data = await user.getUserName(4);
  expect(data).toBe('Mark');
});

// async/await를 .resolves와 함께 사용할 수도 있습니다.
it('works with async/await and resolves', async () => {
  expect.assertions(1);
  await expect(user.getUserName(5)).resolves.toBe('Paul');
});
```
프로젝트에서 `async`/`await를` 사용하려면 `@babel/preset-env`를 설치하고 `babel.config.js` 파일에서 해당 기능을 활성화해야 합니다.

## 에러 처리
`.catch` 메서드를 사용하여 에러를 처리할 수 있습니다. 특정한 수의 어설션(assertion)이 호출되는지 확인하기 위해 `expect.assertions`를 추가해야 합니다. 그렇지 않으면 이행된 Promise는 테스트를 실패시키지 않을 것입니다.

```ts
// Promise.catch를 사용하여 비동기 에러 테스트.
it('tests error with promises', () => {
  expect.assertions(1);
  return user.getUserName(2).catch(e =>
    expect(e).toEqual({
      error: 'User with 2 not found.',
    }),
  );
});

// 또는 async/await을 사용합니다.
it('tests error with async/await', async () => {
  expect.assertions(1);
  try {
    await user.getUserName(1);
  } catch (e) {
    expect(e).toEqual({
      error: 'User with 1 not found.',
    });
  }
});
```

## .rejects
`.rejects` 헬퍼는 `.resolves` 헬퍼와 동일하게 작동합니다. Promise가 이행된 경우 테스트는 자동으로 실패합니다. `expect.assertions(number)`은 필요하지 않지만 테스트 중에 특정한 수의 어설션(assertion)이 호출되는지 확인하는 것이 권장됩니다. `.resolves` 어설션들을 `return`/`await`하는 것을 잊는 실수를 하기 쉽기 때문입니다.

```ts
// `.rejects`를 사용하여 비동기 에러 테스트.
it('tests error with rejects', () => {
  expect.assertions(1);
  return expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});

// `.rejects`를 사용하여 async/await과 함께 사용합니다.
it('tests error with async/await and rejects', async () => {
  expect.assertions(1);
  await expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});
```
이 예제의 코드는 [examples/async](https://github.com/jestjs/jest/tree/main/examples/async)에서 확인할 수 있습니다.

`setTimeout`과 같은 타이머를 테스트하려면 Timer mocks 문서를 참조하세요.
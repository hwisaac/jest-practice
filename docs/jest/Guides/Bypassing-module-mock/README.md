# Bypassing module mock

Jest는 테스트에서 전체 모듈을 목(mock)으로 대체할 수 있도록 해줍니다. 이는 코드가 해당 모듈의 함수를 올바르게 호출하는지 테스트하는 데 유용합니다. 그러나 때로는 테스트 파일에서 모듈의 일부를 사용하고 싶은 경우가 있습니다. 이 경우 목(mock)된 버전이 아닌 원래의 구현에 접근해야 합니다.

다음 `createUser` 함수에 대한 테스트 케이스를 작성해보겠습니다.

```ts
// createUser.js
import fetch from 'node-fetch';

export const createUser = async () => {
  const response = await fetch('https://website.com/users', {method: 'POST'});
  const userId = await response.text();
  return userId;
};
```
테스트에서 실제 네트워크 요청을 수행하지 않고 `fetch` 함수를 목(mock)으로 대체해야 합니다. 그러나 `fetch`의 반환값을 `Response`(`Promise`로 래핑된)로 목(mock)해야 합니다. 함수 내에서 이 값을 사용하여 생성된 사용자의 ID를 가져오기 때문입니다. 다음과 같이 테스트를 작성해보려고 할 수 있습니다.

```ts
jest.mock('node-fetch');

import fetch, {Response} from 'node-fetch';
import {createUser} from './createUser';

test('createUser calls fetch with the right args and returns the user id', async () => {
  fetch.mockReturnValue(Promise.resolve(new Response('4')));

  const userId = await createUser();

  expect(fetch).toHaveBeenCalledTimes(1);
  expect(fetch).toHaveBeenCalledWith('https://website.com/users', {
    method: 'POST',
  });
  expect(userId).toBe('4');
});
```
그러나 위의 테스트를 실행하면 `createUser` 함수가 실패하고 `TypeError: response.text is not a function` 오류가 발생합니다. 이는 테스트 파일의 상단에 있는 `jest.mock` 호출로 인해 `node-fetch`에서 가져온 `Response` 클래스가 목(mock)으로 대체되었기 때문입니다. 따라서 원래의 동작을 수행하지 않게 되는 것입니다.

이와 같은 문제를 해결하기 위해 Jest는 `jest.requireActual` 도우미를 제공합니다. 위의 테스트가 올바르게 동작하도록 하려면 테스트 파일에서 다음과 같이 `import` 문을 변경하세요.

```ts
// 이전 코드
jest.mock('node-fetch');
import fetch, {Response} from 'node-fetch';
```
```ts
// 변경된 코드
jest.mock('node-fetch');
import fetch from 'node-fetch';
const {Response} = jest.requireActual('node-fetch');
```

이렇게 하면 테스트 파일이 `node-fetch`에서 실제 `Response` 객체를 가져오게 되어 목(mock) 버전이 아닌 원래의 구현을 사용하게 됩니다. 이렇게 변경하면 테스트가 올바르게 통과될 것입니다.
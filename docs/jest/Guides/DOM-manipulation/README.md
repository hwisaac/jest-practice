# DOM manipulation

DOM을 직접 조작하는 코드는 종종 테스트하기 어렵다고 여겨지는 함수들 중 하나입니다. 

다음과 같은 `jQuery` 코드를 테스트하는 방법을 살펴보겠습니다. 이 코드는 클릭 이벤트를 수신 대기하고 비동기적으로 일부 데이터를 가져와서 span의 내용을 설정합니다.

```javascript
// displayUser.js
'use strict';

const $ = require('jquery');
const fetchCurrentUser = require('./fetchCurrentUser.js');

$('#button').click(() => {
  fetchCurrentUser(user => {
    const loggedText = 'Logged ' + (user.loggedIn ? 'In' : 'Out');
    $('#username').text(user.fullName + ' - ' + loggedText);
  });
});
```
다시 한 번, `__tests__/` 폴더에 테스트 파일을 만듭니다.

```javascript
// __tests__/displayUser-test.js
'use strict';

jest.mock('../fetchCurrentUser');

test('displays a user after a click', () => {
  // 문서 본문을 설정합니다.
  document.body.innerHTML =
    '<div>' +
    '  <span id="username" />' +
    '  <button id="button" />' +
    '</div>';

  // 이 모듈에는 부작용이 있습니다.
  require('../displayUser');

  const $ = require('jquery');
  const fetchCurrentUser = require('../fetchCurrentUser');

  // fetchCurrentUser 목 함수가 콜백과 함께 자동으로 호출되도록 설정합니다.
  fetchCurrentUser.mockImplementation(cb => {
    cb({
      fullName: 'Johnny Cash',
      loggedIn: true,
    });
  });

  // jQuery를 사용하여 버튼을 클릭한 것처럼 동작합니다.
  $('#button').click();

  // fetchCurrentUser 함수가 호출되었고, #username span의 내용이 예상한 대로 업데이트되었는지 확인합니다.
  expect(fetchCurrentUser).toBeCalled();
  expect($('#username').text()).toBe('Johnny Cash - Logged In');
});
```

`fetchCurrentUser.js`를 목(mock)화하여 실제 네트워크 요청이 발생하지 않고 테스트가 로컬에서 모의 데이터로 해결되도록 합니다. 이렇게 하면 테스트가 몇 밀리초 안에 완료되고 빠른 단위 테스트 반복 속도를 보장할 수 있습니다.

또한, 테스트하는 함수는 `#button` DOM 요소에 이벤트 리스너를 추가하므로 테스트를 위해 DOM을 올바르게 설정해야 합니다. `jsdom`과 `jest-environment-jsdom` 패키지는 브라우저에서와 같은 방식으로 DOM 환경을 시뮬레이션합니다. 이는 우리가 호출하는 모든 DOM API를 브라우저에서와 동일한 방식으로 관찰할 수 있다는 것을 의미합니다.

`JSDOM` 테스트 환경을 사용하기 위해서는 `jest-environment-jsdom` 패키지를 설치해야 합니다. 이미 설치되어 있지 않은 경우 다음과 같이 설치합니다.

```bash
npm install --save-dev jest-environment-jsdom
```
이 예제의 코드는 [examples/jquery](https://github.com/jestjs/jest/tree/main/examples/jquery)에서 사용할 수 있습니다.
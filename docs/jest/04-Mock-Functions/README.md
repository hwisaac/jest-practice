# 모의 함수(Mock Functions)
모의 함수(Mock Functions)는 코드 간의 연결을 테스트하기 위해 실제 함수의 구현을 지워버리고, 함수에 대한 호출 (및 해당 호출에 전달된 매개변수), `new`로 인스턴스화된 생성자 함수의 인스턴스를 캡처하고, 반환 값의 테스트 시간 구성을 허용하는 기능을 제공합니다.

함수를 모의하기 위해 두 가지 방법이 있습니다. 테스트 코드에서 사용할 모의 함수를 생성하거나, 모듈 종속성을 재정의하기 위해 `manual mock`를 작성하는 것입니다.

## 모의 함수 사용하기
`forEach`라는 함수를 구현한 것을 테스트한다고 상상해 봅시다. 이 함수는 제공된 배열의 각 항목에 대해 콜백을 호출합니다.

```ts
// forEach.js
export function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
```

이 함수를 테스트하기 위해 모의 함수를 사용하고, 모의 함수의 상태를 검사하여 콜백이 예상대로 호출되는지 확인할 수 있습니다.

```ts
// forEach.test.js
const forEach = require('./forEach');

const mockCallback = jest.fn(x => 42 + x);

test('forEach mock function', () => {
  forEach([0, 1], mockCallback);

  // 모의 함수는 두 번 호출되었습니다.
  expect(mockCallback.mock.calls).toHaveLength(2);

  // 함수의 첫 번째 호출의 첫 번째 인자는 0입니다.
  expect(mockCallback.mock.calls[0][0]).toBe(0);

  // 함수의 두 번째 호출의 첫 번째 인자는 1입니다.
  expect(mockCallback.mock.calls[1][0]).toBe(1);

  // 첫 번째 호출의 반환 값은 42입니다.
  expect(mockCallback.mock.results[0].value).toBe(42);
});
```

## `.mock` property

모든 모의 함수는 특별한 `.mock` 속성을 갖고 있으며, 함수가 호출되고 반환된 내용에 대한 데이터가 여기에 저장됩니다. `.mock` 속성은 각 호출에 대한 `this` 값도 추적하므로 이를 검사하는 것도 가능합니다.

```ts
const myMock1 = jest.fn();
const a = new myMock1();
console.log(myMock1.mock.instances);
// > [ <a> ]

const myMock2 = jest.fn();
const b = {};
const bound = myMock2.bind(b);
bound();
console.log(myMock2.mock.contexts);
// > [ <b> ]
```

이러한 모의 속성들은 테스트에서 이 함수들이 어떻게 호출되는지, 인스턴스화되는지 또는 반환하는지를 단언하는 데 매우 유용합니다.

```ts
// 함수는 정확히 한 번 호출되었습니다.
expect(someMockFunction.mock.calls).toHaveLength(1);

// 함수의 첫 번째 호출의 첫 번째 인자는 'first arg'입니다.
expect(someMockFunction.mock.calls[0][0]).toBe('first arg');

// 함수의 첫 번째 호출의 두 번째 인자는 'second arg'입니다.
expect(someMockFunction.mock.calls[0][1]).toBe('second arg');

// 함수의 첫 번째 호출의 반환 값은 'return value'입니다.
expect(someMockFunction.mock.results[0].value).toBe('return value');

// 함수가 특정한 `this` 컨텍스트(element 객체)로 호출되었습니다.
expect(someMockFunction.mock.contexts[0]).toBe(element);

// 이 함수가 정확히 두 번 인스턴스화되었습니다.
expect(someMockFunction.mock.instances.length).toBe(2);

// 이 함수의 첫 번째 인스턴스화에서 반환된 객체에 `name` 속성이 'test'로 설정되었습니다.
expect(someMockFunction.mock.instances[0].name).toBe('test');

// 마지막 호출의 첫 번째 인자는 'test'입니다.
expect(someMockFunction.mock.lastCall[0]).toBe('test');
```

## 모의 반환 값

모의 함수를 사용하여 코드에 테스트 값을 주입할 수도 있습니다.

```ts
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock.mockReturnValueOnce(10).mockReturnValueOnce('x').mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```

모의 함수는 함수형 continuation-passing 스타일을 사용하는 코드에서도 매우 유용합니다. 이 스타일로 작성된 코드는 테스트 직전에 값을 직접 주입하는 대신 실제 구성 요소를 복제하는 복잡한 스텁의 필요성을 피할 수 있습니다.

```ts
const filterTestFn = jest.fn();

// 모의 함수의 첫 번째 호출은 `true`를 반환하고,
// 두 번째 호출은 `false`를 반환합니다.
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter(num => filterTestFn(num));

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls[0][0]); // 11
console.log(filterTestFn.mock.calls[1][0]); // 12
```

실제 예제 대부분은 종속 컴포넌트에 대한 모의 함수를 얻고 구성하는 것을 포함하지만, 기술은 동일합니다. 이러한 경우 직접 테스트되지 않은 함수 내부에서 로직을 구현하는 유혹을 피하도록 노력해야 합니다.

## 모듈 모의

API에서 사용자를 가져오는 클래스를 가지고 있다고 가정해 봅시다. 이 클래스는 API를 호출하기 위해 `axios`를 사용하고 `data` 속성을 반환합니다.

```ts

// users.js
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}

export default Users;
```

이제 API를 실제로 호출하지 않고(따라서 느리고 취약한 테스트를 만들지 않음), axios 모듈을 자동으로 모의하기 위해 `jest.mock(...)` 함수를 사용할 수 있습니다.

모듈을 모의(mock)하면 `.get`에 대해 가짜 응답을 반환하는 `mockResolvedValue`를 제공할 수 있습니다. 이렇게 하면 `axios.get('/users.json')`가 가짜 응답을 반환하도록 지정합니다.

```ts
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const users = [{name: 'Bob'}];
  const resp = {data: users};
  axios.get.mockResolvedValue(resp);

  // 또는 다음을 사용할 수도 있습니다.
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(data => expect(data).toEqual(users));
});
```

## 부분적인 모의
모듈의 부분 집합을 모의하고 다른 부분은 실제 구현을 유지할 수 있습니다.

```ts
// foo-bar-baz.js
export const foo = 'foo';
export const bar = () => 'bar';
export default () => 'baz';
```

```ts
//test.js
import defaultExport, {bar, foo} from '../foo-bar-baz';

jest.mock('../foo-bar-baz', () => {
  const originalModule = jest.requireActual('../foo-bar-baz');

  // default export와 named export 'foo'를 모의(mock)합니다.
  return {
    __esModule: true,
    ...originalModule,
    default: jest.fn(() => 'mocked baz'),
    foo: 'mocked foo',
  };
});

test('should do a partial mock', () => {
  const defaultExportResult = defaultExport();
  expect(defaultExportResult).toBe('mocked baz');
  expect(defaultExport).toHaveBeenCalled();

  expect(foo).toBe('mocked foo');
  expect(bar()).toBe('bar');
});
```

## 모의 구현
여전히 반환 값만 지정하고 모의 함수의 구현을 완전히 대체하는 것이 유용한 경우도 있습니다. 이는 `jest.fn` 또는 mock 함수의 `mockImplementationOnce` 메서드를 사용하여 수행할 수 있습니다.

```ts
const myMockFn = jest.fn(cb => cb(null, true));

myMockFn((err, val) => console.log(val));
// > true
```
mockImplementation 메서드는 다른 모듈에서 생성된 모의 함수의 기본 구현을 정의해야 할 때 유용합니다.

```ts
// foo.js
module.exports = function () {
  // 일부 구현;
};
```

```ts
// test.js
jest.mock('../foo'); // 이는 자동으로 발생하는 일입니다.
const foo = require('../foo');

// foo는 모의 함수입니다.
foo.mockImplementation(() => 42);
foo();
// > 42
```

`mockImplementationOnce` 메서드를 사용하면 여러 함수 호출이 서로 다른 결과를 생성하는 복잡한 모의 함수의 동작을 재현할 수 있습니다.

```ts
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > false
```

`mockImplementationOnce`에 정의된 구현이 모두 사용되면 `jest.fn`의 기본 구현이 실행됩니다(정의된 경우):

```ts
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

일반적으로 체인으로 연결되는 메서드를 가진 메서드의 경우, `this`를 항상 반환해야 하는 경우가 있습니다. 이럴 때 모든 모의 함수에 `.mockReturnThis()` 함수가 있는 단축 API가 있습니다.

```ts
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// 다음과 같습니다.

const otherObj = {
  myMethod: jest.fn(function () {
    return this;
  }),
};
```

## 모의 함수 이름
테스트 오류 출력에서 `jest.fn()` 대신 테스트하는 동안 오류를 보고하는 모의 함수에 대한 이름을 제공할 수 있습니다. 테스트 출력에서 오류를 보고하는 모의 함수를 신속하게 식별하려면 `.mockName()`을 사용하십시오.

```ts
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockImplementation(scalar => 42 + scalar)
  .mockName('add42');
```

## 사용자 정의 매처
마지막으로, 모의 함수가 어떻게 호출되었는지 검증하기 위해 일반적인 형식의 매처 함수를 제공합니다.

```ts
// 모의 함수는 최소한 한 번 호출되었습니다.
expect(mockFunc).toHaveBeenCalled();

// 모의 함수는 지정된 인수와 최소한 한 번 호출되었습니다.
expect(mockFunc).toHaveBeenCalledWith(arg1, arg2);

// 모의 함수의 마지막 호출은 지정된 인수와 일치합니다.
expect(mockFunc).toHaveBeenLastCalledWith(arg1, arg2);

// 모든 호출 및 모의 이름이 스냅샷으로 작성됩니다.
expect(mockFunc).toMatchSnapshot();
```

이러한 매처들은 `.mock` 속성을 검사하는 일반적인 형식을 위한 간편한 함수들입니다. 맛에 맞게 직접 수동으로 수행하거나 더 구체적인 작업을 수행해야 하는 경우 언제든지 수동으로 수행할 수 있습니다.

```ts
// 모의 함수는 최소한 한 번 호출되었습니다.
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// 모의 함수는 지정된 인수와 최소한 한 번 호출되었습니다.
expect(mockFunc.mock.calls).toContainEqual([arg1, arg2]);

// 모의 함수의 마지막 호출은 지정된 인수와 일치합니다.
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual([
  arg1,
  arg2,
]);

// 마지막 호출의 첫 번째 인수는 `42`입니다.
// (이러한 구체적인 단언에 대한 sugar 도우미가 없음에 유의하세요.)
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);

// 스냅샷은 모의 함수가 동일한 횟수로 호출되었으며, 동일한 순서로, 동일한 인수를 사용하는지 확인합니다. 또한 이름에 대해 단언합니다.
expect(mockFunc.mock.calls).toEqual([[arg1, arg2]]);
expect(mockFunc.getMockName()).toBe('a mock name');
```

매처의 전체 목록은 [참조 문서](https://jestjs.io/docs/expect)를 확인하십시오.
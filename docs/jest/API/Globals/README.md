# Globals
Jest는 각각의 메서드와 객체를 전역 환경(global environment)에 넣습니다. 이들을 사용하려면 `require` 또는 `import`를 하지 않아도 됩니다. 그러나 명시적인 import를 선호하는 경우 `import {describe, expect, test} from '@jest/globals'`와 같이 사용할 수 있습니다.

## 메서드
- Reference
  - afterAll(fn, timeout)
  - afterEach(fn, timeout)
  - beforeAll(fn, timeout)
  - beforeEach(fn, timeout)
  - describe(name, fn)
  - describe.each(table)(name, fn, timeout)
  - describe.only(name, fn)
  - describe.only.each(table)(name, fn)
  - describe.skip(name, fn)
  - describe.skip.each(table)(name, fn)
  - test(name, fn, timeout)
  - test.concurrent(name, fn, timeout)
  - test.concurrent.each(table)(name, fn, timeout)
  - test.concurrent.only.each(table)(name, fn)
  - test.concurrent.skip.each(table)(name, fn)
  - test.each(table)(name, fn, timeout)
  - test.failing(name, fn, timeout)
  - test.failing.each(name, fn, timeout)
  - test.only.failing(name, fn, timeout)
  - test.skip.failing(name, fn, timeout)
  - test.only(name, fn, timeout)
  - test.only.each(table)(name, fn)
  - test.skip(name, fn)
  - test.skip.each(table)(name, fn)
  - test.todo(name)
- TypeScript Usage
  - .each

## Reference
### afterAll(fn, timeout)
이 함수는 해당 파일의 모든 테스트가 완료된 후에 실행됩니다. 함수가 프로미스를 반환하거나 제너레이터인 경우, Jest는 해당 프로미스가 해결될 때까지 대기합니다.

선택적으로 `timeout`(밀리초)을 제공하여 중단하기 전까지 대기할 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

이 메서드는 테스트간에 공유되는 전역 설정 상태를 정리하는 데 유용합니다.

예를 들어:
```ts
const globalDatabase = makeGlobalDatabase();

function cleanUpDatabase(db) {
  db.cleanUp();
}

afterAll(() => {
  cleanUpDatabase(globalDatabase);
});

test('can find things', () => {
  return globalDatabase.find('thing', {}, results => {
    expect(results.length).toBeGreaterThan(0);
  });
});

test('can insert a thing', () => {
  return globalDatabase.insert('thing', makeThing(), response => {
    expect(response.success).toBeTruthy();
  });
});
```
위의 예제에서 `afterAll`은 모든 테스트가 실행된 후에 `cleanUpDatabase`를 호출하도록 보장합니다.

만약 `afterAll`이 `describe` 블록 내에 있다면, 해당 describe 블록이 끝날 때 실행됩니다.

모든 테스트가 아닌 각 테스트가 완료된 후에 일부 정리 코드를 실행하려면 `afterEach`를 사용하세요.

### afterEach(fn, timeout)
이 함수는 해당 파일의 각 테스트가 완료된 후에 실행됩니다. 함수가 프로미스를 반환하거나 제너레이터인 경우, Jest는 해당 프로미스가 해결될 때까지 대기합니다.

선택적으로 `timeout`(밀리초)을 제공하여 중단하기 전까지 대기할 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

이 메서드는 각 테스트마다 생성된 일시적인 상태를 정리하는 데 유용합니다.

예를 들어:

```ts
const globalDatabase = makeGlobalDatabase();

function cleanUpDatabase(db) {
  db.cleanUp();
}

afterEach(() => {
  cleanUpDatabase(globalDatabase);
});

test('can find things', () => {
  return globalDatabase.find('thing', {}, results => {
    expect(results.length).toBeGreaterThan(0);
  });
});

test('can insert a thing', () => {
  return globalDatabase.insert('thing', makeThing(), response => {
    expect(response.success).toBeTruthy();
  });
});
```
위의 예제에서 `afterEach`는 각 테스트가 실행된 후에 `cleanUpDatabase`를 호출하도록 보장합니다.

만약 `afterEach`가 `describe` 블록 내에 있다면, 해당 `describe` 블록에 속한 테스트가 완료된 후에만 실행됩니다.

한 번만 실행되는 대신에 각 테스트마다 실행되는 정리 코드가 필요한 경우에는 `afterAll`을 사용하세요.

### beforeAll(fn, timeout)
이 함수는 해당 파일의 어떤 테스트도 실행되기 전에 실행됩니다. 함수가 프로미스를 반환하거나 제너레이터인 경우, Jest는 해당 프로미스가 해결될 때까지 테스트를 실행하기 전에 대기합니다.

선택적으로 `timeout`(밀리초)을 제공하여 중단하기 전까지 대기할 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

이 메서드는 많은 테스트에서 사용될 전역 상태를 설정하는 데 유용합니다.

예를 들어:

```ts
const globalDatabase = makeGlobalDatabase();

beforeAll(() => {
  // 데이터베이스를 초기화하고 테스트 데이터를 추가합니다.
  // Jest는 이 프로미스가 해결될 때까지 대기합니다.
  return globalDatabase.clear().then(() => {
    return globalDatabase.insert({testData: 'foo'});
  });
});

// 이 예제에서는 데이터베이스를 한 번만 설정하기 때문에 beforeAll이 필요합니다.
test('can find things', () => {
  return globalDatabase.find('thing', {}, results => {
    expect(results.length).toBeGreaterThan(0);
  });
});
```

위의 예제에서 `beforeAll`은 테스트가 실행되기 전에 데이터베이스를 설정하도록 보장합니다. 동기적인 설정이라면 `beforeAll` 없이 이 작업을 수행할 수 있습니다. 중요한 점은 Jest가 프로미스가 해결될 때까지 대기하므로 비동기적인 설정도 가능하다는 점입니다.

만약 `beforeAll`이 `describe` 블록 내에 있다면, 해당 `describe` 블록이 시작되기 전에 실행됩니다.

모든 테스트가 아닌 모든 테스트마다 실행되는 설정 코드가 필요한 경우에는 `beforeEach`를 사용하세요.

### beforeEach(fn, timeout)
이 함수는 해당 파일의 각 테스트가 실행되기 전에 실행됩니다. 함수가 프로미스를 반환하거나 제너레이터인 경우, Jest는 해당 프로미스가 해결될 때까지 테스트를 실행하기 전에 대기합니다.

선택적으로 타임아웃(밀리초)을 제공하여 중단하기 전까지 대기할 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

이 메서드는 많은 테스트에서 사용되는 전역 상태를 초기화하는 데 유용합니다.

예를 들어:

```ts
const globalDatabase = makeGlobalDatabase();

beforeEach(() => {
  // 데이터베이스를 초기화하고 테스트 데이터를 추가합니다.
  // Jest는 이 프로미스가 해결될 때까지 대기합니다.
  return globalDatabase.clear().then(() => {
    return globalDatabase.insert({testData: 'foo'});
  });
});

test('can find things', () => {
  return globalDatabase.find('thing', {}, results => {
    expect(results.length).toBeGreaterThan(0);
  });
});

test('can insert a thing', () => {
  return globalDatabase.insert('thing', makeThing(), response => {
    expect(response.success).toBeTruthy();
  });
});
```
위의 예제에서 `beforeEach`은 각 테스트가 실행되기 전에 데이터베이스를 초기화하도록 보장합니다.

만약 `beforeEach`가 `describe` 블록 내에 있다면, 해당 `describe` 블록에 속한 각 테스트마다 실행됩니다.

한 번만 실행되는 설정 코드가 아닌 각 테스트마다 실행되는 설정 코드가 필요한 경우에는 beforeAll 대신 `beforeEach`를 사용하세요.

### describe(name, fn)
`describe(name, fn)`은 여러 관련된 테스트를 그룹화하는 블록을 생성합니다. 예를 들어, 맛있지만 시큼하지 않은 `myBeverage` 객체를 테스트하려면 다음과 같이 할 수 있습니다:

```ts
const myBeverage = {
  delicious: true,
  sour: false,
};

describe('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});
```
이것은 필수적인 것은 아닙니다. `test` 블록을 직접 최상위 수준에 작성할 수도 있습니다. 그러나 테스트를 그룹화하는 것이 선호되는 경우가 있습니다.

테스트의 계층 구조가 있는 경우 `describe` 블록을 중첩할 수도 있습니다:

```ts
const binaryStringToNumber = binString => {
  if (!/^[01]+$/.test(binString)) {
    throw new CustomError('Not a binary number.');
  }

  return parseInt(binString, 2);
};

describe('binaryStringToNumber', () => {
  describe('given an invalid binary string', () => {
    test('composed of non-numbers throws CustomError', () => {
      expect(() => binaryStringToNumber('abc')).toThrow(CustomError);
    });

    test('with extra whitespace throws CustomError', () => {
      expect(() => binaryStringToNumber('  100')).toThrow(CustomError);
    });
  });

  describe('given a valid binary string', () => {
    test('returns the correct number', () => {
      expect(binaryStringToNumber('100')).toBe(4);
    });
  });
});
```

### describe.each(table)(name, fn, timeout)
같은 테스트 스위트를 데이터를 바꿔가며 반복적으로 작성하는 경우 `describe.each`를 사용할 수 있습니다. `describe.each`를 사용하면 테스트 스위트를 한 번만 작성하고 데이터를 전달할 수 있습니다.

`describe.each`는 두 가지 API로 제공됩니다:

1. `describe.each(table)(name, fn, timeout)`
- `table`: 테이블 형식의 배열로, 각 행마다 fn에 전달되는 인수를 포함합니다. 1차원 배열의 경우 내부적으로 테이블로 매핑됩니다. 예: `[1, 2, 3] -> [[1], [2], [3]]`.
- `name`: 테스트 스위트의 제목 문자열입니다.
  - 위치에 따라 매개변수를 `printf` 포맷팅을 사용하여 고유한 테스트 제목을 생성할 수 있습니다:
    - `%p` - pretty-format
    - `%s` - 문자열
    - `%d` - 숫자
    - `%i` - 정수
    - `%f` - 부동 소수점 값
    - `%j` - JSON
    - `%o` - 객체
    - `%#` - 테스트 케이스의 인덱스
    - `%%` - 단일 퍼센트 기호 ('%'). 인수를 사용하지 않습니다.
  - 또는 `$variable`를 사용하여 테스트 케이스 개체의 속성을 삽입하여 고유한 테스트 제목을 생성할 수 있습니다. 
- 중첩된 객체 값을 삽입하려면 `$variable.path.to.value`와 같이 키 경로를 사용할 수 있습니다.
- `$#`를 사용해서 테스트케이스의 인덱스에 인젝트 할수 있습니다.
- `%%`을 제외한 printf 포맷팅과 함께 $variable를 사용할 수는 없습니다.

- `fn`: 각 행에 대해 실행될 테스트 스위트입니다. 각 행의 인수가 `Function`의 인수로 전달됩니다.
- 선택적으로 `timeout`(밀리초)을 제공하여 각 행에 대해 중단하기 전까지 대기할 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

예시:

```ts
describe.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected);
  });

  test(`returned value not be greater than ${expected}`, () => {
    expect(a + b).not.toBeGreaterThan(expected);
  });

  test(`returned value not be less than ${expected}`, () => {
    expect(a + b).not.toBeLessThan(expected);
  });
});
```
```ts
describe.each([
  {a: 1, b: 1, expected: 2},
  {a: 1, b: 2, expected: 3},
  {a: 2, b: 1, expected: 3},
])('.add($a, $b)', ({a, b, expected}) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected);
  });

  test(`returned value not be greater than ${expected}`, () => {
    expect(a + b).not.toBeGreaterThan(expected);
  });

  test(`returned value not be less than ${expected}`, () => {
    expect(a + b).not.toBeLessThan(expected);
  });
});
```
2. `describe.eachtable(name, fn, timeout)`
- `table`: 태그된 템플릿 리터럴입니다.
  - 첫 번째 행은 변수 이름 열 제목이 `|`로 구분됩니다.
  - 하나 이상의 후속 행은 `${value}` 구문을 사용하여 템플릿 리터럴 표현식으로 데이터를 제공합니다.
- `name`: 테스트 스위트의 제목 문자열입니다. 표현식에서 테스트 데이터를 테스트 스위트 제목에 주입하기 위해 `$variable`를 사용하며, `$#`을 사용하여 행의 인덱스를 주입합니다.
  - 중첩된 객체 값을 삽입하려면 `$variable.path.to.value`와 같이 키 경로를 사용할 수 있습니다.
- `fn`: 테스트 데이터 개체를 받는 테스트 스위트입니다.
- 선택적으로 `timeout`(밀리초)을 제공하여 각 행에 대해 중단하기 전까지 대기할 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.
예시:

```ts
describe.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('$a + $b', ({a, b, expected}) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected);
  });

  test(`returned value not be greater than ${expected}`, () => {
    expect(a + b).not.toBeGreaterThan(expected);
  });

  test(`returned value not be less than ${expected}`, () => {
    expect(a + b).not.toBeLessThan(expected);
  });
});
```

### describe.only(name, fn)
이 함수는 특정 `describe` 블록만 실행하고 싶을 때 사용합니다:

```ts
describe.only('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});

describe('my other beverage', () => {
  // ... 실행되지 않습니다
});
```
### describe.only.each(table)(name, fn)
`describe.only.each`를 사용하면 특정 테스트 스위트만 실행할 수 있습니다.

`describe.only.each`는 다음 두 가지 API로 제공됩니다:

#### `describe.only.each(table)(name, fn)`
```ts
describe.only.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected);
  });
});

test('will not be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```
#### `describe.only.eachtable(name, fn)`
```ts
describe.only.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added to $b', ({a, b, expected}) => {
  test('passes', () => {
    expect(a + b).toBe(expected);
  });
});

test('will not be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```

### describe.skip(name, fn)
이 함수는 특정 `describe` 블록의 테스트를 실행하지 않고 건너뛰고자 할 때 사용합니다:

```ts
describe('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});

describe.skip('my other beverage', () => {
  // ... 건너뜁니다
});
```
`describe.skip`를 사용하면 테스트를 주석 처리하는 것보다 코드를 더 깔끔하게 만들 수 있습니다. 주의해야 할 점은 `describe` 블록은 여전히 실행된다는 점입니다. 건너뛰어야 할 설정이 있다면 `beforeAll` 또는 `beforeEach` 블록에서 수행하세요.

### `describe.skip.each(table)(name, fn)`
`describe.skip.each`를 사용하면 테스트 스위트의 실행을 중지할 수 있습니다.

`describe.skip.each`는 다음 두 가지 API로 제공됩니다:

#### `describe.skip.each(table)(name, fn)`
```ts
describe.skip.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected); // 실행되지 않습니다
  });
});

test('will be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```

#### `describe.skip.eachtable(name, fn)`
```ts
describe.skip.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added to $b', ({a, b, expected}) => {
  test('will not be run', () => {
    expect(a + b).toBe(expected); // 실행되지 않습니다
  });
});

test('will be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```

### test(name, fn, timeout)
Also under the alias: `it(name, fn, timeout)`

테스트 파일에 필요한 모든 것은 테스트 메서드입니다. 예를 들어, `0`이어야 하는 `inchesOfRain()` 함수를 테스트하는 경우 테스트는 다음과 같을 수 있습니다:

```ts
test('did not rain', () => {
  expect(inchesOfRain()).toBe(0);
});
```
첫 번째 인수는 테스트 이름이고, 두 번째 인수는 테스트할 기대값이 포함된 함수입니다. 세 번째 인수(선택 사항)는 타임아웃(밀리초)으로, 중단하기 전까지 대기할 시간을 지정합니다. 기본 타임아웃은 5초입니다.

테스트에서 **프로미스가 반환되면** Jest는 프로미스가 해결될 때까지 테스트가 완료되지 않습니다. 예를 들어, `fetchBeverageList()`가 레몬이 포함된 리스트로 해결되어야 하는 프로미스를 반환하는 경우 다음과 같이 테스트할 수 있습니다:

```ts
test('has lemon in it', () => {
  return fetchBeverageList().then(list => {
    expect(list).toContain('lemon');
  });
});
```
test 호출은 즉시 반환되지만 프로미스가 해결될 때까지 테스트가 완료되지 않습니다. 자세한 내용은 [Testing Asynchronous Code 페이지](https://jestjs.io/docs/asynchronous)를 참조하세요.

> TIP : `done`과 같은 인수를 test 함수에 제공하면 Jest는 대기하기 위해 대기합니다. 이는 콜백을 테스트하는 경우에 유용합니다.

### test.concurrent(name, fn, timeout)
Also under the alias: `it.concurrent(name, fn, timeout)`

> 주의 : `test.concurrent`는 실험적인 기능으로, 누락된 기능 및 기타 문제에 대한 자세한 내용은 [여기](https://github.com/jestjs/jest/labels/Area%3A%20Concurrent)를 참조하세요.

`test.concurrent`를 사용하면 테스트를 동시에 실행할 수 있습니다.

첫 번째 인수는 테스트 이름이고, 두 번째 인수는 비동기 함수로서 테스트할 기대값을 포함합니다. 세 번째 인수(선택 사항)는 타임아웃(밀리초)으로, 중단하기 전까지 대기할 시간을 지정합니다. 기본 타임아웃은 5초입니다.

```ts
test.concurrent('addition of 2 numbers', async () => {
  expect(5 + 3).toBe(8);
});

test.concurrent('subtraction 2 numbers', async () => {
  expect(5 - 3).toBe(2);
});
```
> TIP : Jest가 동시에 실행되는 테스트 수를 지정한 수보다 많이 실행하지 않도록 `maxConcurrency` 설정 옵션을 사용하세요.

### test.concurrent.each(table)(name, fn, timeout)

`test.concurrent.each`는 동일한 테스트를 다른 데이터로 반복하는 경우 사용됩니다. `test.each`처럼 테스트를 한 번 작성하고 데이터를 전달할 수 있도록 해줍니다. 이러한 테스트는 비동기로 실행됩니다.

`test.concurrent.each`는 두 가지 API로 사용할 수 있습니다:

1. `test.concurrent.each(table)(name, fn, timeout)`
- `table`: 각 행에 대한 테스트 함수에 전달되는 인수로 구성된 배열입니다. 1차원 원시 배열을 전달하면 내부적으로 테이블로 매핑됩니다. 예: [1, 2, 3] -> [[1], [2], [3]]
- `name`: 테스트 블록의 제목인 문자열입니다.
- `printf` 형식 지정을 사용하여 위치에 따라 매개변수를 삽입하여 고유한 테스트 제목을 생성합니다.
- `%p` - [pretty-format](https://www.npmjs.com/package/pretty-format)
- `%s` - 문자열.
- `%d` - 숫자.
- `%i` - 정수.
- `%f` - 부동 소수점 값.
- `%j` - JSON.
- `%o` - 객체.
- `%#` - 테스트 케이스의 인덱스.
- `%%` - 단일 퍼센트 기호 ('%'). 이는 인수를 소비하지 않습니다.
- `fn`: 각 행의 매개변수를 받는 테스트 함수입니다. 이는 비동기 함수여야 합니다.
- 선택적으로 `timeout`(밀리초)을 제공하여 각 행을 중단하기 전까지 기다리는 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.
예시:
```ts
test.concurrent.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', async (a, b, expected) => {
  expect(a + b).toBe(expected);
});
```
2. `test.concurrent.eachtable(name, fn, timeout)`
- `table`: 태그된 템플릿 리터럴
- `|`로 구분된 변수 이름 열 제목의 첫 번째 행
- 템플릿 리터럴 표현을 사용하여 하나 이상의 후속 데이터 행을 제공합니다. `${value}` 구문을 사용합니다.
- `name`: 테스트의 제목인 문자열입니다. 태그된 템플릿 표현식에서 테스트 데이터를 테스트 제목에 삽입하는 데에는 `$variable`을 사용합니다.
- 중첩된 객체 값을 삽입하려면 키 경로를 사용할 수 있습니다. 예: `$variable.path.to.value`
- `fn`: 테스트 데이터 개체를 받는 테스트 함수입니다. 이는 비동기 함수여야 합니다.
- 선택적으로 `timeout`(밀리초)을 제공하여 각 행을 중단하기 전까지 기다리는 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

예시:
```ts
test.concurrent.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added to $b', async ({a, b, expected}) => {
  expect(a + b).toBe(expected);
});
```
### test.concurrent.only.each(table)(name, fn)
`test.concurrent.only.each`는 다른 테스트 데이터와 함께 특정 테스트만 실행하려는 경우에 사용합니다.

`test.concurrent.only.each`도 두 가지 API로 사용할 수 있습니다:

#### `test.concurrent.only.each(table)(name, fn)`
```ts
test.concurrent.only.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', async (a, b, expected) => {
  expect(a + b).toBe(expected);
});

test('will not be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```
#### `test.only.eachtable(name, fn)`
```ts
test.concurrent.only.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added to $b', async ({a, b, expected}) => {
  expect(a + b).toBe(expected);
});

test('will not be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```
### test.concurrent.skip.each(table)(name, fn)
`test.concurrent.skip.each`는 비동기 데이터 주도 테스트의 실행을 중지하려는 경우 사용합니다.

`test.concurrent.skip.each`도 두 가지 API로 사용할 수 있습니다:

#### `test.concurrent.skip.each(table)(name, fn)`
```ts
test.concurrent.skip.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', async (a, b, expected) => {
  expect(a + b).toBe(expected); // 실행되지 않을 것입니다.
});
test('will be run', () => {
  expect(1 / 0).toBe(Infinity);
});
test.concurrent.skip.eachtable(name, fn)
```
```ts
test.concurrent.skip.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added to $b', async ({a, b, expected}) => {
  expect(a + b).toBe(expected); // 실행되지 않을 것입니다.
});

test('will be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```

### test.each(table)(name, fn, timeout)

`test.each`는 동일한 테스트를 다른 데이터로 반복하는 경우 사용됩니다. `test.each`를 사용하면 테스트를 한 번 작성하고 데이터를 전달할 수 있습니다.

`test.each`도 두 가지 API로 사용할 수 있습니다:

#### `test.each(table)(name, fn, timeout)`
- `table`: 각 행에 대한 테스트 함수에 전달되는 인수로 구성된 배열입니다. 1차원 원시 배열을 전달하면 내부적으로 테이블로 매핑됩니다. 예: [1, 2, 3] -> [[1], [2], [3]]
- `name`: 테스트 블록의 제목인 문자열입니다.
  - `printf` 형식 지정을 사용하여 위치에 따라 매개변수를 삽입하여 고유한 테스트 제목을 생성합니다.
    - `%p` - pretty-format.
    - `%s`- 문자열.
    - `%d`- 숫자.
    - `%i` - 정수.
    - `%f` - 부동 소수점 값.
    - `%j` - JSON.
    - `%o` - 객체.
    - `%#` - 테스트 케이스의 인덱스.
    - `%%` - 단일 퍼센트 기호 ('%'). 이는 인수를 소비하지 않습니다.
  - 또는 테스트 케이스 개체의 속성을 `$variable`로 삽입하여 고유한 테스트 제목을 생성할 수 있습니다.
    - 중첩된 객체 값을 삽입하려면 키 경로를 사용할 수 있습니다. 예: `$variable.path.to.value`
    - 테스트 케이스의 인덱스를 `$variable`로 삽입하려면 `$#`을 사용할 수 있습니다.
    - `printf` 형식 지정에서는 `%%`을 제외하고 `$variable`를 사용할 수 없습니다.
- `fn`: 각 행의 매개변수를 받는 테스트 함수입니다.
- 선택적으로 `timeout`(밀리초)을 제공하여 각 행을 중단하기 전까지 기다리는 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

예시:
```ts
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  expect(a + b).toBe(expected);
});
```
```ts
test.each([
  {a: 1, b: 1, expected: 2},
  {a: 1, b: 2, expected: 3},
  {a: 2, b: 1, expected: 3},
])('.add($a, $b)', ({a, b, expected}) => {
  expect(a + b).toBe(expected);
});
```

2. `test.eachtable(name, fn, timeout)`
- `table`: 태그된 템플릿 리터럴
  - `|`로 구분된 변수 이름 열 제목의 첫 번째 행
  - 템플릿 리터럴 표현을 사용하여 하나 이상의 후속 데이터 행을 제공합니다. `${value}` 구문을 사용합니다.
- `name`: 테스트의 제목인 문자열입니다. 태그된 템플릿 표현식에서 테스트 데이터를 테스트 제목에 삽입하는 데에는 `$variable`을 사용합니다.
  - 중첩된 객체 값을 삽입하려면 키 경로를 사용할 수 있습니다. 예: `$variable.path.to.value`
- `fn`: 테스트 데이터 개체를 받는 테스트 함수입니다.
- 선택적으로 `timeout`(밀리초)을 제공하여 각 행을 중단하기 전까지 기다리는 시간을 지정할 수 있습니다. 기본 타임아웃은 5초입니다.

예시:
```ts
test.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added to $b', ({a, b, expected}) => {
  expect(a + b).toBe(expected);
});
```

### test.failing(name, fn, timeout)

`test.failing(name, fn, timeout)` 또는 `it.failing(name, fn, timeout)`은 테스트가 실패할 것으로 예상되는 경우에 사용됩니다. 이러한 테스트는 일반적인 테스트와는 반대로 동작합니다. 실패하는 테스트는 오류를 발생시키면 통과하고, 오류를 발생시키지 않으면 실패합니다.

> BDD(Behavior-Driven Development) 방식으로 코드를 작성하는 경우에 이러한 유형의 테스트를 사용할 수 있습니다. 이 경우 테스트는 통과하지 않은 상태로 나타나지만, 테스트가 통과되면 failing 수정자를 제거하여 테스트를 통과시킬 수 있습니다. <br />또한 버그를 수정하는 방법을 모르더라도 프로젝트에 실패하는 테스트를 기여할 수 있는 좋은 방법일 수 있습니다.

아래는 사용 예시입니다:

```ts
test.failing('it is not equal', () => {
  expect(5).toBe(6); // 이 테스트는 통과됩니다.
});

test.failing('it is equal', () => {
  expect(10).toBe(10); // 이 테스트는 실패합니다.
});
```
### test.failing.each(name, fn, timeout)
`test.failing.each(name, fn, timeout)` 또는 `it.failing.each(table)(name, fn)` 또는 `it.failing.eachtable(name, fn)`을 사용하면 여러 테스트를 한 번에 실행할 수 있습니다. `failing` 다음에 `each`를 추가하여 사용합니다.

아래는 사용 예시입니다:

```ts
test.failing.each([
  {a: 1, b: 1, expected: 2},
  {a: 1, b: 2, expected: 3},
  {a: 2, b: 1, expected: 3},
])('.add($a, $b)', ({a, b, expected}) => {
  expect(a + b).toBe(expected);
});
```

### test.only.failing(name, fn, timeout)
`test.only.failing(name, fn, timeout)` 또는 `it.only.failing(name, fn, timeout)` 또는 `fit.failing(name, fn, timeout)`은 특정 실패하는 테스트만 실행하려는 경우에 사용합니다.

### test.skip.failing(name, fn, timeout)
`test.skip.failing(name, fn, timeout)` 또는 `it.skip.failing(name, fn, timeout)` 또는 `xit.failing(name, fn, timeout)` 또는 `xtest.failing(name, fn, timeout)`은 특정 실패하는 테스트의 실행을 건너뛰려는 경우에 사용합니다.

### test.only(name, fn, timeout)
`test.only(name, fn, timeout)` 또는 `it.only(name, fn, timeout)` 또는 `fit(name, fn, timeout)`는 대형 테스트 파일을 디버깅할 때 특정 테스트만 실행하고자 할 때 사용합니다. `.only`로 지정된 테스트만 실행됩니다.

아래는 사용 예시입니다:

```ts
test.only('it is raining', () => {
  expect(inchesOfRain()).toBeGreaterThan(0);
});

test('it is not snowing', () => {
  expect(inchesOfSnow()).toBe(0);
});
```
보통 `test.only`를 소스 컨트롤에 체크인하지 않으며, 디버깅을 위해 사용한 후 고장난 테스트를 수정한 다음 제거합니다.

### test.only.each(table)(name, fn)
`test.only.each(table)(name, fn)` 또는 `it.only.each(table)(name, fn)` 또는 `fit.each(table)(name, fn)` 또는 `it.only.eachtable(name, fn)` 또는 `fit.eachtable(name, fn)`을 사용하면 다른 테스트 데이터로 특정 테스트만 실행할 수 있습니다.

### test.skip(name, fn)
`test.skip(name, fn)` 또는 `it.skip(name, fn)` 또는 `xit(name, fn)` 또는 `xtest(name, fn)`을 사용하면 임시로 실패하는 테스트를 실행하지 않고 건너뛸 수 있습니다. 이렇게 하면 코드를 삭제하지 않고도 테스트를 건너뛸 수 있습니다.

아래는 사용 예시입니다:

```ts
test('it is raining', () => {
  expect(inchesOfRain()).toBeGreaterThan(0);
});

test.skip('it is not snowing', () => {
  expect(inchesOfSnow()).toBe(0);
});
```
테스트를 주석 처리할 수도 있지만, 들여쓰기와 구문 강조 표시를 유지하는 데에는 test.skip을 사용하는 것이 좋습니다.

### test.skip.each(table)(name, fn)
`test.skip.each(table)(name, fn)` 또는 `it.skip.each(table)(name, fn)` 또는 `xit.each(table)(name, fn)` 또는 `xtest.each(table)(name, fn)` 또는 `it.skip.eachtable(name, fn)` 또는 `xit.eachtable(name, fn)` 또는 `xtest.eachtable(name, fn)`을 사용하면 데이터 주도형 테스트의 실행을 중지할 수 있습니다.

아래는 사용 예시입니다:

```ts
test.skip.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  expect(a + b).toBe(expected); // 실행되지 않습니다.
});

test('will be run', () => {
  expect(1 / 0).toBe(Infinity);
});
```

### test.todo(name)

`test.todo(name)` 또는 `it.todo(name)`은 테스트를 작성할 계획이 있는 경우 사용합니다. 이러한 테스트는 요약 출력에서 강조 표시되어 아직 해야 할 테스트의 수를 알 수 있습니다.

```ts
const add = (a, b) => a + b;

test.todo('add should be associative');
```

> TIP: test.todo는 테스트 콜백 함수를 전달하면 오류가 발생합니다. 이미 테스트를 구현했지만 실행하고 싶지 않은 경우 test.skip을 사용하세요.

## TypeScript Usage

> TypeScript 사용 예시에서는 Jest API를 명시적으로 가져와야 합니다.

```ts
import {expect, jest, test} from '@jest/globals';
```

TypeScript에서는 위의 예시 코드와 동일한 방법으로 사용할 수 있습니다. 다만, TypeScript의 경우 Jest API를 명시적으로 가져와야 하며, 예시 코드에서는 다음과 같이 Jest API를 가져옵니다.

```ts
import {test} from '@jest/globals';
```

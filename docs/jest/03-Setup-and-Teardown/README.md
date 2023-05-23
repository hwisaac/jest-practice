# 설정(Setup) 및 해체(Teardown)
테스트를 작성하는 동안 일부 설정 작업이 테스트가 실행되기 전에 수행되어야 하고, 일부 마무리 작업이 테스트가 완료된 후에 수행되어야 하는 경우가 있습니다. Jest는 이를 처리하기 위한 도우미 함수를 제공합니다.

## 반복 설정
여러 테스트에서 반복적으로 수행해야 할 작업이 있는 경우 `beforeEach`와 `afterEach` 훅을 사용할 수 있습니다.

예를 들어, 여러 테스트가 도시 데이터베이스와 상호 작용하는 경우가 있다고 가정해봅시다. 각 테스트 전에 호출되어야 하는 `initializeCityDatabase()` 메서드와 각 테스트 후에 호출되어야 하는 `clearCityDatabase()` 메서드가 있습니다. 다음과 같이 할 수 있습니다.

```ts
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('도시 데이터베이스에는 Vienna가 포함되어 있어야 합니다', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('도시 데이터베이스에는 San Juan이 포함되어 있어야 합니다', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```
`beforeEach`와 `afterEach`는 테스트와 동일한 방식으로 비동기 코드를 처리할 수 있습니다. 즉, `done` 매개변수를 사용하거나 프로미스를 반환할 수 있습니다. 예를 들어, `initializeCityDatabase()`가 데이터베이스 초기화가 완료될 때 해결되는 프로미스를 반환한다고 가정하면 해당 프로미스를 반환하고자 합니다.

```ts
beforeEach(() => {
  return initializeCityDatabase();
});
```

## 한 번만 설정
일부 경우에는 파일의 시작 부분에서 한 번만 설정을 수행해야 할 수 있습니다. 특히 설정이 비동기적인 경우에는 인라인으로 수행할 수 없습니다. Jest는 이러한 상황을 처리하기 위해 `beforeAll`과 `afterAll` 훅을 제공합니다.

예를 들어, `initializeCityDatabase()`와 `clearCityDatabase()` 모두 프로미스를 반환하고 도시 데이터베이스를 테스트 사이에서 재사용할 수 있는 경우, 테스트 코드를 다음과 같이 변경할 수 있습니다.

```ts
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('도시 데이터베이스에는 Vienna가 포함되어 있어야 합니다', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('도시 데이터베이스에는 San Juan이 포함되어 있어야 합니다', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

## 범위 지정
최상위 `before*` 및 `after*` 훅은 파일의 모든 테스트에 적용됩니다. `describe` 블록 내에 선언된 훅은 해당 `describe` 블록 내의 테스트에만 적용됩니다.

예를 들어, 도시 데이터베이스뿐만 아니라 음식 데이터베이스도 있는 경우 다른 테스트에 대해 다른 설정을 수행할 수 있습니다.

```ts
// 이 파일의 모든 테스트에 적용됩니다.
beforeEach(() => {
  return initializeCityDatabase();
});

test('도시 데이터베이스에는 Vienna가 포함되어 있어야 합니다', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('도시 데이터베이스에는 San Juan이 포함되어 있어야 합니다', () => {
  expect(isCity('San Juan')).toBeTruthy();
});

describe('도시와 음식을 매칭', () => {
  // 이 `describe` 블록 내의 테스트에만 적용됩니다.
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 veal', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```

주의할 점은 최상위 `beforeEach`가 `describe` 블록 내부의 `beforeEach`보다 먼저 실행된다는 것입니다. 모든 훅이 실행되는 순서를 이해하는 데 도움이 될 수 있습니다.

```ts
beforeAll(() => console.log('1 - beforeAll'));
afterAll(() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach(() => console.log('1 - afterEach'));

test('', () => console.log('1 - test'));

describe('Scoped / Nested block', () => {
  beforeAll(() => console.log('2 - beforeAll'));
  afterAll(() => console.log('2 - afterAll'));
  beforeEach(() => console.log('2 - beforeEach'));
  afterEach(() => console.log('2 - afterEach'));

  test('', () => console.log('2 - test'));
});

// 1 - beforeAll
// 1 - beforeEach
// 1 - test
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
```

## 실행 순서(Order of Execution)

Jest는 실제 테스트를 실행하기 전에 테스트 파일의 모든 describe 핸들러를 실행합니다. 이것은 설명 블록 내부의 대신 `before*` 및 `after*` 핸들러 내에서 설정 및 해체 작업을 수행하는 것이 더 좋은 이유입니다. `describe` 블록이 완료되면 Jest는 기본적으로 컬렉션 단계에서 만난 순서대로 테스트를 직렬로 실행하며, 각 테스트가 완료되고 정리되기를 기다립니다.

다음은 설명 테스트 파일과 출력을 보여주는 예입니다.

```ts
describe('describe outer', () => {
  console.log('describe outer-a');

  describe('describe inner 1', () => {
    console.log('describe inner 1');

    test('test 1', () => console.log('test 1'));
  });

  console.log('describe outer-b');

  test('test 2', () => console.log('test 2'));

  describe('describe inner 2', () => {
    console.log('describe inner 2');

    test('test 3', () => console.log('test 3'));
  });

  console.log('describe outer-c');
});

// describe outer-a
// describe inner 1
// describe outer-b
// describe inner 2
// describe outer-c
// test 1
// test 2
// test 3
```

`describe` 및 `test` 블록과 마찬가지로 Jest는 `before*` 및 `after*` 훅을 선언한 순서대로 호출합니다. 주의할 점은 바깥쪽 범위의 `after*` 훅이 먼저 호출된다는 것입니다. 예를 들어, 서로 의존하는 리소스를 설정 및 해체하는 경우 다음과 같이 할 수 있습니다.

```ts
beforeEach(() => console.log('connection setup'));
beforeEach(() => console.log('database setup'));

afterEach(() => console.log('database teardown'));
afterEach(() => console.log('connection teardown'));

test('test 1', () => console.log('test 1'));

describe('extra', () => {
  beforeEach(() => console.log('extra database setup'));
  afterEach(() => console.log('extra database teardown'));

  test('test 2', () => console.log('test 2'));
});

// connection setup
// database setup
// test 1
// database teardown
// connection teardown

// connection setup
// database setup
// extra database setup
// test 2
// extra database teardown
// database teardown
// connection teardown
```

## 일반적인 팁

테스트가 실패하는 경우, 먼저 해당 테스트만 실행했을 때 실패하는지 확인해야 하는 경우가 많습니다. Jest에서 하나의 테스트만 실행하려면 해당 테스트 명령을 임시로 `test.only`로 변경하면 됩니다.

```ts
test.only('이 테스트만 실행됩니다', () => {
  expect(true).toBe(false);
});

test('이 테스트는 실행되지 않습니다', () => {
  expect('A').toBe('A');
});
```

한 개의 테스트가 더 큰 스위트의 일부로 실행될 때 자주 실패하지만 단독으로 실행될 때는 실패하지 않는 경우, 다른 테스트에서 이 테스트와 상호 작용하는 경우일 가능성이 높습니다. 이 경우 `beforeEach`를 사용하여 일부 공유 상태를 지워주면 해결할 수 있습니다. 어떤 공유 상태가 수정되었는지 확실하지 않은 경우 데이터를 로깅하는 `beforeEach`도 사용해 볼 수 있습니다.
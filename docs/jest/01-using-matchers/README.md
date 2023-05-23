# Matchers

Jest는 값을 다양한 방식으로 테스트하기 위해 "매처(matcher)"를 사용합니다. 이 문서에서는 일반적으로 사용되는 매처 몇 가지를 소개합니다. 전체 목록은 expect API 문서를 참조하세요.

## 일반적인 매처
값을 테스트하는 가장 간단한 방법은 정확한 동등성으로 테스트하는 것입니다.

```ts
test('2 더하기 2는 4이다', () => {
  expect(2 + 2).toBe(4);
});
```
이 코드에서 `expect(2 + 2)`는 "기대(expectation)" 객체를 반환합니다. 일반적으로 이러한 기대 객체에 대해 매처를 호출하는 것 외에는 많은 작업을 수행하지 않습니다. 이 코드에서 `.toBe(4)`는 매처입니다. Jest가 실행되면 실패한 매처를 추적하여 오류 메시지를 표시할 수 있습니다.

> `toBe`는 정확한 동등성을 테스트하기 위해 `Object.is`를 사용합니다. 객체의 값을 확인하려면 `toEqual`을 사용하세요:

```ts
test('객체 할당', () => {
  const data = { one: 1 };
  data['two'] = 2;
  expect(data).toEqual({ one: 1, two: 2 });
});
```

`toEqual`은 객체나 배열의 모든 필드를 재귀적으로 확인합니다.

> 팁: `toEqual`은 `undefined` 속성을 가진 객체 키, `undefined` 배열 항목, 배열 희소성 또는 객체 유형 불일치를 무시합니다. 이러한 사항을 고려하려면 대신 `toStrictEqual`을 사용하세요.

또한 매처의 반대 경우를 테스트할 수 있습니다. 이를 위해 `not`을 사용하세요:

```ts
test('양수끼리 더하면 0이 아니다', () => {
  for (let a = 1; a < 10; a++) {
    for (let b = 1; b < 10; b++) {
      expect(a + b).not.toBe(0);
    }
  }
});
```

## 참 거짓(Truthiness)
테스트에서 `undefined`, `null`, `false`를 구분해야 할 때도 있지만 이들을 다르게 처리하고 싶지 않은 경우도 있습니다. Jest에는 명시적으로 원하는 동작을 지정할 수 있는 도우미가 포함되어 있습니다.

- `toBeNull`은 `null`에만 일치합니다.
- `toBeUndefined`는 `undefined`에만 일치합니다.
- `toBeDefined`는 `toBeUndefined`의 반대입니다.
- `toBeTruthy`는 if 문에서 `true`로 처리되는 모든 것에 일치합니다.
- `toBeFalsy`는 if 문에서 `false`로 처리되는 모든 것에 일치합니다.

예를 들면 다음과 같습니다:
```ts
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```
코드가 실제로 하는 작업에 가장 정확하게 대응하는 매처를 사용해야 합니다.

## 숫자
숫자를 비교하는 대부분의 방법에는 매처가 있습니다.

```ts
test('2 더하기 2', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe와 toEqual은 숫자에 대해 동일한 결과를 반환합니다.
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```
부동 소수점 동등성을 비교할 때는 작은 반올림 오류로 인해 테스트가 실패하지 않도록 `toBeCloseTo`를 사용하세요.

```ts
test('부동 소수점 숫자 더하기', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);           반올림 오류로 인해 작동하지 않습니다.
  expect(value).toBeCloseTo(0.3); // 이것은 작동합니다.
});
```

## 문자열
`toMatch`를 사용하여 문자열을 정규식과 일치시킬 수 있습니다.

```ts
test('팀에는 I가 없다', () => {
  expect('team').not.toMatch(/I/);
});

test('하지만 Christoph에는 "stop"이 들어있다', () => {
  expect('Christoph').toMatch(/stop/);
});
```

## 배열과 이터러블
`toContain`을 사용하여 배열이나 이터러블이 특정 항목을 포함하는지 확인할 수 있습니다.

```ts
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'milk',
];

test('쇼핑 목록에는 우유가 있다', () => {
  expect(shoppingList).toContain('milk');
  expect(new Set(shoppingList)).toContain('milk');
});
```

## 예외 처리
특정 함수가 호출될 때 오류를 `throw`하는지 테스트하려면 `toThrow`를 사용하세요.

```ts
function compileAndroidCode() {
  throw new Error('you are using the wrong JDK!');
}

test('안드로이드 컴파일이 예상대로 진행됩니다', () => {
  expect(() => compileAndroidCode()).toThrow();
  expect(() => compileAndroidCode()).toThrow(Error);

  // 오류 메시지에 포함되어야 하는 문자열 또는 정규식을 사용할 수도 있습니다.
  expect(() => compileAndroidCode()).toThrow('you are using the wrong JDK');
  expect(() => compileAndroidCode()).toThrow(/JDK/);

  // 정규식을 사용하여 정확한 오류 메시지를 일치시킬 수도 있습니다.
  expect(() => compileAndroidCode()).toThrow(/^you are using the wrong JDK$/); // 테스트 실패
  expect(() => compileAndroidCode()).toThrow(/^you are using the wrong JDK!$/); // 테스트 통과
});
```

> 팁: 예외를 `throw`하는 함수는 wrapping 함수 내에서 호출되어야 합니다. 그렇지 않으면 `toThrow` assertion은 실패합니다.

## 기타
이것은 간단한 예시입니다. 전체 매처 목록은 참조 문서를 확인하세요.

사용 가능한 매처에 대해 알게 된 후 다음 단계는 Jest가 비동기 코드를 테스트할 수 있도록 하는 방법을 살펴보는 것입니다.
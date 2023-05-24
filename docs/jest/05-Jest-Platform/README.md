# Jest 플랫폼
Jest의 특정 기능을 선택하여 독립적인 패키지로 사용할 수 있습니다. 다음은 사용 가능한 패키지 목록입니다:

## jest-changed-files
git/hg 저장소에서 변경된 파일을 식별하는 도구입니다. 다음 두 가지 함수를 내보냅니다:

- `getChangedFilesForRoots`는 변경된 파일과 저장소가 포함된 객체로 해결되는 프로미스를 반환합니다.
- `findRepos`는 지정된 경로에 포함된 저장소 집합으로 해결되는 프로미스를 반환합니다.
### 예시
```ts
const { getChangedFilesForRoots } = require('jest-changed-files');

// 현재 저장소에서 마지막 커밋 이후로 수정된 파일 집합 출력
getChangedFilesForRoots(['./'], {
  lastCommit: true,
}).then(result => console.log(result.changedFiles));
```
자세한 내용은 `jest-changed-files`의 [readme 파일](https://github.com/jestjs/jest/blob/main/packages/jest-changed-files/README.md)을 참조하십시오.

## jest-diff
데이터 변경 사항을 시각화하는 도구입니다. 임의의 유형의 두 값을 비교하여 차이점을 "예쁘게 출력"하는 문자열을 반환하는 함수를 내보냅니다.

예시
```ts
const { diff } = require('jest-diff');

const a = { a: { b: { c: 5 } } };
const b = { a: { b: { c: 6 } } };

const result = diff(a, b);

// 차이점 출력
console.log(result);
```

## jest-docblock
JavaScript 파일의 상단에 있는 주석을 추출하고 구문 분석하는 도구입니다. 주석 블록 내부의 데이터를 조작하기 위한 여러 함수를 내보냅니다.

### 예시
```ts
const { parseWithComments } = require('jest-docblock');

const code = `
/**
 * This is a sample
 *
 * @flow
 */

 console.log('Hello World!');
`;

const parsed = parseWithComments(code);

// 주석과 프라그마를 포함하는 객체 출력
console.log(parsed);
```

자세한 내용은 `jest-docblock`의 [readme](https://github.com/jestjs/jest/blob/main/packages/jest-docblock/README.md) 파일을 참조하십시오.

## jest-get-type
JavaScript 값의 기본 유형을 식별하는 모듈입니다. 인수로 전달된 값의 유형을 나타내는 문자열을 반환하는 함수를 내보냅니다.

### 예시
```ts
const { getType } = require('jest-get-type');

const array = [1, 2, 3];
const nullValue = null;
const undefinedValue = undefined;

// 'array' 출력
console.log(getType(array));
// 'null' 출력
console.log(getType(nullValue));
// 'undefined' 출력
console.log(getType(undefinedValue));
```

## jest-validate
사용자가 제출한 구성을 유효성 검사하는 도구입니다. 사용자의 구성과 예시 구성 및 기타 옵션을 포함하는 객체 두 개의 인수를 전달하여 사용됩니다. 반환 값은 다음 두 속성을 가진 객체입니다:

- `hasDeprecationWarnings` - 제출된 구성에 사용되는 경고가 있는지 여부를 나타내는 부울 값입니다.
- `isValid` - 구성이 올바른지 여부를 나타내는 부울 값입니다.

### 예시
```ts
const { validate } = require('jest-validate');

const configByUser = {
  transform: '<rootDir>/node_modules/my-custom-transform',
};

const result = validate(configByUser, {
  comment: '  Documentation: http://custom-docs.com',
  exampleConfig: { transform: '<rootDir>/node_modules/babel-jest' },
});

console.log(result);
```
자세한 내용은 jest-`validate`의 [readme 파일](https://github.com/jestjs/jest/blob/main/packages/jest-validate/README.md)을 참조하십시오.

## jest-worker
작업의 병렬화를 위해 사용되는 모듈입니다. `JestWorker`라는 클래스를 내보내며, `Node.js` 모듈의 경로를 사용하여 모듈의 내보낸 메서드를 클래스 메서드처럼 호출할 수 있습니다. 호출한 메서드가 forked 프로세스에서 실행을 완료할 때까지 해결되는 프로미스를 반환합니다.

예시
```ts
// heavy-task.js
module.exports = {
  myHeavyTask: args => {
    // 오래 걸리는 CPU 집약적인 작업.
  },
};
```

```ts
// main.js
async function main() {
  const worker = new Worker(require.resolve('./heavy-task.js'));

  // 서로 다른 인수로 2개의 작업을 병렬로 실행
  const results = await Promise.all([
    worker.myHeavyTask({ foo: 'bar' }),
    worker.myHeavyTask({ bar: 'foo' }),
  ]);

  console.log(results);
}

main();
```
자세한 내용은 `jest-worker`의 [readme 파일](https://github.com/jestjs/jest/blob/main/packages/jest-worker/README.md)을 참조하십시오.

## pretty-format
임의의 JavaScript 값을 사람이 읽기 쉬운 문자열로 변환하는 함수를 내보냅니다. 기본적으로 모든 내장 JavaScript 유형을 지원하며 사용자 정의 플러그인을 통해 응용 프로그램별 유형을 확장할 수 있습니다.

### 예시
```ts
const { format: prettyFormat } = require('pretty-format');

const val = { object: {} };
val.circularReference = val;
val[Symbol('foo')] = 'foo';
val.map = new Map([['prop', 'value']]);
val.array = [-0, Infinity, NaN];

console.log(prettyFormat(val));
```
자세한 내용은 `pretty-format`의 [readme 파일](https://github.com/jestjs/jest/blob/main/packages/pretty-format/README.md)을 참조하십시오.
## 시작하기

```bash
npm install --save-dev jest
```
시작하기 전에 두 숫자를 더하는 가상의 함수에 대한 테스트를 작성해보겠습니다. 먼저 `sum.js` 파일을 생성하세요:

```ts
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```
그런 다음, `sum.test.js`라는 파일을 생성합니다. 이 파일에는 실제 테스트가 포함될 것입니다:

```ts
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```
`package.json`에 다음 섹션을 추가하세요:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```
마지막으로, `yarn test` 또는 `npm test`를 실행하면 Jest가 다음 메시지를 출력합니다:

```bash
PASS  ./sum.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```
**축하합니다! Jest를 사용하여 첫 번째 테스트를 성공적으로 작성했습니다!**

이 테스트는 `expect`와 `toBe`를 사용하여 두 값이 정확히 동일한지 테스트했습니다. Jest가 테스트할 수 있는 다른 기능에 대해서는 "Matchers 사용하기"를 참조하세요.

## 명령줄에서 실행하기
Jest를 CLI에서 직접 실행할 수 있습니다 (`PATH`에 전역으로 설치되어 있는 경우, 예를 들어 `yarn global add jest` 또는 `npm install jest --global`와 같이). 유용한 다양한 옵션을 사용하여 일치하는 파일에서 Jest를 실행할 수 있습니다.

다음은 `my-test`과 일치하는 파일에서 Jest를 실행하고, `config.json`을 구성 파일로 사용하며 실행 후 네이티브 OS 알림을 표시하는 방법입니다:

```bash
jest my-test --notify --config=config.json
```
명령줄에서 Jest를 실행하는 방법에 대해 더 알고 싶다면 Jest CLI 옵션 페이지를 참조하세요.

## 추가 구성
### 기본 구성 파일 생성
프로젝트에 따라 Jest가 몇 가지 질문을 하고 각 옵션에 대한 간단한 설명이 포함된 기본 구성 파일을 생성합니다:

```bash
jest --init
```

### Babel 사용
Babel을 사용하려면 필요한 종속성을 설치하세요:

```bash
npm install --save-dev babel-jest @babel/core @babel/preset-env
```
현재 버전의 Node를 대상으로 Babel을 구성하기 위해 프로젝트의 루트에 `babel.config.js` 파일을 생성하세요:

```ts
// babel.config.js
module.exports = {
  presets: [['@babel/preset-env', {targets: {node: 'current'}}]],
};
```
Babel의 이상적인 구성은 프로젝트에 따라 다를 수 있습니다. 자세한 내용은 Babel 문서를 참조하세요.

#### Babel 구성을 Jest에서 사용할 수 있도록 만들기
Jest는 `process.env.NODE_ENV`를 `'test'`로 설정합니다 (다른 값으로 설정되어 있지 않은 경우). 이를 사용하여 Jest에 필요한 컴파일만 조건부로 설정할 수 있습니다.

```ts
babel.config.js
module.exports = api => {
  const isTest = api.env('test');
  // isTest를 사용하여 필요한 프리셋과 플러그인을 결정할 수 있습니다.

  return {
    // ...
  };
};
```

> 참고: `babel-jest`는 Jest를 설치할 때 자동으로 설치되며, 프로젝트에 babel 구성이 있으면 자동으로 파일을 변환합니다. 이 동작을 피하려면 변환 구성 옵션을 명시적으로 재설정할 수 있습니다:

```ts
// jest.config.js
module.exports = {
  transform: {},
};
```

### webpack 사용
웹팩을 사용하여 에셋, 스타일 및 컴파일을 관리하는 프로젝트에서 Jest를 사용할 수 있습니다. 웹팩은 다른 도구와는 다른 독특한 도전 과제를 제공합니다. 시작하려면 웹팩 가이드를 참조하세요.

### Vite 사용
네이티브 `ESM`으로 소스 코드를 제공하기 위해 vite를 사용하는 프로젝트에서 Jest를 사용할 수 있습니다. vite는 의견이 분분한 도구이며, `out-of-the-box` 워크플로우를 제공합니다. Jest는 vite의 플러그인 시스템 작동 방식 때문에 완전히 지원되지는 않지만, `vite-jest`를 사용한 일부 작동 예제가 있습니다. 완전히 지원되지 않기 때문에 `vite-jest`의 제한 사항을 읽어보는 것이 좋습니다. 시작하려면 vite 가이드를 참조하세요.

### Parcel 사용
프로젝트에서 에셋, 스타일 및 컴파일을 관리하기 위해 `parcel-bundler`를 사용하는 경우 Jest를 사용할 수 있습니다. Parcel은 구성이 필요하지 않습니다. 시작하려면 공식 문서를 참조하세요.

### TypeScript 사용

#### Babel을 통해

Jest는 Babel을 통해 TypeScript를 지원합니다. 

먼저, 위의 Babel 사용 지침을 따랐는지 확인하세요. 그런 다음, `@babel/preset-typescript`를 설치하세요:

```bash
npm install --save-dev @babel/preset-typescript
```
그런 다음, `babel.config.js`에서 프리셋 목록에 `@babel/preset-typescript`를 추가하세요.

```ts
babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {targets: {node: 'current'}}],
    '@babel/preset-typescript',
  ],
};
```
그러나 Babel을 사용하여 TypeScript를 처리할 때 몇 가지 주의 사항이 있습니다. Babel에서의 TypeScript 지원은 순수하게 트랜스파일링이므로 Jest는 테스트를 실행하는 동안 타입 검사를 수행하지 않습니다. 타입 검사를 원한다면 대신 `ts-jest`를 사용하거나 별도로 TypeScript 컴파일러 `tsc`를 실행할 수 있습니다 (또는 빌드 프로세스의 일부로 포함시킬 수 있습니다).

#### ts-jest를 통해
`ts-jest`는 `TypeScript` 프로젝트를 테스트하기 위해 Jest를 사용할 수 있도록 해주는 `TypeScript` 전처리기입니다.

```bash
npm install --save-dev ts-jest
```
Jest가 `ts-jest`를 사용하여 TypeScript를 변환하도록 하려면 구성 파일을 생성해야 할 수도 있습니다.

#### 타입 정의
TypeScript로 작성된 테스트 파일에 대한 Jest 전역 API를 타입 지정하는 두 가지 방법이 있습니다.

`@jest/globals` 패키지를 설치하세요. 이 패키지는 Jest와 함께 제공되는 타입 정의를 제공하며, Jest를 업데이트할 때마다 업데이트됩니다.

```css
npm install --save-dev @jest/globals
```
그리고 이를 import하세요:

```ts
// sum.test.ts
import {describe, expect, test} from '@jest/globals';
import {sum} from './sum';

describe('sum module', () => {
  test('adds 1 + 2 to equal 3', () => {
    expect(sum(1, 2)).toBe(3);
  });
});
```
> 팁: `describe.each/test.each`와 `mock` 함수의 추가 사용 설명서를 참조하세요.

또는 `@types/jest` 패키지를 설치할 수도 있습니다. 이를 통해 Jest의 전역 API에 대한 타입을 제공합니다.

```bash
npm install --save-dev @types/jest
```
> 정보: `@types/jest`는 `DefinitelyTyped`에서 유지되는 제3자 라이브러리이므로 최신 Jest 기능 또는 버전이 아직 포함되지 않을 수 있습니다. 가능하면 Jest와 `@types/jest`의 버전을 가능한 한 일치시키는 것이 좋습니다. 예를 들어, `Jest 27.4.0`을 사용하는 경우 `27.4.x`의 `@types/jest`를 설치하는 것이 이상적입니다.

이전 내용을 기반으로 한 중요한 변경 사항입니다:

TypeScript에서 Babel 대신 `ts-jest`를 사용하는 방법을 소개했습니다.
Jest 전역 API를 TypeScript로 타입 지정하는 방법을 소개했습니다.
# 매뉴얼 목 (Manual Mock)

매뉴얼 목(mock)은 목 데이터로 기능을 대체하는 데 사용됩니다. 예를 들어, 웹 사이트나 데이터베이스와 같은 원격 리소스에 액세스하는 대신 가짜 데이터를 사용할 수 있는 매뉴얼 목을 만들어 테스트를 신속하고 안정적으로 만들 수 있습니다.

## 사용자 모듈 목(mocking)
매뉴얼 목은 모듈과 동일한 디렉토리에 `__mocks__/` 하위 디렉토리에 모듈을 작성하여 정의합니다. 예를 들어, `models` 디렉토리에서 `user`라는 모듈을 목화하려면 `user.js`라는 파일을 만들고 `models/__mocks__` 디렉토리에 넣습니다.

> 주의: `__mocks__` 폴더는 대소문자를 구분하므로 디렉토리 이름을 `__MOCKS__`로 지정하면 일부 시스템에서 문제가 발생할 수 있습니다.

> 참고: 테스트에서 해당 모듈을 require/import 할 때 (실제 구현 대신 매뉴얼 목을 사용하려는 경우) `jest.mock('./moduleName')`을 명시적으로 호출해야 합니다.

## Node 모듈 목(mocking)
목록을 목화하려는 모듈이 Node 모듈인 경우 (예: `lodash`), 목은 `node_modules`와 인접한 `__mocks__` 디렉토리에 배치되어 자동으로 목화됩니다 (루트가 프로젝트 루트가 아닌 다른 폴더를 가리키도록 설정하지 않은 한). `jest.mock('module_name')`를 명시적으로 호출할 필요가 없습니다.

범위가 지정된 모듈(스코프 패키지라고도 함)은 해당 스코프 모듈 이름과 일치하는 디렉토리 구조에 파일을 만들어 목화할 수 있습니다. 예를 들어, `@scope/project-name`이라는 스코프 모듈을 목화하려면 `mocks/@scope/project-name.js`에 파일을 만들어 해당 `@scope/` 디렉토리를 생성합니다.

> 주의사항 : Node의 기본 제공 모듈 (예: `fs` 또는 `path`)을 목화하려면 명시적으로 `jest.mock('path')`와 같이 호출해야 합니다. 기본 제공 모듈은 기본적으로 목화되지 않습니다.

## 예시

```bash
.
├── config
├── __mocks__
│   └── fs.js
├── models
│   ├── __mocks__
│   │   └── user.js
│   └── user.js
├── node_modules
└── views
```

주어진 모듈에 대한 매뉴얼 목이 있는 경우, Jest의 모듈 시스템은 `jest.mock('moduleName')`를 명시적으로 호출할 때 해당 모듈을 사용합니다. 그러나 `automock`가 `true`로 설정된 경우 `jest.mock('moduleName')`이 호출되지 않아도 자동으로 생성된 목 대신 매뉴얼 목 구현이 사용됩니다. 이 동작을 비활성화하려면 특정 모듈 구현을 사용하는 테스트에서 `jest.unmock('moduleName')`을 명시적으로 호출해야 합니다.

> 정보: 올바르게 목화하려면 `jest.mock('moduleName')`을 `require/import` 문과 동일한 스코프에 위치시켜야 합니다.

여기에는 주어진 디렉토리의 모든 파일에 대한 요약을 제공하는 모듈이 있는 가짜 예제가 있습니다. 이 경우, Node의 핵심(`fs`) 모듈을 사용합니다.

```ts
// FileSummarizer.js
'use strict';

const fs = require('fs');

function summarizeFilesInDirectorySync(directory) {
  return fs.readdirSync(directory).map(fileName => ({
    directory,
    fileName,
  }));
}

exports.summarizeFilesInDirectorySync = summarizeFilesInDirectorySync;
```

실제로 디스크에 접근하지 않고 테스트하려는 경우, `fs` 모듈에 대한 매뉴얼 목을 만들어야 합니다. 매뉴얼 목은 자동 목을 확장하는 방식으로 작성됩니다. 매뉴얼 목은 `fs` API의 사용자 정의 버전을 구현하므로 테스트에 사용할 수 있습니다.

```ts
'use strict';
// __mocks__/fs.js

const path = require('path');

const fs = jest.createMockFromModule('fs');

// 테스트에서 사용할 파일 시스템의 목 데이터를 설정하기 위해 테스트 설정 중에 사용할 수 있는 사용자 정의 함수입니다.
let mockFiles = Object.create(null);
function __setMockFiles(newMockFiles) {
  mockFiles = Object.create(null);
  for (const file in newMockFiles) {
    const dir = path.dirname(file);

    if (!mockFiles[dir]) {
      mockFiles[dir] = [];
    }
    mockFiles[dir].push(path.basename(file));
  }
}

// readdirSync의 사용자 정의 버전으로, 특수 목 파일 목록인 __setMockFiles를 읽습니다.
function readdirSync(directoryPath) {
  return mockFiles[directoryPath] || [];
}

fs.__setMockFiles = __setMockFiles;
fs.readdirSync = readdirSync;

module.exports = fs;
```

이제 테스트를 작성합니다. 이 경우, `fs`는 Node의 기본 제공 모듈이므로 명시적으로 `jest.mock('fs')`를 호출해야 합니다.

```ts
'use strict';
// __tests__/FileSummarizer-test.js

jest.mock('fs');

describe('listFilesInDirectorySync', () => {
  const MOCK_FILE_INFO = {
    '/path/to/file1.js': 'console.log("file1 contents");',
    '/path/to/file2.txt': 'file2 contents',
  };

  beforeEach(() => {
    // 각 테스트 전에 목 데이터를 설정합니다.
    require('fs').__setMockFiles(MOCK_FILE_INFO);
  });

  test('디렉토리의 모든 파일이 요약에 포함되어야 합니다.', () => {
    const FileSummarizer = require('../FileSummarizer');
    const fileSummary =
      FileSummarizer.summarizeFilesInDirectorySync('/path/to');

    expect(fileSummary.length).toBe(2);
  });
});
```

여기서 보여지는 예제 목은 `jest.createMockFromModule`[링크](https://jestjs.io/docs/jest-object#jestcreatemockfrommodulemodulename)을 사용하여 자동 목을 생성하고 기본 동작을 재정의합니다. 이것이 권장되는 접근 방식이지만, 완전히 매뉴얼 목을 사용하지 않으려면 자동 목을 사용하거나 확장하는 것이 좋습니다.

매뉴얼 목과 실제 구현이 동기화되도록 하려면 목 파일에서 `jest.requireActual(moduleName)`[링크](https://jestjs.io/docs/jest-object#jestrequireactualmodulename)을 사용하여 실제 모듈을 `require`하는 것이 유용할 수 있습니다. 이후에 mock 함수를 추가로 내보내기 전에 실제 모듈을 수정하면 됩니다.

이 예제의 코드는 `examples/manual-mocks`[링크](https://github.com/jestjs/jest/tree/main/examples/manual-mocks)에서 확인할 수 있습니다.

## ES 모듈 import와 함께 사용하기
ES 모듈 [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)를 사용하는 경우, 일반적으로 테스트 파일의 맨 위에 `import` 문을 위치시키는 경향이 있습니다. 그러나 종종 모듈이 사용하기 전에 Jest에게 목을 사용하도록 지시해야 합니다. 이 때문에 Jest는 `jest.mock` 호출을 모듈의 맨 위로 hoist합니다(임포트 이전에). 이에 대한 자세한 내용과 실제 동작을 보려면 [이 저장소](https://github.com/kentcdodds/how-jest-mocking-works)를 참조하세요.

> 주의: ECMAScript 모듈 지원을 활성화한 경우, `jest.mock` 호출을 모듈 상단으로 hoist할 수 없습니다. ESM 모듈 로더는 코드를 실행하기 전에 정적 임포트를 평가합니다. 자세한 내용은 `ECMAScriptModules`를 참조하세요.

## JSDOM에서 구현되지 않은 메서드를 목화하는 경우
코드가 JSDOM( Jest가 사용하는 DOM 구현)에서 아직 구현되지 않은 메서드를 사용하는 경우, 해당 테스트를 수행할 수 없습니다. 예를 들어 `window.matchMedia()`를 사용하는 코드의 테스트는 쉽게 수행할 수 없습니다. Jest는 `TypeError: window.matchMedia is not a function`과 같은 오류를 반환하고 테스트를 제대로 실행하지 않습니다.

이 경우, 테스트 파일에서 `matchMedia`를 목화하면 문제가 해결될 수 있습니다:

```ts
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // deprecated
    removeListener: jest.fn(), // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```
이 방법은 `window.matchMedia()`가 테스트에서 호출되는 함수(또는 메서드)에서 작동합니다. 테스트하는 파일에서 `window.matchMedia()`를 직접 실행하는 경우, Jest는 동일한 오류를 보고합니다. 이 경우, 해결 방법은 매뉴얼 목을 별도의 파일로 이동시키고 테스트 파일에서 테스트되는 파일 이전에 해당 파일을 포함하는 것입니다:

```ts
import './matchMedia.mock'; // 테스트되는 파일 이전에 가져와야 함
import {myMethod} from './file-to-test';

describe('myMethod()', () => {
  // 메서드를 여기에서 테스트...
});
```
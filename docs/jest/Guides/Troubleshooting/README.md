# 문제 해결
문제가 발생했나요? Jest의 문제를 해결하기 위해 이 가이드를 사용해보세요.

## 테스트가 실패하고 원인을 모르겠을 때
Node에 내장된 디버깅 지원을 사용해보세요. 테스트 중 하나의 테스트에 `debugger;` 문을 넣고, 프로젝트 디렉토리에서 다음 명령을 실행해보세요:

```bash
node --inspect-brk node_modules/.bin/jest --runInBand [다른 인수들]
```

Windows에서는
```bash
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [다른 인수들]
```
이렇게 하면 Jest를 외부 디버거가 연결할 수 있는 Node 프로세스에서 실행합니다. 프로세스는 디버거가 연결될 때까지 일시 중지됩니다.

`Google Chrome`(또는 기반 Chromium 브라우저)에서 디버깅하려면 브라우저를 열고 `chrome://inspect`로 이동한 다음 "Node용 독립적인 DevTools 열기"를 클릭하면 사용 가능한 노드 인스턴스 목록이 표시됩니다. 위의 명령을 실행한 후 터미널에 표시된 주소(일반적으로 `localhost:9229`와 같은)를 클릭하면 Chrome의 DevTools를 사용하여 Jest를 디버깅할 수 있습니다.

Chrome 개발자 도구가 표시되며, Jest CLI 스크립트의 첫 번째 줄에 중단점이 설정됩니다(이는 개발자 도구를 열고 Jest가 실행되기 전에 시간을 주기 위해 수행됩니다). 화면 오른쪽 상단에 "재생" 버튼과 같은 버튼을 클릭하여 실행을 계속할 수 있습니다. Jest가 디버거 문을 포함한 테스트를 실행하면 실행이 일시 중지되고 현재 스코프와 호출 스택을 검사할 수 있습니다.

> 참고: `--runInBand` CLI 옵션을 사용하면 Jest가 개별 테스트에 대해 프로세스를 생성하는 대신 동일한 프로세스에서 테스트를 실행합니다. 일반적으로 Jest는 여러 프로세스에 대해 테스트 실행을 병렬화하지만 동시에 많은 프로세스를 디버그하는 것은 어렵기 때문에 이 옵션을 사용합니다.

## VS Code에서 디버깅
`Visual Studio Code`의 내장 디버거를 사용하여 Jest 테스트를 디버그하는 여러 가지 방법이 있습니다.

내장 디버거를 연결하려면 이전에 언급한 대로 테스트를 실행하세요.

```bash
node --inspect-brk node_modules/.bin/jest --runInBand [다른 인수들]
```
Windows에서는
```bash
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [다른 인수들]
```
그런 다음 다음과 같은 `launch.json` 구성을 사용하여 VS Code의 디버거를 연결하세요.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach",
      "port": 9229
    }
  ]
}
```
테스트를 실행 중인 프로세스에 자동으로 연결하려면 다음 구성을 사용하세요.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/.bin/jest",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```
Windows의 경우 다음 구성을 사용하세요.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/jest/bin/jest.js",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

Facebook의 `create-react-app`을 사용하는 경우 다음 구성으로 Jest 테스트를 디버그할 수 있습니다.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug CRA Tests",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/react-scripts",
      "args": [
        "test",
        "--runInBand",
        "--no-cache",
        "--env=jsdom",
        "--watchAll=false"
      ],
      "cwd": "${workspaceRoot}",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```
노드 디버깅에 대한 자세한 내용은 [여기](https://nodejs.org/api/debugger.html)에서 확인할 수 있습니다.

## WebStorm에서 디버깅
WebStorm은 Jest를 지원합니다. [Testing With Jest in WebStorm](https://blog.jetbrains.com/webstorm/2018/10/testing-with-jest-in-webstorm/)을 읽어보세요.

## 캐싱 문제
변환 스크립트가 변경되었거나 Babel이 업데이트되었는데 변경 사항이 Jest에 인식되지 않는 경우가 있나요?

`--no-cache`로 다시 시도해보세요. Jest는 테스트 실행 속도를 높이기 위해 변환된 모듈 파일을 캐시합니다. 사용자 정의 변환기를 사용하는 경우 변환기에 `getCacheKey` 함수를 추가해보세요: [getCacheKey in Relay](https://github.com/facebook/relay/blob/58cf36c73769690f0bbf90562707eadb062b029d/scripts/jest/preprocessor.js#L56-L61).

## Unresolved Promise 
프로미스가 전혀 해결되지 않으면 다음과 같은 오류가 발생할 수 있습니다:

```bash
- Error: Timeout - Async callback was not invoked within timeout specified by jasmine.DEFAULT_TIMEOUT_INTERVAL.`
```

가장 일반적으로 이 오류는 프로미스 구현 간 충돌로 인해 발생합니다. 예를 들어 다음과 같이 전역 프로미스 구현을 직접 대체할 수 있습니다: `globalThis.Promise = jest.requireActual('promise');` 또는 사용된 프로미스 라이브러리를 하나로 통합해보세요.

테스트가 오래 걸리는 경우 `jest.setTimeout`을 호출하여 타임아웃을 늘릴 수 있습니다.

```ts
jest.setTimeout(10000); // 10초 타임아웃
```

## Watchman 문제
`--no-watchman`으로 Jest를 실행하거나 watchman 구성 옵션을 `false`로 설정해보세요.

또한 [watchman 문제 해결](https://facebook.github.io/watchman/docs/troubleshooting)을 참조하세요.

## `coveragePathIgnorePatterns`가 작동하지 않는 것 같습니다. (`coveragePathIgnorePatterns` seems to not have any effect.)

`babel-plugin-istanbul` 플러그인을 사용하지 않았는지 확인하세요. Jest는 `Istanbul`을 감싸므로 Jest가 커버리지 수집을 위해 어떤 파일을 `instrument`할지를 `Istanbul`에 알립니다. `babel-plugin-istanbul`을 사용하면 Babel에서 처리되는 모든 파일에 커버리지 수집 코드가 있으므로 `coveragePathIgnorePatterns`에 의해 무시되지 않습니다.

## 테스트 정의하기
테스트는 Jest가 테스트를 수집할 수 있도록 동기적으로 정의되어야 합니다.

다음과 같은 예제를 생각해보세요.

```ts
// 이렇게 하지 마세요. 동작하지 않습니다.
setTimeout(() => {
  it('passes', () => expect(1).toBe(1));
}, 0);
```

Jest가 테스트를 수집할 때는 정의를 다음 이벤트 루프의 다음 틱에서 비동기적으로 수행하기 때문에 테스트를 찾을 수 없습니다. 따라서 `test.each`를 사용하는 경우 `beforeEach`/`beforeAll` 내에서 테이블을 비동기적으로 설정할 수 없습니다.





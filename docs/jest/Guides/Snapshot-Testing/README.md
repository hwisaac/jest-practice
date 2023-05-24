## Snapshot Testing

스냅샷 테스트는 UI가 예기치 않게 변경되지 않도록하는 데 매우 유용한 도구입니다.

전형적인 스냅샷 테스트 케이스는 UI 컴포넌트를 렌더링하고 스냅샷을 캡처한 다음 테스트와 함께 저장된 참조 스냅샷 파일과 비교합니다. 두 스냅샷이 일치하지 않으면 테스트가 실패합니다. 이는 변경 사항이 예기치 않은 것이거나 UI 컴포넌트의 새 버전에 대한 참조 스냅샷을 업데이트해야하는 경우입니다.

### Jest를 사용한 스냅샷 테스팅
React 컴포넌트를 테스트할 때도 비슷한 방식을 사용할 수 있습니다. 전체 앱을 빌드해야하는 그래픽 UI를 렌더링하는 대신 테스트 렌더러를 사용하여 React 트리의 직렬화 가능한 값 생성을 빠르게 수행할 수 있습니다. 다음은 [Link 컴포넌트](https://github.com/jestjs/jest/blob/main/examples/snapshot/Link.js)에 대한 [예시 테스트](https://github.com/jestjs/jest/blob/main/examples/snapshot/__tests__/link.test.js)입니다:

```ts
import renderer from 'react-test-renderer';
import Link from '../Link';

it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.facebook.com">Facebook</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```
이 테스트를 처음 실행하면 Jest가 다음과 같은 [스냅샷 파일](https://github.com/jestjs/jest/blob/main/examples/snapshot/__tests__/__snapshots__/link.test.js.snap)을 생성합니다:

```ts
exports[`renders correctly 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;
```
스냅샷 아티팩트는 코드 변경과 함께 커밋되어 코드 리뷰 과정의 일부로 검토되어야합니다. Jest는 코드 리뷰 중에 스냅샷을 사람이 읽을 수 있도록 `pretty-format`을 사용합니다. 이후의 테스트 실행에서 Jest는 렌더링된 출력을 이전 스냅샷과 비교합니다. 일치하는 경우 테스트가 통과됩니다. 일치하지 않는 경우, 이 경우 `<Link>` 컴포넌트의 코드에서 테스트 러너가 버그를 발견했거나 구현이 변경되어 스냅샷을 업데이트해야합니다.

> 참고: 스냅샷은 렌더링하는 데이터에 직접 적용됩니다. 예를 들어, 이 예시에서 `<Link>` 컴포넌트에 전달되는 `page` prop입니다. 이는 다른 파일에서 (예: `App.js`) `<Link>` 컴포넌트에서 prop이 누락된 경우에도 테스트가 통과됩니다. 또한 다른 스냅샷 테스트에서 다른 prop으로 동일한 컴포넌트를 렌더링하는 것은 첫 번째 테스트에 영향을주지 않습니다. 각 테스트는 서로를 알지 못하기 때문입니다.

> 정보: 스냅샷 테스트의 작동 방식 및 왜 생성되었는지에 대한 자세한 내용은 [릴리스 블로그 게시물](https://jestjs.io/blog/2016/07/27/jest-14)에서 찾을 수 있습니다. 스냅샷 테스트를 언제 사용해야하는지에 대한 좋은 이해를 위해 이 블로그 게시물을 읽는 것을 권장합니다. 또한 이 [egghead 비디오](https://egghead.io/lessons/javascript-use-jest-s-snapshot-testing-feature?pl=testing-javascript-with-jest-a36c4074)에서 Jest와 함께 스냅샷 테스트를 배울 것을 권장합니다.

### 스냅샷 업데이트
버그가 도입되어 스냅샷 테스트가 실패하는 경우는 쉽게 발견할 수 있습니다. 그럴 때 문제를 해결하고 스냅샷 테스트가 다시 통과하도록 해야합니다. 이제 의도적인 구현 변경으로 인해 스냅샷 테스트가 실패하는 경우에 대해 이야기해 보겠습니다.

예를 들어, 우리 예시의 `Link` 컴포넌트가 가리키는 주소를 의도적으로 변경하는 경우가 있습니다.

```ts
// 주소가 다른 Link로 업데이트된 테스트 케이스
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.instagram.com">Instagram</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```

이 경우, Jest는 다음 출력을 표시합니다:

![](https://jestjs.io/assets/images/failedSnapshotTest-754bd8c54c17338fe046c6273fd3f9d1.png)

`Link` 컴포넌트를 새 주소로 업데이트했으므로 이 테스트 케이스에 대한 스냅샷이 더 이상 해당 테스트 케이스의 스냅샷 아티팩트와 일치하지 않기 때문에 스냅샷 테스트 케이스가 실패합니다.

이를 해결하기 위해 스냅샷 아티팩트를 업데이트해야합니다. `--updateSnapshot` 플래그를 사용하여 Jest에게 스냅샷을 다시 생성하도록 지시할 수 있습니다:

```bash
jest --updateSnapshot
```

위의 명령을 실행하여 변경 사항을 수락하세요. 원한다면 동일한 기능을 가진 단일 문자 `-u` 플래그를 사용하여 스냅샷을 다시 생성할 수도 있습니다. 이렇게 하면 실패한 스냅샷 테스트에 대한 스냅샷 아티팩트가 모두 다시 생성됩니다. 의도하지 않은 버그로 인해 추가로 실패한 스냅샷 테스트가 있는 경우, 스냅샷이 지저분한 동작을 기록하지 않도록 스냅샷을 다시 생성하기 전에 버그를 수정해야합니다.

스냅샷 테스트를 다시 생성할 특정 테스트 케이스를 제한하려면 추가 `--testNamePattern` 플래그를 전달하여 패턴과 일치하는 테스트에 대해서만 스냅샷을 다시 기록할 수 있습니다.

스냅샷 예제를 클론하고 `Link` 컴포넌트를 수정한 후 Jest를 실행하여 이 기능을 시도해 볼 수 있습니다.

### 대화형 스냅샷 모드
실패한 스냅샷을 대화형으로 업데이트할 수도 있습니다. 감시 모드에서 대화형 스냅샷 모드로 진입할 수 있습니다.

![](https://jestjs.io/assets/images/interactiveSnapshot-58ae38e9cae13140c56d8472453f0595.png)

대화형 스냅샷 모드에 진입하면 Jest가 한 번에 한 번씩 실패한 스냅샷을 보여주고 실패한 출력을 검토할 수 있습니다.

여기서 스냅샷을 업데이트하거나 다음으로 건너뛸 수 있습니다.

![](https://jestjs.io/assets/images/interactiveSnapshotUpdate-a17d8d77f94702048b4d0e0e4c580719.gif)

완료하면 Jest가 요약을 제공하고 감시 모드로 돌아갑니다.

![](https://jestjs.io/assets/images/interactiveSnapshotDone-59ee291ee320accbc4bfc4f33b22638a.png)

### 인라인 스냅샷
인라인 스냅샷은 외부 스냅샷 (`.snap` 파일)과 동일하게 작동하지만 스냅샷 값이 자동으로 소스 코드에 기록됩니다. 이는 올바른 값이 기록되었는지 확인하기 위해 외부 파일로 전환하지 않고도 자동으로 생성된 스냅샷의 이점을 얻을 수 있습니다.

#### 예시:
먼저, 아무 인수도 전달하지 않고 `.toMatchInlineSnapshot()`을 호출하는 테스트를 작성합니다:

```ts
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://example.com">Example Site</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot();
});
```

Jest를 다음에 실행하면 `tree`가 평가되고 스냅샷이 `toMatchInlineSnapshot`의 인수로 작성됩니다:

```ts
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://example.com">Example Site</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot(`
<a
  className="normal"
  href="https://example.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Example Site
</a>
`);
});
```
이게 다입니다! `--updateSnapshot`로 스냅샷을 업데이트하거나 `--watch` 모드에서 `u` 키를 사용하여 스냅샷을 업데이트할 수도 있습니다.

기본적으로 Jest는 스냅샷을 소스 코드에 작성합니다. 그러나 프로젝트에서 `prettier`를 사용하고 있다면 Jest는 이를 감지하고 `prettier`에 작업을 위임합니다(구성도 존중함).

### 속성 Matchers
스냅샷에는 ID 및 날짜와 같이 생성되는 객체의 필드가있는 경우가 많습니다. 이러한 객체를 스냅샷으로 캡처하면 스냅샷이 매번 실행할 때마다 실패합니다:

```ts
it('will fail every time', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot();
});

// 스냅샷
exports[`will fail every time 1`] = `
Object {
  "createdAt": 2018-05-19T23:36:09.816Z,
  "id": 3,
  "name": "LeBron James",
}
`;
```

이 경우, Jest는 어떤 속성에 대한 asymmetric matcher를 제공할 수 있습니다. 이러한 matcher는 스냅샷이 작성되거나 테스트되기 전에 확인되고 수신된 값 대신 스냅샷 파일에 저장됩니다:

```ts
it('will check the matchers and pass', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    id: expect.any(Number),
  });
});

// 스냅샷
exports[`will check the matchers and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "id": Any<Number>,
  "name": "LeBron James",
}
`;
```

matcher가 아닌 값을 지정하는 경우 정확히 확인한 다음 스냅샷에 저장됩니다:

```ts
it('will check the values and pass', () => {
  const user = {
    createdAt: new Date(),
    name: 'Bond... James Bond',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    name: 'Bond... James Bond',
  });
});

// 스냅샷
exports[`will check the values and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "name": 'Bond... James Bond',
}
`;
```

> 팁: 문자열이 아닌 객체의 경우 테스트하기 전에 해당 문자열의 일부를 직접 변경해야합니다. 이를 위해 `replace()`와 정규 표현식을 사용할 수 있습니다.

```ts
const randomNumber = Math.round(Math.random() * 100);
const stringWithRandomData = `<div id="${randomNumber}">Lorem ipsum</div>`;
const stringWithConstantData = stringWithRandomData.replace(/id="\d+"/, 123);
expect(stringWithConstantData).toMatchSnapshot();
```

다른 방법은 스냅샷을 테스트하는 코드에서 무작위 부분을 생성하는 라이브러리를 모킹하는 것입니다.

## 최적의 방법
스냅샷은 응용 프로그램의 예기치 않은 인터페이스 변경을 식별하는 데 훌륭한 도구입니다. API 응답, UI, 로그 또는 오류 메시지와 같은 인터페이스의 변경을 테스트하는 경우 항상 사용할 수 있는 도구입니다. 모든 테스트 전략과 마찬가지로 효과적으로 사용하기 위해 알아야 할 몇 가지 가이드라인과 권장 사항이 있습니다.

### 1. 스냅샷을 코드처럼 취급하세요
스냅샷을 커밋하고 정기적인 코드 리뷰 과정의 일부로 검토하세요. 이는 스냅샷을 프로젝트의 다른 테스트나 코드와 마찬가지로 다루는 것을 의미합니다.

스냅샷이 가독성 있도록 유지하기 위해 집중되고 간결하며 이러한 스타일 가이드를 시행하는 도구 (예: `eslint-plugin-jest`와 `no-large-snapshots` 옵션 또는 `snapshot-diff`와 같은 도구)를 사용하는 것이 유용할 수 있습니다.

목표는 스냅샷을 pull request에서 검토하기 쉽도록하고, 테스트 스위트가 실패 할 때 루트 원인을 검토하는 대신 스냅샷을 다시 생성하는 습관을 막는 것입니다.

### 2. 테스트는 결정론적이어야합니다(Deterministic)
테스트는 결정론적이어야합니다. 변경되지 않은 컴포넌트에서 동일한 테스트를 여러 번 실행하면 항상 동일한 결과가 나와야합니다. 생성된 스냅샷이 플랫폼별이거나 비결정론적인 데이터를 포함하지 않도록 주의해야합니다.

예를 들어, `Date.now()`를 사용하는 Clock 컴포넌트가 있다면 이 컴포넌트로 생성된 스냅샷은 테스트 케이스가 실행될 때마다 다르게 됩니다. 이 경우 `Date.now()` 메서드를 mocking하여 테스트 실행 시 항상 일관된 값을 반환하도록 할 수 있습니다:

```ts
Date.now = jest.fn(() => 1482363367071);
```
이제 스냅샷 테스트 케이스가 실행될 때마다 `Date.now()`가 항상 `1482363367071`을 반환합니다. 따라서 이 컴포넌트에 대해 항상 동일한 스냅샷이 생성됩니다.

### 3. 서술적인 스냅샷 이름을 사용하세요
스냅샷의 이름은 항상 서술적인 테스트 또는 스냅샷 이름을 사용해야합니다. 가장 좋은 이름은 예상되는 스냅샷 콘텐츠를 설명해야합니다. 이렇게하면 리뷰어가 리뷰 중에 스냅샷을 확인하고 더 나은 변경 사항인지 여부를 확인하는 데 도움이되며, 스냅샷이 올바른 동작인지 확인하기 전에 오래된 스냅샷인지 여부를 파악할 수 있습니다.

예를 들어 비교해보세요:

```ts
exports[`<UserName /> should handle some test case`] = `null`;

exports[`<UserName /> should handle some other test case`] = `
<div>
  Alan Turing
</div>
```
대신에 다음과 같은 서술적인 이름을 사용하는 것이 좋습니다:

```ts
exports[`<UserName /> should render null`] = `null`;

exports[`<UserName /> should render Alan Turing`] = `
<div>
  Alan Turing
</div>
```
후자는 출력에서 예상 결과를 정확히 설명하기 때문에 잘못된 경우를 쉽게 알 수 있습니다:

```ts
exports[`<UserName /> should render null`] = `
<div>
  Alan Turing
</div>
`;

exports[`<UserName /> should render Alan Turing`] = `null`;
```

## 자주 묻는 질문

### 스냅샷은 CI(Continuous Integration) 시스템에서 자동으로 작성됩니까?
`Jest 20`부터는 스냅샷은 명시적으로 `--updateSnapshot`를 전달하지 않고 CI 시스템에서 실행할 때 자동으로 작성되지 않습니다. CI에서 실행되는 코드에 스냅샷이 포함되어 있고 새로운 스냅샷이 자동으로 통과되기 때문에 CI 시스템에서 실행되는 테스트 실행에서 스냅샷이 작성되지 않도록 예상됩니다. 스냅샷은 항상 커밋되고 버전 관리 시스템에 보관하는 것이 좋습니다.

### 스냅샷 파일을 커밋해야합니까?
네, 모든 스냅샷 파일은 해당하는 모듈과 테스트와 함께 커밋해야합니다. 스냅샷은 Jest의 다른 어설션 값과 마찬가지로 테스트의 일부로 간주되어야합니다. 사실, 스냅샷은 시간에 따라 소스 모듈의 상태를 나타냅니다. 따라서 소스 모듈이 수정되면 Jest는 이전 버전과의 변경 내용을 알 수 있습니다. 리뷰 중에 스냅샷을 검토할 때 추가적인 컨텍스트를 제공할 수도 있습니다.

### 스냅샷 테스트는 React 컴포넌트에만 적용됩니까?
React 및 React Native 컴포넌트는 스냅샷 테스트에 적합한 사용 사례입니다. 그러나 스냅샷은 직렬화 가능한 값이라면 어떤 값이든 캡처하고 테스트할 수 있으며, 출력이 올바른지 테스트해야하는 경우 언제든지 사용할 수 있습니다. Jest 자체의 출력, Jest의 어설션 라이브러리의 출력 및 Jest 코드베이스의 여러 부분에서의 로그 메시지와 같은 스냅샷을 캡처할 수 있습니다. Jest 저장소에서 CLI 출력의 스냅샷을 캡처하는 예를 볼 수 있습니다.

### 스냅샷 테스트는 단위 테스트를 대체합니까?
스냅샷 테스트는 Jest와 함께 제공되는 20개 이상의 어설션 중 하나에 불과합니다. 스냅샷 테스트의 목표는 기존의 단위 테스트를 대체하는 것이 아니라 추가적인 가치를 제공하고 테스트를 쉽게 만드는 것입니다. 일부 시나리오에서는 특정 기능 집합 (예: React 컴포넌트)의 단위 테스트를 대체할 수 있는 가능성이 있지만, 함께 사용할 수도 있습니다.

### 스냅샷 테스트의 성능은 어떻습니까? 생성된 파일의 크기는 어떻습니까?
Jest는 성능을 고려하여 다시 작성되었으며, 스냅샷 테스트도 예외는 아닙니다. 스냅샷은 텍스트 파일에 저장되므로 이 방법은 빠르고 안정적입니다. Jest는 toMatchSnapshot matcher를 호출하는 각 테스트 파일에 대해 새 파일을 생성합니다. 스냅샷의 크기는 매우 작습니다. 참고로 Jest 코드베이스의 모든 스냅샷 파일 크기는 300KB보다 작습니다.

### 스냅샷 파일 내에서 충돌을 해결하는 방법은 무엇인가요?
스냅샷 파일은 항상 해당하는 모듈의 현재 상태를 나타내야합니다. 따라서 두 개의 브랜치를 병합하다가 스냅샷 파일에서 충돌이 발생하는 경우 충돌을 수동으로 해결하거나 Jest를 실행하여 결과를 검사하고 스냅샷 파일을 업데이트해야합니다.

### 스냅샷 테스트와 TDD(테스트 주도 개발) 원칙을 함께 사용할 수 있습니까?
스냅샷 파일을 수동으로 작성하는 것은 가능하지만 일반적으로 추천되지 않습니다. 스냅샷은 테스트가 컴포넌트의 출력이 변경되었는지 여부를 확인하는 데 도움을주는 도구입니다. 코드를 처음부터 설계하는 데 도움을주는 가이드로 사용되는 것은 일반적으로 어렵습니다.

### 스냅샷 테스트는 코드 커버리지와 함께 작동합니까?
네, 다른 테스트와 마찬가지로 코드 커버리지와 함께 작동합니다.
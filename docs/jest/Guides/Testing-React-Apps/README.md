# Testing React Apps
Facebook에서는 React 애플리케이션을 테스트하기 위해 Jest를 사용합니다.

## 설정
### Create React App으로 설정하기

React를 처음 사용하시는 경우 Create React App을 사용하는 것을 추천합니다. Create React App은 사용 준비가 되어 있으며 Jest가 함께 제공됩니다! 스냅샷을 렌더링하기 위해 `react-test-renderer`만 추가하면 됩니다.

실행
```bash
npm install --save-dev react-test-renderer
```

### Create React App 없이 설정하기
기존 애플리케이션이 있는 경우 잘 작동하도록 몇 가지 패키지를 설치해야 합니다. `babel-jest` 패키지와 `react` babel 프리셋을 사용하여 테스트 환경에서 코드를 변환합니다. [using babel](https://jestjs.io/docs/getting-started#using-babel) 참조하세요.

실행

```bash
npm install --save-dev jest babel-jest @babel/preset-env @babel/preset-react react-test-renderer
```

`package.json` 파일은 다음과 같은 형식이어야 합니다. (`<current-version>`은 해당 패키지의 최신 버전 번호로 실제 버전 번호로 대체해야 합니다). 스크립트 및 jest 구성 항목을 추가해야 합니다.

```json
{
  "dependencies": {
    "react": "<current-version>",
    "react-dom": "<current-version>"
  },
  "devDependencies": {
    "@babel/preset-env": "<current-version>",
    "@babel/preset-react": "<current-version>",
    "babel-jest": "<current-version>",
    "jest": "<current-version>",
    "react-test-renderer": "<current-version>"
  },
  "scripts": {
    "test": "jest"
  }
}
```

`babel.config.js` 파일은 다음과 같이 설정해야 합니다.

```js
module.exports = {
  presets: [
    '@babel/preset-env',
    ['@babel/preset-react', {runtime: 'automatic'}],
  ],
};
```
이제 준비가 되었습니다!

## 스냅샷 테스팅
하이퍼링크를 렌더링하는 Link 컴포넌트에 대한 스냅샷 테스트를 만들어 보겠습니다.

```js
// Link.js
import {useState} from 'react';

const STATUS = {
  HOVERED: 'hovered',
  NORMAL: 'normal',
};

export default function Link({page, children}) {
  const [status, setStatus] = useState(STATUS.NORMAL);

  const onMouseEnter = () => {
    setStatus(STATUS.HOVERED);
  };

  const onMouseLeave = () => {
    setStatus(STATUS.NORMAL);
  };

  return (
    <a
      className={status}
      href={page || '#'}
      onMouseEnter={onMouseEnter}
      onMouseLeave={onMouseLeave}
    >
      {children}
    </a>
  );
}
```

> 참고: 이 예제는 함수 컴포넌트를 사용하지만, 클래스 컴포넌트도 동일한 방식으로 테스트할 수 있습니다. React: 함수 컴포넌트와 클래스 컴포넌트를 [참조](https://legacy.reactjs.org/docs/components-and-props.html#function-and-class-components)하세요.

이제 React의 test renderer와 Jest의 스냅샷 기능을 사용하여 컴포넌트와 상호 작용하고 렌더링된 출력을 캡처하여 스냅샷 파일을 생성해 보겠습니다.

```js
// Link.test.js
import renderer from 'react-test-renderer';
import Link from '../Link';

it('changes the class when hovered', () => {
  const component = renderer.create(
    <Link page="http://www.facebook.com">Facebook</Link>,
  );
  let tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // 수동으로 콜백 트리거
  renderer.act(() => {
    tree.props.onMouseEnter();
  });
  // 다시 렌더링
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // 수동으로 콜백 트리거
  renderer.act(() => {
    tree.props.onMouseLeave();
  });
  // 다시 렌더링
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();
});
```

npm run test 또는 jest를 실행하면 다음과 같은 출력 파일이 생성됩니다.


```js
// tests/snapshots/Link.test.js.snap
exports[`changes the class when hovered 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;

exports[`changes the class when hovered 2`] = `
<a
  className="hovered"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;

exports[`changes the class when hovered 3`] = `
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

다음에 테스트를 실행할 때는 렌더링된 출력이 이전에 생성된 스냅샷과 비교됩니다. 스냅샷은 코드 변경과 함께 커밋되어야 합니다. 스냅샷 테스트가 실패하면 의도된 변경인지 아닌지를 확인해야 합니다. 변경이 의도된 것인 경우 `jest -u` 명령을 사용하여 기존 스냅샷을 덮어쓸 수 있습니다.

이 예제의 코드는 [examples/snapshot](https://github.com/jestjs/jest/tree/main/examples/snapshot)에서 확인할 수 있습니다.

### Mocks, Enzyme 및 React 16+를 사용한 스냅샷 테스팅
Enzyme과 React 16+를 사용하여 스냅샷 테스트를 진행할 때 주의해야 할 점이 있습니다. 다음과 같은 스타일로 모듈을 목킹하는 경우:
```ts
jest.mock('../SomeDirectory/SomeComponent', () => 'SomeComponent');
```
콘솔에 경고 메시지가 표시됩니다:
```bash
Warning: <SomeComponent /> is using uppercase HTML. Always use lowercase HTML tags in React.
```
또는
```bash
Warning: The tag <SomeComponent> is unrecognized in this browser. If you meant to render a React component, start its name with an uppercase letter.
```
React 16은 요소 유형을 확인하는 방식 때문에 이러한 경고가 트리거되며, 목킹된 모듈은 이러한 확인을 통과하지 못합니다. 다음과 같은 옵션이 있습니다:

1. 텍스트로 렌더링합니다. 이 경우 스냅샷에 전달된 모의 컴포넌트의 프롭스를 볼 수 없지만 간단합니다:
```js
jest.mock('./SomeComponent', () => () => 'SomeComponent');
```

2. 사용자 정의 요소로 렌더링합니다. DOM "사용자 정의 요소"는 어떠한 확인도 수행하지 않으며 경고를 표시하지 않습니다. 요소 이름은 소문자로 시작하고 대시가 포함됩니다.
```js
jest.mock('./Widget', () => () => <mock-widget />);
```
3. `react-test-renderer`를 사용합니다. 테스트 렌더러는 요소 유형에 대해 신경쓰지 않으며 `SomeComponent`와 같은 요소를 기껏해야 허용합니다. 스냅샷을 테스트 렌더러를 사용하여 확인하고 `Enzyme`을 사용하여 컴포넌트 동작을 별도로 확인할 수 있습니다.

4. 경고를 완전히 비활성화합니다(일반적으로 추천되지 않습니다):

```js
jest.mock('fbjs/lib/warning', () => require('fbjs/lib/emptyFunction'));
```
일반적으로 이 옵션은 유용한 경고가 무시되기 때문에 선택 사항으로 고려되지 않습니다. 그러나 `react-native`의 컴포넌트를 테스트하는 경우와 같이 `DOM`으로 `react-native` 태그를 렌더링하는 경우 많은 경고가 관련이 없을 수 있습니다. 다른 옵션은 `console.warn`을 바꾸어 경고를 억제하는 것입니다.

## DOM 테스팅
렌더링된 컴포넌트를 어설션하고 조작하려면 `react-testing-library`, `Enzyme` 또는 React의 `TestUtils`를 사용할 수 있습니다. 다음 두 가지 예제는 `react-testing-library` 및 `Enzyme`을 사용합니다.

### react-testing-library

```bash
npm install --save-dev @testing-library/react
```
다음과 같이 두 개의 라벨 사이에서 전환되는 체크박스를 구현해 보겠습니다:

```js
// CheckboxWithLabel.js
import {useState} from 'react';

export default function CheckboxWithLabel({labelOn, labelOff}) {
  const [isChecked, setIsChecked] = useState(false);

  const onChange = () => {
    setIsChecked(!isChecked);
  };

  return (
    <label>
      <input type="checkbox" checked={isChecked} onChange={onChange} />
      {isChecked ? labelOn : labelOff}
    </label>
  );
}
```

```js
// __tests__/CheckboxWithLabel-test.js
import {cleanup, fireEvent, render} from '@testing-library/react';
import CheckboxWithLabel from '../CheckboxWithLabel';

// 주의: @testing-library/react@9.0.0 이상에서는 자동으로 afterEach에 대한 cleanup이 수행됩니다.
// 테스트가 완료되면 DOM을 언마운트하고 정리합니다.
afterEach(cleanup);

it('CheckboxWithLabel changes the text after click', () => {
  const {queryByLabelText, getByLabelText} = render(
    <CheckboxWithLabel labelOn="On" labelOff="Off" />,
  );

  expect(queryByLabelText(/off/i)).toBeTruthy();

  fireEvent.click(getByLabelText(/off/i));

  expect(queryByLabelText(/on/i)).toBeTruthy();
});
```

이 예제의 코드는 `examples/react-testing-library`에서 확인할 수 있습니다.

### Enzyme

```bash
npm install --save-dev enzyme
```
React 버전이 `15.5.0` 미만인 경우 `react-addons-test-utils`도 설치해야 합니다.

`react-testing-library`를 사용하는 대신 `Enzyme`을 사용하여 위의 테스트를 다시 작성해 보겠습니다. 이 예제에서는 `Enzyme`의 `shallow` 렌더러를 사용합니다.

```js
// __tests__/CheckboxWithLabel-test.js
import Enzyme, {shallow} from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
import CheckboxWithLabel from '../CheckboxWithLabel';

Enzyme.configure({adapter: new Adapter()});

it('CheckboxWithLabel changes the text after click', () => {
  // 문서에 레이블이 있는 체크박스를 렌더링합니다.
  const checkbox = shallow(<CheckboxWithLabel labelOn="On" labelOff="Off" />);

  expect(checkbox.text()).toBe('Off');

  checkbox.find('input').simulate('change');

  expect(checkbox.text()).toBe('On');
});
```
이 예제의 코드는 [examples/enzyme](https://github.com/jestjs/jest/tree/main/examples/enzyme)에서 확인할 수 있습니다.

## 사용자 정의 변환기
더 고급 기능이 필요한 경우 사용자 정의 변환기를 작성할 수도 있습니다. `babel-jest` 대신 `@babel/core`를 사용하는 예제입니다.

```js
'use strict';
// custom-transformer.js
const {transform} = require('@babel/core');
const jestPreset = require('babel-preset-jest');

module.exports = {
  process(src, filename) {
    const result = transform(src, {
      filename,
      presets: [jestPreset],
    });

    return result || src;
  },
};
```

이 예제가 작동하려면 `@babel/core`와 `babel-preset-jest` 패키지를 설치해야 합니다.

Jest 구성을 다음과 같이 업데이트하여 사용할 수 있습니다: `"transform": {"\.js$": "path/to/custom-transformer.js"}`.

babel 지원이 있는 변환기를 작성하려면 `babel-jest`를 사용하여 사용자 정의 구성 옵션을 전달할 수도 있습니다:

```js
const babelJest = require('babel-jest');

module.exports = babelJest.createTransformer({
  presets: ['my-custom-preset'],
});
```

자세한 내용은 [해당 문서](https://jestjs.io/docs/code-transformation#writing-custom-transformers)를 참조하세요.

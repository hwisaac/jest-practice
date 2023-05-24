# ES6 Class Mocks
Jest는 테스트하려는 파일에 가져온 ES6 클래스를 모킹하는 데 사용할 수 있습니다.

ES6 클래스는 구문적 설탕을 포함한 생성자 함수입니다. 따라서 ES6 클래스의 모킹은 함수 또는 실제 ES6 클래스(즉, 또 다른 함수)인 모킹 함수를 사용해야 합니다. 따라서 [모킹 함수](https://jestjs.io/docs/mock-functions)를 사용하여 이러한 클래스를 모킹할 수 있습니다.

## ES6 클래스 예제
우리는 사운드 파일을 재생하는 `SoundPlayer` 클래스와 해당 클래스를 사용하는 소비자 클래스인 `SoundPlayerConsumer`의 가짜 예제를 사용할 것입니다. `SoundPlayer`를 테스트용으로 모킹할 것입니다.

```ts
// sound-player.js
export default class SoundPlayer {
  constructor() {
    this.foo = 'bar';
  }

  playSoundFile(fileName) {
    console.log('Playing sound file ' + fileName);
  }
}
```
```ts
// sound-player-consumer.js
import SoundPlayer from './sound-player';

export default class SoundPlayerConsumer {
  constructor() {
    this.soundPlayer = new SoundPlayer();
  }

  playSomethingCool() {
    const coolSoundFileName = 'song.mp3';
    this.soundPlayer.playSoundFile(coolSoundFileName);
  }
}
```
## ES6 클래스를 모킹하는 4가지 방법
### 자동 모킹
`jest.mock('./sound-player')`를 호출하면 "자동 모킹"이라는 유용한 모킹이 반환됩니다. 이 모킹은 클래스 생성자와 모든 메서드 호출을 감시하는 데 사용됩니다. ES6 클래스를 모킹하기 위해 해당 클래스를 모킹된 생성자로 대체하고, 모든 메서드를 항상 `undefined`를 반환하는 모킹 함수로 대체합니다. 메서드 호출은 `AutomaticMock.mock.instances[index].methodName.mock.calls`에 저장됩니다.

참고: 클래스 내에서 화살표 함수를 사용하는 경우, 모킹에 포함되지 않습니다. 이유는 화살표 함수가 객체의 프로토타입에 존재하지 않고, 단지 함수에 대한 참조를 보유하는 속성이기 때문입니다.

클래스의 구현을 대체할 필요가 없는 경우, 이 방법은 설정하기 가장 쉬운 옵션입니다. 예를 들어:

```ts
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';
jest.mock('./sound-player'); // SoundPlayer는 이제 모킹된 생성자입니다.

beforeEach(() => {
  // 생성자와 모든 메서드에 대한 모든 인스턴스 및 호출을 지웁니다:
  SoundPlayer.mockClear();
});

it('소비자가 클래스 생성자를 호출했는지 확인할 수 있습니다', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('소비자가 클래스 인스턴스의 메서드를 호출했는지 확인할 수 있습니다', () => {
  // mockClear()가 작동하는지 보여줍니다:
  expect(SoundPlayer).not.toHaveBeenCalled();

  const soundPlayerConsumer = new SoundPlayerConsumer();
  // 생성자가 다시 호출되어야 합니다:
  expect(SoundPlayer).toHaveBeenCalledTimes(1);

  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();

  // mock.instances는 자동 모킹에서 사용할 수 있습니다:
  const mockSoundPlayerInstance = SoundPlayer.mock.instances[0];
  const mockPlaySoundFile = mockSoundPlayerInstance.playSoundFile;
  expect(mockPlaySoundFile.mock.calls[0][0]).toBe(coolSoundFileName);
  // 위와 동일한 확인:
  expect(mockPlaySoundFile).toHaveBeenCalledWith(coolSoundFileName);
  expect(mockPlaySoundFile).toHaveBeenCalledTimes(1);
});
```

### 매뉴얼 모킹(Manual Mock)
`__mocks__` 폴더에 모킹된 구현을 저장하여 매뉴얼 모킹을 생성할 수 있습니다. 이렇게 하면 구현을 지정할 수 있으며, 테스트 파일 전체에서 사용할 수 있습니다.

```ts
// __mocks__/sound-player.js
// 테스트 파일에서 이 이름을 가져오세요:
export const mockPlaySoundFile = jest.fn();
const mock = jest.fn().mockImplementation(() => {
  return {playSoundFile: mockPlaySoundFile};
});

export default mock;
```
모킹과 공유되는 `mockPlaySoundFile` 메서드를 가져와서 모킹을 가져옵니다:

```ts
// sound-player-consumer.test.js
import SoundPlayer, {mockPlaySoundFile} from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';
jest.mock('./sound-player'); // SoundPlayer는 이제 모킹된 생성자입니다.

beforeEach(() => {
  // 생성자와 모든 메서드에 대한 모든 인스턴스 및 호출을 지웁니다:
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});

it('소비자가 클래스 생성자를 호출했는지 확인할 수 있습니다', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('소비자가 클래스 인스턴스의 메서드를 호출했는지 확인할 수 있습니다', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();
  expect(mockPlaySoundFile).toHaveBeenCalledWith(coolSoundFileName);
});
```

### 모듈 팩토리 매개변수를 사용하여 `jest.mock()` 호출
`jest.mock(path, moduleFactory)`는 모듈 팩토리(module factory) 인수를 사용합니다. 모듈 팩토리는 모킹을 반환하는 함수입니다.

생성자 함수를 모킹하기 위해 모듈 팩토리는 생성자 함수를 반환해야 합니다. 다시 말해, 모듈 팩토리는 함수를 반환하는 함수, 즉 고차 함수(HOF)여야 합니다.

```ts
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});
```

> 주의: `jest.mock()` 호출은 코드 상단으로 호이스팅되므로 범위 외 변수에 대한 접근을 방지합니다. 기본적으로 jest는 단어 `mock`로 시작하는 변수에 대해 이러한 점검을 비활성화합니다. 그러나 초기화가 제시간에 이루어지도록 보장하는 것은 여전히 사용자의 책임입니다. [Temporal Dead Zone](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz)에 주의하세요.

> TDZ(Teporal Dead Zone) : Temporal Dead Zone (TDZ)는 변수가 선언되었지만 아직 초기화되지 않은 상태일 때 발생하는 자바스크립트의 동작입니다. TDZ는 변수에 접근할 때 발생하며, 해당 변수에 대한 어떠한 동작도 수행할 수 없습니다. 이는 변수가 선언되기 전에 변수를 사용하려는 시도를 막아줍니다.

예를 들면 다음은 변수 선언에서 `mock` 대신 `fake`를 사용하여 범위 외 오류가 발생합니다.

```ts
// 주의: 이렇게 하면 실패합니다.
import SoundPlayer from './sound-player';
const fakePlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: fakePlaySoundFile};
  });
});
```
다음은 `mockSoundPlayer`가 화살표 함수로 감싸져 있지 않기 때문에 호이스팅 후 초기화되기 전에 액세스되므로 `ReferenceError`가 발생합니다.

```ts
import SoundPlayer from './sound-player';
const mockSoundPlayer = jest.fn().mockImplementation(() => {
  return {playSoundFile: mockPlaySoundFile};
});
// 에러가 발생합니다.
jest.mock('./sound-player', () => {
  return mockSoundPlayer;
});
```

### `mockImplementation()` 또는 `mockImplementationOnce()`를 사용하여 모킹 대체
기존 모킹에 대해 구현을 변경하여 단일 테스트 또는 모든 테스트에 대한 구현을 변경할 수 있습니다. 기존 모킹에 대해 `mockImplementation()`을 호출하여 변경할 수 있습니다.

`jest.mock()`의 모듈 팩토리 매개변수 대신 기존 모킹에 `mockImplementation()` (또는 `mockImplementationOnce()`)을 호출하면 `jest.mock()` 호출을 사용하지 않고도 테스트 간에 모킹을 변경할 수 있습니다.

```ts
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';

jest.mock('./sound-player');

describe('SoundPlayer가 오류를 throw할 때', () => {
  beforeAll(() => {
    SoundPlayer.mockImplementation(() => {
      return {
        playSoundFile: () => {
          throw new Error('테스트 오류');
        },
      };
    });
  });

  it('playSomethingCool을 호출할 때 오류를 throw해야 합니다', () => {
    const soundPlayerConsumer = new SoundPlayerConsumer();
    expect(() => soundPlayerConsumer.playSomethingCool()).toThrow();
  });
});
```

## 자세히 알아보기: 모킹 생성자 함수 이해
`jest.fn().mockImplementation()`을 사용하여 생성자 함수 모킹을 구축하면 모킹이 실제보다 복잡해 보일 수 있습니다. 이 섹션에서는 모킹이 작동하는 방식을 설명하기 위해 직접 모킹을 생성하는 방법을 보여줍니다.

### 매뉴얼 모킹이 다른 ES6 클래스인 경우
모킹된 클래스와 동일한 파일 이름을 `__mocks__` 폴더에 정의하면 모킹으로 사용됩니다. 이 클래스는 실제 클래스 대신 사용됩니다. 이렇게 하면 테스트용으로 테스트 구현을 주입할 수 있지만 호출을 감시할 수는 없습니다.

위 예제에서 모킹은 다음과 같을 수 있습니다:

```ts
// __mocks__/sound-player.js
export default class SoundPlayer {
  constructor() {
    console.log('Mock SoundPlayer: constructor was called');
  }

  playSoundFile() {
    console.log('Mock SoundPlayer: playSoundFile was called');
  }
}
```

### 모듈 팩토리 매개변수를 사용한 모킹
`jest.mock(path, moduleFactory)`에 전달되는 모듈 팩토리 함수는 함수를 반환할 수 있는 HOF(Higher-Order Function)일 수 있습니다*. 이렇게 하면 모킹된 객체에서 `new`를 호출할 수 있게 됩니다. 다시 말해, 테스트용으로 다른 동작을 주입할 수는 있지만 호출을 감시할 수는 없습니다.

#### * 모듈 팩토리 함수는 함수를 반환해야 함
생성자 함수를 모킹하려면 모듈 팩토리 함수가 생성자 함수를 반환해야 합니다. 다시 말해, 모듈 팩토리 함수는 함수를 반환하는 함수, 즉 고차 함수(HOF)여야 합니다.

```ts
jest.mock('./sound-player', () => {
  return function () {
    return {playSoundFile: () => {}};
  };
});
```

> 참고: 모킹은 화살표 함수일 수 없습니다. JavaScript에서는 화살표 함수에서 new를 호출하는 것이 허용되지 않기 때문입니다. 따라서 다음은 작동하지 않습니다.

```ts
jest.mock('./sound-player', () => {
  return () => {
    // 동작하지 않습니다. 화살표 함수에서 new를 호출할 수 없음
    return {playSoundFile: () => {}};
  };
});
```
이 경우 `@babel/preset-env`와 같은 도구를 사용하여 코드를 `ES5`로 변환하여 JavaScript가 아닌 `ES5`로 변환하는 것이 일반적입니다. (`ES5`에는 화살표 함수나 클래스가 없으므로 이 둘은 모두 일반 함수로 변환됩니다.)

## 클래스의 특정 메서드 모킹
`SoundPlayer` 클래스의 `playSoundFile` 메서드를 모킹하거나 호출을 감시하고 싶다고 가정해 봅시다. 간단한 예를 살펴보겠습니다:

```ts
// jest 테스트 파일 아래에 작성하세요
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';

const playSoundFileMock = jest
  .spyOn(SoundPlayer.prototype, 'playSoundFile')
  .mockImplementation(() => {
    console.log('모킹된 함수');
  }); // "감시"만 하려면 이 줄을 주석 처리하세요

it('consumer는 음악을 재생할 수 있어야 합니다', () => {
  const player = new SoundPlayerConsumer();
  player.playSomethingCool();
  expect(playSoundFileMock).toHaveBeenCalled();
});
```

### 정적, getter 및 setter 메서드
`SoundPlayer` 클래스에 getter 메서드 `foo`와 static 메서드 `brand`가 있다고 가정해 봅시다.

```ts
export default class SoundPlayer {
  constructor() {
    this.foo = 'bar';
  }

  playSoundFile(fileName) {
    console.log('Playing sound file ' + fileName);
  }

  get foo() {
    return 'bar';
  }
  static brand() {
    return 'player-brand';
  }
}
```

이러한 메서드를 모킹/감시하는 것은 쉽습니다. 다음은 예시입니다:

```ts
// jest 테스트 파일 아래에 작성하세요
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';

const staticMethodMock = jest
  .spyOn(SoundPlayer, 'brand')
  .mockImplementation(() => 'some-mocked-brand');

const getterMethodMock = jest
  .spyOn(SoundPlayer.prototype, 'foo', 'get')
  .mockImplementation(() => 'some-mocked-result');

it('사용자 정의 메서드가 호출되었는지 확인합니다', () => {
  const player = new SoundPlayer();
  const foo = player.foo;
  const brand = SoundPlayer.brand();

  expect(staticMethodMock).toHaveBeenCalled();
  expect(getterMethodMock).toHaveBeenCalled();
});
```

## 사용 추적하기 (모킹 감시하기)
테스트 구현을 주입하는 것은 도움이 되지만, 클래스 생성자와 메서드가 올바른 매개변수로 호출되었는지 테스트하기도 원할 것입니다.

### 생성자 감시하기 (Spying on the constructor)
생성자 호출을 추적하려면 HOF에 의해 반환된 함수 대신 jest 모킹 함수로 이전 함수를 바꿉니다. `jest.fn()`으로 생성하고 `mockImplementation()`을 사용하여 구현을 지정합니다.

```ts
import SoundPlayer from './sound-player';
jest.mock('./sound-player', () => {
  // 작동하고 생성자 호출 여부를 확인할 수 있습니다:
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: () => {}};
  });
});
```
이렇게 하면 `SoundPlayer.mock.calls`를 사용하여 모킹된 클래스의 사용 기록을 검사할 수 있습니다. 예를 들어 `expect(SoundPlayer).toHaveBeenCalled()` 또는 거의 동등한 표현 `expect(SoundPlayer.mock.calls.length).toBeGreaterThan(0)`과 같은 테스트를 수행할 수 있습니다.

### Non-default 클래스 모킹하기
클래스가 모듈의 기본 내보내기가 **아닌 경우**, 클래스 내보내기 이름과 동일한 키를 가진 객체를 반환해야 합니다.

```ts
import {SoundPlayer} from './sound-player';
jest.mock('./sound-player', () => {
  // 작동하고 생성자 호출 여부를 확인할 수 있습니다:
  return {
    SoundPlayer: jest.fn().mockImplementation(() => {
      return {playSoundFile: () => {}};
    }),
  };
});
```

### 클래스의 메서드 감시하기(Spying on methods of our class)
모킹된 클래스는 테스트 중에 생성된 개체마다 새로운 객체가 생성됩니다. 이러한 객체에서 메서드 호출을 감시하기 위해 `playSoundFile`을 다른 모킹 함수로 채워 넣고 테스트 파일에서 동일한 모킹 함수에 대한 참조를 저장하여 테스트 중에 사용할 수 있도록 합니다.

```ts
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
    // 이제 playSoundFile 호출을 추적할 수 있습니다.
  });
});
```

이를 매뉴얼 모킹으로 표현하면 다음과 같습니다:

```ts
// __mocks__/sound-player.js
// 이 이름을 테스트 파일에서 가져옵니다.
export const mockPlaySoundFile = jest.fn();
const mock = jest.fn().mockImplementation(() => {
  return {playSoundFile: mockPlaySoundFile};
});

export default mock;
```

사용법은 모듈 팩토리 함수와 유사하지만 `jest.mock()` 호출에서 두 번째 인수를 생략할 수 있으며, 테스트 파일에서 모킹된 메서드를 가져와야 합니다. 이 경우 해당 모듈 경로를 사용하고 `__mocks__`를 포함하지 않아야 합니다.

### 테스트 간 정리 (Cleaning up between tests)

모킹된 생성자 함수와 해당 메서드의 호출 기록을 지우려면 `beforeEach()` 함수에서 `mockClear()`를 호출합니다.

```ts
beforeEach(() => {
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});
```

## 전체 예시(Complete example)
다음은 `jest.mock()`에 모듈 팩토리 매개변수를 사용하여 완전한 테스트 파일 예시입니다.

```ts
// sound-player-consumer.test.js
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';

const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});

beforeEach(() => {
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});

it('consumer는 SoundPlayer에서 new()를 호출할 수 있어야 합니다', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  // 생성자가 객체를 생성했는지 확인합니다:
  expect(soundPlayerConsumer).toBeTruthy();
});

it('consumer가 클래스 생성자를 호출했는지 확인할 수 있습니다', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('consumer가 클래스 인스턴스의 메서드를 호출했는지 확인할 수 있습니다', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();
  expect(mockPlaySoundFile.mock.calls[0][0]).toBe(coolSoundFileName);
});
```
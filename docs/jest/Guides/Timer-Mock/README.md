# Timer Mocks

네이티브 타이머 함수들인 `setTimeout()`, `setInterval()`, `clearTimeout()`, `clearInterval()`은 실제 시간의 경과에 의존하기 때문에 테스트 환경에서는 이상적이지 않습니다. Jest는 타이머를 제어할 수 있는 함수로 교체하는 기능을 제공합니다. 멋지게 해결되었군요!

정보: Fake Timers API 문서도 참조하세요.

## Fake Timers 활성화
다음 예시에서는 `jest.useFakeTimers()`를 호출하여 가짜 타이머를 활성화합니다. 이는 `setTimeout()`과 다른 타이머 함수의 원래 구현을 대체합니다. 타이머는 `jest.useRealTimers()`를 사용하여 정상 동작으로 복원할 수 있습니다.

```ts
// timerGame.js
function timerGame(callback) {
  console.log('준비... 시작!');
  setTimeout(() => {
    console.log("시간 초과 -- 중지!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```

```ts
// __tests__/timerGame-test.js
jest.useFakeTimers();
jest.spyOn(global, 'setTimeout');

test('게임이 종료되기 전에 1초를 기다립니다.', () => {
  const timerGame = require('../timerGame');
  timerGame();

  expect(setTimeout).toHaveBeenCalledTimes(1);
  expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);
});
```

## 모든 타이머 실행하기
이 모듈에 대해 작성할 수 있는 또 다른 테스트는 1초 후에 콜백이 호출되는지 확인하는 것입니다. 이를 위해 Jest의 타이머 제어 API를 사용하여 테스트 중에 시간을 빠르게 진행시킬 것입니다:

```ts
jest.useFakeTimers();
test('1초 후에 콜백을 호출합니다.', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // 현재 시점에서는 콜백이 아직 호출되지 않아야 합니다.
  expect(callback).not.toBeCalled();

  // 모든 타이머가 실행될 때까지 시간을 빠르게 진행시킵니다.
  jest.runAllTimers();

  // 이제 콜백이 호출되었어야 합니다!
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```

## 대기 중인 타이머 실행하기
재귀적인 타이머가 있는 시나리오도 있을 수 있습니다. 이는 타이머 콜백에서 새로운 타이머를 설정하는 타이머입니다. 이 경우 모든 타이머를 실행하면 무한 루프가 발생하여 `"Aborting after running 100000 timers, assuming an infinite loop!"`라는 오류가 발생합니다.

이럴 경우 `jest.runOnlyPendingTimers()`를 사용하여 문제를 해결할 수 있습니다:

```ts
// infiniteTimerGame.js
function infiniteTimerGame(callback) {
  console.log('준비... 시작!');

  setTimeout(() => {
    console.log("시간 초과! 다음 게임 시작까지 10초 남았습니다...");
    callback && callback();

    // 10초 후에 다음 게임 예약
    setTimeout(() => {
      infiniteTimerGame(callback);
    }, 10000);
  }, 1000);
}

module.exports = infiniteTimerGame;
```
```ts
// __tests__/infiniteTimerGame-test.js
jest.useFakeTimers();
jest.spyOn(global, 'setTimeout');

describe('infiniteTimerGame', () => {
  test('1초 후에 10초 타이머를 예약합니다.', () => {
    const infiniteTimerGame = require('../infiniteTimerGame');
    const callback = jest.fn();

    infiniteTimerGame(callback);

    // 현재 시점에서 1초 후에 게임 종료를 예약하기 위해 setTimeout이 한 번 호출되어야 합니다.
    expect(setTimeout).toHaveBeenCalledTimes(1);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);

    // 현재 대기 중인 타이머만 실행합니다.
    // (이 프로세스 동안 생성된 새로운 타이머는 실행하지 않음)
    jest.runOnlyPendingTimers();

    // 이 시점에서 1초 타이머의 콜백이 실행되어야 합니다.
    expect(callback).toBeCalled();

    // 그리고 10초 후에 게임을 다시 시작하기 위해 새로운 타이머를 생성했어야 합니다.
    expect(setTimeout).toHaveBeenCalledTimes(2);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 10000);
  });
});
```

> 참고: 디버깅이나 다른 목적으로 타이머가 실행되기 전에 오류가 발생하는 횟수를 변경할 수 있습니다:

```ts
jest.useFakeTimers({timerLimit: 100});
```

## 시간에 따라 타이머 진행하기
`jest.advanceTimersByTime(msToRun)`를 사용하는 것도 가능합니다. 이 API를 호출하면 모든 타이머가 `msToRun` 밀리초만큼 진행됩니다. 이 시간 동안 실행되어야 하는 모든 대기 중인 "매크로 태스크"(`setTimeout()`이나 `setInterval()`을 통해 예약된 태스크)는 실행됩니다. 또한 이러한 매크로 태스크가 동일한 시간 프레임 내에서 실행되도록 예약한 새 매크로 태스크도 실행됩니다. `msToRun` 밀리초 내에 실행해야 할 대기 중인 매크로 태스크가 없을 때까지 계속해서 실행됩니다.

```ts
// timerGame.js
function timerGame(callback) {
  console.log('준비... 시작!');
  setTimeout(() => {
    console.log("시간 초과 -- 중지!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```
```ts
// __tests__/timerGame-test.js
jest.useFakeTimers();
it('advanceTimersByTime을 사용하여 1초 후에 콜백을 호출합니다.', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // 현재 시점에서는 콜백이 아직 호출되지 않아야 합니다.
  expect(callback).not.toBeCalled();

  // 모든 타이머가 실행될 때까지 시간을 1초만큼 진행시킵니다.
  jest.advanceTimersByTime(1000);

  // 이제 콜백이 호출되었어야 합니다!
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```
마지막으로, 때로는 모든 대기 중인 타이머를 지우는 것이 유용할 수 있습니다. 이를 위해 `jest.clearAllTimers()`를 사용할 수 있습니다.

## 선택적으로 모의하기
가끔 코드가 특정 API의 원래 구현을 덮어쓰지 않아야 할 수도 있습니다. 이 경우 `doNotFake` 옵션을 사용할 수 있습니다. 예를 들어 `jsdom` 환경에서 `performance.mark()`를 사용자 정의 모의 함수로 제공하는 방법은 다음과 같습니다:

```ts
/**
 * @jest-environment jsdom
 */

const mockPerformanceMark = jest.fn();
window.performance.mark = mockPerformanceMark;

test('performance.mark()를 모의할 수 있습니다.', () => {
  jest.useFakeTimers({doNotFake: ['performance']});

  expect(window.performance.mark).toBe(mockPerformanceMark);
});
```
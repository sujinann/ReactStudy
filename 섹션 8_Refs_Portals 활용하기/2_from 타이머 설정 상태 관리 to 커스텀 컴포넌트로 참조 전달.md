## **타이머 설정 & State 관리**

```jsx
// TimerChallendge.jsx

import { useState } from 'react';

export default function TimerChallenge({ title, targetTime }) {

  // 의도한 기능을 동작시키기 위한 로직 추가 ( 시작, 멈춤, 타이머 )
  const [timerStarted, setTimerStarted] = useState(false);
  const [timerExpired, setTimerExpired] = useState(false);

  function handleStart() {
    // 자바스크립트 내장 함수 setTimeout 이용, 단위 차이 고려
    setTimeout(() => {
      setTimerExpired(true);
    }, targetTime * 1000);

    setTimerStarted(true);
  }

  function handleStop() {
    // ref 이용해서 타이머 접근 할 것
  }

  return (
    <section className="challenge">
      <h2>{title}</h2>
      {timerExpired && <p>You lost!</p>}
      <p className="challenge-time">
        {targetTime} second{targetTime > 1 ? 's' : ''}
      </p>
      <p>
        <button onClick={handleStart}>
          {timerStarted ? 'Stop' : 'Start'} Challenge
        </button>
      </p>
      <p className={timerStarted ? 'active' : undefined}>
        {timerStarted ? 'Time is running...' : 'Timer inactive'}
      </p>
    </section>
  );
}
```

<br/>

## DOM요소 연결 외 ref 사용법

```jsx
// TimerChallendge.jsx

import { useState, useRef } from 'react';

// let timer;

export default function TimerChallenge({ title, targetTime }) {
  // ref 활용
  const timer = useRef();

  const [timerStarted, setTimerStarted] = useState(false);
  const [timerExpired, setTimerExpired] = useState(false);

  function handleStart() {
    timer.current = setTimeout(() => {
      setTimerExpired(true);
    }, targetTime * 1000);

    setTimerStarted(true);
  }

  function handleStop() {
    // 1. 함수 사용을 위해 input 값으로 포인터(id)가 필요함
    // 2. 따라서 변수를 추가하는 방법을 생각해 보았지만 오류가 생김
    // 3. ref 도입
    clearTimeout(timer.current);
  }

  return (
    <section className="challenge">
      <h2>{title}</h2>
      {timerExpired && <p>You lost!</p>}
      <p className="challenge-time">
        {targetTime} second{targetTime > 1 ? 's' : ''}
      </p>
      <p>
        <button onClick={timerStarted ? handleStop : handleStart}>
          {timerStarted ? 'Stop' : 'Start'} Challenge
        </button>
      </p>
      <p className={timerStarted ? 'active' : undefined}>
        {timerStarted ? 'Time is running...' : 'Timer inactive'}
      </p>
    </section>
  );
}
```

ref 사용 특징

- UI 에 직접적인 영향을 끼치지 않으면서도,
- 컴포넌트가 재실행 될 때 초기화 되지 않는 값을 제어 하고 싶을 때 ref 를 사용

<br/>

## 모달 컴포넌트 추가하기

```jsx
// ResultModal.jsx

export default function ResultModal({ result, targetTime }) {
  return (
    // 내장 요소 dialog 사용
    <dialog className="result-modal">
      <h2>You {result}</h2>
      <p>
        The target time was <strong>{targetTime} seconds.</strong>
      </p>
      <p>
        You stopped the timer with <strong>X seconds left.</strong>
      </p>
      // 닫는 기능
      <form method="dialog">
        <button>Close</button>
      </form>
    </dialog>
  );
}
```

dialog

- 내장 스타일과 내장 기능 제공, 오버레이로 나타내며 편하고 이쁨
- form method=”dialog” 안의 버튼으로 닫음
- dialog의 open 속성(prop)에 따라 나타나게 함(backdrop 포함).
    - 기본적으로는 보이지 않는 상태
- 나타내고 싶은 컴포넌트에 import하고 리턴 값에 fragment로 기존 요소들과 묶어 반환

<br/>

## 커스텀 컴포넌트로 ref 전달

```jsx
// ResultModal.jsx

import { forwardRef } from 'react';

const ResultModal = forwardRef(function ResultModal({ result, targetTime }, ref) {
  return (
    <dialog ref={ref} className="result-modal">
      <h2>You {result}</h2>
      <p>
        The target time was <strong>{targetTime} seconds.</strong>
      </p>
      <p>
        You stopped the timer with <strong>X seconds left.</strong>
      </p>
      <form method="dialog">
        <button>Close</button>
      </form>
    </dialog>
  );
})

export default ResultModal;
```

forwardRef 사용 방법

- 컴포넌트와 컴포넌트 사이에 ref 값을 전달하며 연관해서 사용하고 싶을 때 사용
- 함수 앞을 forwardRef로 감싸서 반환
- 2번째 prop을 ref로 받음
<br/>

```jsx
// TimerChallenge.jsx

// forwardRef 적용
// 이전 코드
{timerExpired && <ResultModal targetTime={targetTime} result="lost" />}

// 이후 코드
dialog.current.showModal();
<ResultModal ref={dialog} targetTime={targetTime} result="lost" />
```

- 내장 dialog 요소는 showModal method를 가지고 있음
- 이는 리액트의 기능이 아니고 표준 브라우저의 기능 중 하나임

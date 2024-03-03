# useImperativeHandle 훅으로 컴포넌트 API 노출시 ~ Portals(포탈) 소개 및 이해하기

### useImperativeHandle 훅으로 컴포넌트 API 노출시

- 더 큰 React 앱을 만들거나, 다른 사람과 작업하게 될 경우: dialog 태그를 (예를 들어 div로) 수정해버리면 모달의 showModal 호출이 제대로 작동하지 않게 될 수 있다는 문제.
    - 내부 dialog 요소에서 showModal을 호출하는 것은 비효율적.
    - ResultModal 컴포넌트가 자신의 함수를 노출시키도록 구축하자.
        - 컴포넌트 밖에서 ref로 호출할 수 있도록.
- useImperativeHandle 훅 이용: 호출될 함수를 노출시킨다. (속성, 메소드 등 정의 가능)
    - 일반적으로는 useImperativeHandle 훅보다는 props를 사용하는 쪽을 선호하는 게 맞다.
    - 훅을 사용한 컴포넌트를 더욱 견고하고 재사용 가능하게 만듦.
    - useImperativeHandle의 두 인자 필요.
        - forwardRef로부터 받은 ref
            - forwardRef와 함께 활용되어야 함.
        - 객체를 반환하는 함수
            - 바깥에 노출되어야 하는 모든 속성, 메소드 등을 묶은 객체.
            - 함수명은 본인이 임의로 정해도 무방.
    
    ```jsx
    # ResultModal.jsx
    
    import { forwardRef, useImperativeHandle } from "react"; // useImperativeHandle을 import
    
    const ResultModal = forwardRef(function ResultModal(
      { result, targetTime },
      ref, // useImperativeHandle의 인수로도 들어갈 ref
    ) {
      useImperativeHandle(ref, () => { // forwardRef에서 return하기 전에 useImperativeHandle 작성
        return { open() {} }; // 컴포넌트 바깥에서 호출할 함수를 객체 형태로 반환
      }); 
    ```
    
- useRef 훅 추가: dialog 요소를 분리하기 위함.
    - 새로운 ref 참조 값 추가. (여기서는 dialog)
    - open 함수가 호출될 때 dialog 값의 showModal 함수도 호출되도록 작성.
    
    ```jsx
    # ResultModal.jsx
    
    import { forwardRef, useImperativeHandle, useRef } from "react"; // useRef를 import
    
    const ResultModal = forwardRef(function ResultModal(
      { result, targetTime },
      ref,
    ) {
      const dialog = useRef(); // dialog라는 새로운 ref 참조 값을 생성
    
      useImperativeHandle(ref, () => {
        return {
          open() {
    	 dialog.current.showModal(); // dialog 값의 current 속성이 showModal을 호출하게 함
          },
        };
      });
    
      return (
        // ref={ref}를 ref={dialog}로 수정: dialog라는 ref 값을 가져옴
        <dialog ref={dialog} className="result-modal"> 
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
    });
    
    export default ResultModal;
    ```
    
- 사용되는 component에서 호출하는 함수명 변경.
    - showModal에서 open으로.
    - TimerChallenge에서 쓰이는 dialog 값에 ResultModal에서 useImperativeHandle이 반환한 객체를 담게 됨.
    
    ```jsx
    # TimerChallenge.jsx
    
      function handleStart() {
        timer.current = setTimeout(() => {
          setTimerExpired(true);
          dialog.current.open(); // showModal()을 open()으로 수정
        }, targetTime * 1000);
        setTimerStarted(true);
      }
    ```
    
- dialog 요소를 완벽히 분리하되, 실행 결과는 똑같음.
    - ResultModal 파일을 임의로 수정해도 문제 없어짐.

### 추가 예시: Refs(참조)와 State(상태)를 사용해야 하는 경우

- 성공했을 시 남은 시간을 계산해서 보여주는 모달창을 만들어보자.
    - setTimeout를 setInterval로 수정.
        - setInterval: 설정한 시간이 만료될 때마다 작성된 함수를 반복 실행.
        - 만료 시간을 targetTime * 1000에서 매우 짧은 시간으로 수정.
        - 실시간으로 남은 시간을 만료 시간마다 측정 가능하게 됨.
        
        ```jsx
        # TimerChallenge.jsx
        
        export default function TimerChallenge({ title, targetTime }) {
          const timer = useRef();
          const dialog = useRef();
        
          // const [timerStarted, setTimerStarted] = useState(false);
          // const [timerExpired, setTimerExpired] = useState(false);
          const [timeRemaining, setTimeRemaining] = useState(targetTime * 1000); // 밀리세컨드 단위
          const timerIsActive = timeRemaining > 0 && timeRemaining < targetTime * 1000; // 제한 시간 동안만 타이머가 활성화되어야
        
          function handleStart() {
            timer.current = setInterval(() => {
              // setTimerExpired(true);
              // dialog.current.open();
              setTimeRemaining((prevTimeRemaining) => prevTimeRemaining - 10);
            }, 10); // 밀리세컨드 단위
            // setTimerStarted(true);
          }
        ```
        
    - clearTimeout를 clearInterval로 수정.
        - 여기서도 변수나 상태가 아니라 참조(ref)가 사용됨 (timer.current는 그대로 유지).
        
        ```jsx
          function handleStop() {
            clearInterval(timer.current);
          }
        ```
        
    - 제한 시간이 모두 지나버렸을 때에도 타이머가 멈추도록 설정해보자.
        - if문 설정.
            - 무한 루프 방지 가능: 상태의 업데이트마다 무한 반복되는 것을 if 조건으로 막음.
            - 상태 업데이트 함수를 컴포넌트에서 바로 호출할 때 항상 무한 루프의 위험을 상기하자!
            
            ```jsx
            export default function TimerChallenge({ title, targetTime }) {
              const timer = useRef();
              const dialog = useRef();
            
              // const [timerStarted, setTimerStarted] = useState(false);
              // const [timerExpired, setTimerExpired] = useState(false);
              const [timeRemaining, setTimeRemaining] = useState(targetTime * 1000); // 밀리세컨드 단위
              const timerIsActive = timeRemaining > 0 && timeRemaining < targetTime * 1000;
            
              // if 조건문 추가
              if (timeRemaining <= 0) {
                clearInterval(timer.current);
                setTimeRemaining(targetTime * 1000); // 맨 처음의 목표 시간으로 초기화
              }
            
              function handleStart() {
                timer.current = setInterval(() => {
                  // setTimerExpired(true);
                  // dialog.current.open();
                  setTimeRemaining((prevTimeRemaining) => prevTimeRemaining - 10);
                }, 10); // 밀리세컨드 단위
                // setTimerStarted(true);
              }
            
              function handleStop() {
                clearInterval(timer.current);
              }
            ```
            
    - 삼항연산자 조건들을 timerStarted에서 timerIsActive 변수로 모두 수정.
    
    ```jsx
            <p>
              <button onClick={timerIsActive ? handleStop : handleStart}>
                {timerIsActive ? "Stop" : "Start"} Challenge
              </button>
            </p>
            <p className={timerIsActive ? "active" : undefined}>
              {timerIsActive ? "Time is running..." : "Timer inactive"}
            </p>
    ```
    
    - 모달창을 open하는 함수 호출 코드를 추가 작성하자.
        - dialog.current.open()을 각 상황 안에 추가 작성.
    
    ```jsx
      if (timeRemaining <= 0) {
        clearInterval(timer.current);
        setTimeRemaining(targetTime * 1000); // 맨 처음의 목표 시간으로 초기화
        dialog.current.open(); // 타이머 만료로 자동으로 패배할 때의 open
      }
    
      function handleStop() {
        dialog.current.open(); // 수동으로 승리할 때의 open
        clearInterval(timer.current);
      }
    ```
    
- 실행 결과:
    - 승리했을 때, 패배했을 때 모두 모달창이 정상적으로 열림.
    - 문구 업데이트가 아직 없어 둘 다 패배했다는 문구가 뜸.

### 컴포넌트 간의 State(상태) 공유

- 결과에 따른 정확한 문구를 모달창에 반영해보자.
    - TimerChallenge에서 ResultModal 컴포넌트에 넘겨줄 prop을 수정.
        - result 속성 삭제.
        - remainingTime 속성 추가.
        
        ```jsx
        # TimerChallenge.jsx
        
              {/* result 속성 삭제 후 remainingTime 속성 추가 */}
              <ResultModal
                ref={dialog}
                targetTime={targetTime}
                remainingTime={timeRemaining}
              />
        ```
        
    - ResultModal 컴포넌트에서도 구조 분해 할당으로 remainingTime 속성을 받는다.
    
    ```jsx
    # ResultModal.jsx
    
    const ResultModal = forwardRef(function ResultModal(
      // 필요없는 result 속성을 제거하고, remainingTime 속성을 추가로 받아온다.
      { targetTime, remainingTime },
      ref,
    ) {
      const dialog = useRef();
      const userLost = remainingTime <= 0; // true면 패배, false면 승리
    
    // 중략
    
    return (
        // ref={ref}를 ref={dialog}로 수정
        <dialog ref={dialog} className="result-modal">
          {/* userLost가 true일 때만 You lost가 뜨도록 */}
          {userLost && <h2>You lost</h2>}
    ```
    
    - X seconds left라는 문구에 남은 시간을 초 단위로 반영해보자.
        - remainingTime을 1000으로 나눈 값에 소수점을 잘라 표기한다.
        - 자바스크립트 내장 함수인 toFixed를 이용.
        
        ```jsx
        # ResultModal.jsx
        
          const dialog = useRef();
          const userLost = remainingTime <= 0; // true면 패배, false면 승리
          const formattedRemainingTime = (remainingTime / 1000).toFixed(2); // 소수점 둘째자리까지만
        
              <p>
                You stopped the timer with{" "}
                <strong>{formattedRemainingTime} seconds left.</strong>
                {/* 소수점을 정리한 남은 시간 값을 여기에서 반환 */}
              </p>
        ```
        
    - 실행 결과:
        - 패배했을 경우 You lost 문구가 뜨지 않고 목표 시간 seconds left라고 떠버린다.
        - Why?
            - TimerChallenge.jsx에서 조건에 걸리는 순간, 남은 시간을 초기화시켜버린다.
            - timeRemaining 값이 음수가 될 수가 없게 되어버림.
        - Then?
            - 조건문 안에서 시간을 초기화시키지 않고, 새로운 함수를 작성해야 한다!
            
            ```jsx
            # TimerChallenge.jsx
            
              if (timeRemaining <= 0) {
                clearInterval(timer.current);
                setTimeRemaining(targetTime * 1000); // 맨 처음의 목표 시간으로 초기화
                dialog.current.open(); // 타이머 만료로 자동으로 패배할 때의 open
              }
            ```
            
    - 새로운 함수 작성 후 prop으로 넘겨주자.
        - handleReset 값을 넘겨주는 onReset 속성을 추가해 ResultModal로 보낸다.
        
        ```jsx
        # TimerChallenge.jsx
        
          if (timeRemaining <= 0) {
            clearInterval(timer.current);
            // setTimeRemaining(targetTime * 1000); // 맨 처음의 목표 시간으로 초기화 => 삭제
            dialog.current.open(); // 타이머 만료로 자동으로 패배할 때의 open
          }
        
          // 새 함수 추가 작성
          function handleReset() {
            setTimeRemaining(targetTime * 1000);
          }
        
        // 중략
        
          return (
            <>
              {/* result 속성 삭제 후 remainingTime 속성 추가 */}
              <ResultModal
                ref={dialog}
                targetTime={targetTime}
                remainingTime={timeRemaining}
                onReset={handleReset}
              />
              {/* handleReset 값을 넘겨주는 onReset 속성 추가 */}
        ```
        
        - ResultModal에서도 구조 분해 할당으로 onReset이라는 prop을 받아온다.
            - 버튼을 누를 시 form이 제출되는데, 제출되고 나면 onSubmit가 활성화됨.
                - onSubmit은 리액트에 내장되어 있는 form 요소의 기본 기능.
            - onReset 값을 onSubmit 기능에 넣어준다.
            
            ```jsx
            # ResultModal.jsx
            
            const ResultModal = forwardRef(function ResultModal(
              // remainingTime 속성을 받아온다.
              { targetTime, remainingTime, onReset },
              ref,
            ) {
            
            // 중략
            
                  <form method="dialog" onSubmit={onReset}>
                    <button>Close</button>
                  </form>
            ```
            
    - 실행 결과:
        - 정상적으로 패배 시 You lost 문구와 0.00 seconds left로 뜸.
        - 아직 추가 작성하지 않아 승리 시 문구 및 점수는 따로 없는 상태.

### 데모 앱의 “결과 모달창” 개선

- 점수와 성공 메세지를 모달창에 추가해보자.
    - 점수를 계산할 score 변수를 생성.
    - userLost가 false일 때 (= 승리 시) 점수가 뜨도록 문구를 설정.
    
    ```jsx
    # ResultModal.jsx
    
      // 주의: remainingTime은 밀리초 단위, targetTime은 초 단위 => 목표 시간에 가까울 수록 높은 점수!
      const score = Math.round((1 - remainingTime / (targetTime * 1000)) * 100); // 0 ~ 100 까지의 점수
    
    // 중략
    
          {/* userLost가 true일 때만 You lost가 뜨도록 */}
          {userLost && <h2>You lost</h2>}
          {/* userLost가 false일 때는 점수가 뜨도록 */}
          {!userLost && <h2>Your Score: {score}</h2>}
    ```
    

### 모달을 ESC(Escape) 키로 닫기

- ESC 버튼으로도 모달창을 닫을 수는 있지만, onReset 함수가 트리거되지는 않는다.
    - dialog 요소에 내장된 onClose 속성을 추가.
    - onClose 속성에 onReset 값을 넣어준다.
    
    ```jsx
    # ResultModal.jsx
    
        <dialog ref={dialog} className="result-modal" onClose={onReset}>
    ```
    

### Portals(포탈) 소개 및 이해하기

- 기능 작동에는 문제가 없지만, 기술적으로 보면 코드 구조는 아래와 같은 게 좋다.
    - 페이지 맨 위에 보여지는 오버레이 요소(dialog 등): body 밑에 위치. (더 높은 층위에 위치)
    - 더 직관적으로 보여지고, 접근성이 좋아지며, 만약의 형식 문제도 회피 가능.
    - Portal(포탈)로 해결할 수 있다!
- react-dom 라이브러리로부터 createPortal 함수를 가져오자.
    - react 라이브러리와 react-dom 라이브러리의 차이점:
        - react 라이브러리: 모든 환경에서 작동 가능한 함수 및 기능만 노출.
        - react-dom 라이브러리: 리액트가 DOM(브라우저에 렌더링된 웹사이트)와 상호작용.
        
        ```jsx
        # ResultModal.jsx
        
        import { createPortal } from "react-dom";
        ```
        
    - 포탈의 의의:
        - 렌더링될 HTML 코드를 DOM 내의 다른 곳으로 옮겨준다.
- return에 작성된 코드를 createPortal로 래핑(wrapping)해준다.
    - createPortal은 2개의 인수를 받는다.
        - 포탈로 이동시킬 JSX 코드.
        - 위의 코드가 위치하게 될 HTML 요소.
    - index.html 파일의 modal이라는 id를 가진 div 위치로 JSX 코드를 teleport시킨다.
        - 기본 브라우저 API인 document.getElementById를 활용한다.
    
    ```jsx
    # ResultModal.jsx
    
      // 반환 코드를 createPortal 함수로 래핑한다
      return createPortal(
        // ref={ref}를 ref={dialog}로 수정
        <dialog ref={dialog} className="result-modal" onClose={onReset}>
          {/* userLost가 true일 때만 You lost가 뜨도록 */}
          {userLost && <h2>You lost</h2>}
          {/* userLost가 false일 때는 점수가 뜨도록 */}
          {!userLost && <h2>Your Score: {score}</h2>}
          <p>
            The target time was <strong>{targetTime} seconds.</strong>
          </p>
          <p>
            You stopped the timer with{" "}
            <strong>{formattedRemainingTime} seconds left.</strong>
            {/* 소수점을 정리한 남은 시간 값을 여기에서 반환 */}
          </p>
          <form method="dialog" onSubmit={onReset}>
            <button>Close</button>
          </form>
        </dialog>,
        document.getElementById("modal"),
      );
      // modal이라는 id를 가진 HTML 요소에 teleport시킨다.
    });
    ```
    
    ```jsx
    # index.html
    
      <body>
        <!-- 상대적으로 body에 가까움 -->
        <div id="modal"></div>
        <div id="content">
    ```
    
- 실행 결과:
    - 기능은 당연히 정상!
    - 개발자 도구에서 body에 더 가깝게 위치하는 것을 볼 수 있음.
        - content 안에 있던 코드가 modal 안으로 이동함.

![modal](https://github.com/serethia/serethia/assets/137035446/0472900d-538c-4bd4-894f-4194403daef6)

# 모듈 소개 ~ 리액트의 가상 DOM 사용 - 직접 살펴보기

### 모듈 소개

- 학습 목표:
    - 리액트가 어떻게 DOM을 업데이트하는지
    - 어떻게 불필요한 업데이트를 회피하는지
    - key에 대해
    - state 예정(scheduling) 및 묶기(batching)

### 리액트의 컴포넌트 트리 생성 / 리액트가 시스템 뒷편에서 동작하는 방식

- 앱 컴포넌트 함수로 시작
    - App.jsx가 main.jsx에 유일하게 작성되어 있기 때문
    
    ```jsx
    // main.jsx
    
    ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    ```
    
- 컴포넌트
    - 내장 컴포넌트: 소문자로 시작
    - 커스텀 컴포넌트: 대문자로 시작
- 컴포넌트 트리 생성
    - App 컴포넌트에서 시작
    - 자식 커스텀 컴포넌트로 내려감 (트리 구조 형성)
    - 컴포넌트 작동 순서를 알아보기 위해 콘솔에 기록 가능
    
    ```jsx
    // App.jsx
    
      log('<App /> rendered');
    ```
    
    ![콘솔 캡처](https://github.com/serethia/serethia/assets/137035446/0337975a-ab4b-46fa-9f33-7fd795448294)
    

### 리액트 DevTools Profiler로 컴포넌트 함수 실행 분석하기

- 컴포넌트 함수 실행 확인 방법 2가지:
    - 아까처럼 콘솔에 log 출력하거나
    - React Developer Tools를 활용한다
        
        ![react dev tools 캡처](https://github.com/serethia/serethia/assets/137035446/79de4a07-e291-40a8-a59c-076b341965f2)
        
        - 개발자 도구에서 profiler 탭으로 이동
            - 업데이트/렌더링 중인 컴포넌트들 확인 가능
        - 파란 원형 버튼: 프로파일링 (상호작용을 전부 record해 그래픽 데이터로 실행된 컴포넌트 확인)
            - 1번 누르면 빨간색으로 변함 (record 중)
            - 2번 누르면 파란색으로 돌아옴 (record 끝)
        - 불 모양 버튼: flame graph chart 모드 (실행된 컴포넌트 함수 순서 및 관계도 포함)
            
            ![flame chart 모드 캡처](https://github.com/serethia/serethia/assets/137035446/f34a5b4f-17e1-4364-bb07-7926285ff463)
            
            - 컴포넌트에 hover 시 렌더링 화면에 하이라이트되어 표시됨
            - 컴포넌트 재실행 시:
                - 부모 컴포넌트: 영향 X
                - 자식 컴포넌트: 함께 재실행
        - 목록 모양 버튼: ranked chart 모드 (재실행된 컴포넌트들만 확인)
            
            ![ranked chart 모드 캡처](https://github.com/serethia/serethia/assets/137035446/76a4d8bf-7a13-4cb4-9fbf-3df8a54a9d15)
            
        - 톱니 모양 버튼: 설정
            
            ![설정 캡처](https://github.com/serethia/serethia/assets/137035446/79fe9c99-5298-4137-9279-8c70d104d43d)
            
            - profiling 중에 왜 각 컴포넌트가 렌더링되는지 기록 설정 가능
            
            ![설정 후 캡처](https://github.com/serethia/serethia/assets/137035446/758c3d1b-2e55-4c05-9131-c0fb2849ac02)
            

### memo()로 컴포넌트 함수 실행 방지

- memo 함수 활용
    
    ```jsx
    // Counter.jsx
    
    import { useState, memo } from 'react';
    
    // 중략
    
    const Counter = memo(function Counter({ initialCount }) {
    
    // 중략
    
    });
    
    export default Counter;
    ```
    
    - memo를 import
    - memo로 컴포넌트 함수를 wrap
    - 같은 함수명의 변수/상수로 선언
    - 해당 변수/상수를 export
- memo 함수 역할:
    - 이전 prop 값과 새 prop 값을 비교
    - 완전히 동일하다면: memo 내의 함수 재실행이 저지됨 (부모 컴포넌트의 재실행과 관계없이)
        - 그 안에 포함된 자식 컴포넌트들도 함께 재실행되지 않음
    - 다르다면: memo에 아무 영향이 없어 달라진 값에 따라 재실행됨
    - 콘솔 실행 시 Header 컴포넌트만 추가 실행됨
    
    ![콘솔 캡처 2](https://github.com/serethia/serethia/assets/137035446/24cce22c-c69d-4cd2-bac4-81d26eae818f)
    
- memo 함수 남용은 금물!
    - 최대한 상위 트리 컴포넌트에서 사용
    - 모든 컴포넌트에 memo를 활용하면 리액트가 모든 prop을 체크해야 해서 부담이 커짐
    - prop 값이 자주 바뀌는 컴포넌트에서는 사용하지 말 것

### 컴포넌트 함수 실행 방지를 위한 구조

- memo보다 더 강력한 방법: 현명한 컴포넌트 구조 활용
    
    ```jsx
    // ConfigureCounter.jsx
    
    import { useState } from 'react';
    
    export default function ConfigureCounter({ onSet }) {
        const [enteredNumber, setEnteredNumber] = useState(0);
    
        function handleChange(event) {
            setEnteredNumber(+event.target.value);
        }
    
        function handleSetClick() {
            onSet(enteredNumber);
            setEnteredNumber(0);
        }
    
        return (
            <section id="configure-counter">
                <h2>Set Counter</h2>
                <input type="number" onChange={handleChange} value={enteredNumber} />
                <button onClick={handleSetClick}>Set</button>
            </section>
        );
    }
    
    ```
    
    ```jsx
    // App.jsx
    
    import { useState } from 'react';
    
    import Counter from './components/Counter/Counter.jsx';
    import Header from './components/Header.jsx';
    import { log } from './log.js';
    import ConfigureCounter from './components/Counter/ConfigureCounter.jsx';
    
    function App() {
        log('<App /> rendered');
    
        const [chosenCount, setChosenCount] = useState(0);
    
        function handleSetCount(newCount) {
            setChosenCount(newCount);
        }
    
        return (
            <>
                <Header />
                <main>
                    <ConfigureCounter onSet={handleSetCount} />
                    <Counter initialCount={chosenCount} />
                </main>
            </>
        );
    }
    
    export default App;
    
    ```
    
    - ConfigureCounter.jsx 파일 생성
    - App.jsx의 section 태그 및 관련 부분 코드를 잘라 ConfigureCounter.jsx로 옮겨 반환하기
    - App.jsx에 handleSetCount 함수 추가 작성
    - ConfigureCounter 컴포넌트를 import하고 불러온 뒤, 보내줄 prop을 작성
    - ConfigureCounter.jsx에서 useState를 import한 후, handleSetClick 함수 내의 setChosenCount 함수를 onSet으로 바꿔줌
    - 콘솔 실행 시: ConfigureCounter만 재실행됨
    
    ![콘솔 캡처 3](https://github.com/serethia/serethia/assets/137035446/faa8120a-6a7c-439a-ac95-82e0406d3e7f)
    
- memo 함수 삭제해도 무방 (그다지 많은 counter 함수 실행을 저지해주지 않기 때문, 오히려 성능에 부담만 주는 꼴)

### useCallback() 훅 이해하기

- 버튼과 아이콘은 불필요한 재실행이 되는 중
    
    ![콘솔 캡처 4](https://github.com/serethia/serethia/assets/137035446/b4069638-ab2b-41f0-a907-30c11ce06430)
    
    - memo로 IconButton 컴포넌트를 wrap해보자
        
        ```jsx
        // IconButton.jsx
        
        import { memo } from 'react';
        import { log } from '../../log.js';
        
        const IconButton = memo(function IconButton({ children, icon, ...props }) {
            log('<IconButton /> rendered', 2);
        
            const Icon = icon;
            return (
                <button {...props} className="button">
                    <Icon className="button-icon" />
                    <span className="button-text">{children}</span>
                </button>
            );
        });
        
        export default IconButton;
        
        ```
        
        - 실행:
            - 여전히 버튼과 아이콘이 실행됨
            - memo가 정상 작동하지 않고 있음 / 작동했지만 prop 값이 변함
    - 원인:
        
        ```jsx
        # Counter.jsx
        
        // 중략
        
        const Counter = memo(function Counter({ initialCount }) {
            log('<Counter /> rendered', 1);
            const initialCountIsPrime = isPrime(initialCount);
        
            const [counter, setCounter] = useState(initialCount);
        
            function handleDecrement() {
                setCounter((prevCounter) => prevCounter - 1);
            }
        
            function handleIncrement() {
                setCounter((prevCounter) => prevCounter + 1);
            }
            
            // 중략
            
                    <p>
                        <IconButton icon={MinusIcon} onClick={handleDecrement}>
                            Decrement
                        </IconButton>
                        <CounterOutput value={counter} />
                        <IconButton icon={PlusIcon} onClick={handleIncrement}>
                            Increment
                        </IconButton>
                    </p>
        ```
        
        - onClick으로 실행되는 handleDecrement, handleIncrement 함수들이 변했기 때문
        - Counter.jsx 안에 있는 객체로서 함께 재실행되었기 때문
    - 해결법:
        
        ```jsx
        // Counter.jsx
        
        import { useState, memo, useCallback } from 'react';
        
        // 중략
        
            const handleDecrement = useCallback(function handleDecrement() {
                setCounter((prevCounter) => prevCounter - 1);
            }, []);
        
            const handleIncrement = useCallback(function handleIncrement() {
                setCounter((prevCounter) => prevCounter + 1);
            }, []);
            
        // 후략
        ```
        
        - useCallback 훅으로 handle 함수들을 wrap한 후 같은 함수명의 변수/상수로 선언
        - 의존성 배열을 빈 상태로 둠
    - 실행:
        - 콘솔에 버튼과 아이콘들이 재실행되지 않고 필요한 컴포넌트들만 정상적으로 재실행됨
        
        ![콘솔 캡처 5](https://github.com/serethia/serethia/assets/137035446/2a5b0444-7d4f-49d4-8507-0fdf396bd58b)
        

### useMemo() 훅 이해하기

- useMemo 훅의 역할
    - 컴포넌트 함수 내의 일반 함수들을 wrap해 실행을 방지함
    - 복잡한 계산이 있을 경우에만 사용할 것
- useMemo 활용법
    
    ```jsx
    // Counter.jsx
    
    import { useState, memo, useCallback, useMemo } from 'react';
    
    // 중략
    
        const initialCountIsPrime = useMemo(() => isPrime(initialCount), [initialCount]);
        
    // 후략
    ```
    
    - useMemo를 import
    - 재실행되고 있는 isPrime 함수를 useMemo로 wrap한 뒤, 화살표 함수로 작성
    - 의존성 배열에 isPrime의 입력값인 initialCount를 넣어줌
- 실행:
    - 콘솔에 더 이상 isPrime이 재실행되지 않음
    
    ![콘솔 캡처 6](https://github.com/serethia/serethia/assets/137035446/b1d11411-b75d-4b3e-b6d4-5855cdc8c738)
    
    - 새로운 set counter 값을 작성하면 그에 따른 결과 값(의존성 배열에 넣은 값)이 출력됨
    
    ![콘솔 캡처 7](https://github.com/serethia/serethia/assets/137035446/c597be2d-6789-468c-94c4-5fa22a97d7fe)
    
- useMemo 훅 남용은 금물!
    - 코드 실행에 시간이 걸리거나, 불필요한 재실행을 막기 위해서는 useMemo를 활용하는 편이 좋음
    

### 리액트의 가상 DOM 사용 - 직접 살펴보기

- 리액트는 id가 root인 div 태그 안에 완전한 DOM 트리를 자동으로 삽입한다
- 버튼을 클릭해도 전체 DOM 이 모두 재삽입되는 게 아님 (관련 태그만 업데이트됨)
- 이유:
    - 리액트가 가상 DOM (virtual DOM) 스냅샷들을 비교해 실제 DOM에서 업데이트되어야 하는 부분들을 찾아내기 때문
    - 가상 DOM을 활용하는 것이 실제 DOM보다 더 빠르기 때문
- 작동 순서:
    - 컴포넌트 트리 형성
    - 렌더링되어야 하는 실제 HTML 코드의 가상 스냅샷을 형성
    - 생성되었던 마지막 가상 DOM 스냅샷과 새로 생성한 가상 DOM 스냅샷을 비교
    - 실제 DOM에 변동 사항들을 모두 적용

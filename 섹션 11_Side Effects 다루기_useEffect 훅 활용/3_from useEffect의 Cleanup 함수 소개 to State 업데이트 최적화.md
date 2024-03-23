# useEffect의 Cleanup 함수 소개 ~ State(상태) 업데이트 최적화

### useEffect의 Cleanup 함수 소개

- 연습 목표: 모달 창에서 취소 버튼을 눌러도 타이머가 재 실행되어 3초 뒤에 자동으로 삭제되어 버리는 문제를 해결해보자.
- 실습
    - useEffect로 타이머 문제 해결 가능: 컴포넌트가 사라질 때 타이머를 멈춤
    
    ```jsx
    # DeleteConfirmation.jsx
    
    import { useEffect } from 'react'; // (1) useEffect를 import
    
    export default function DeleteConfirmation({ onConfirm, onCancel }) {
        useEffect(() => {
        
            // (7) 실행: 취소 버튼을 눌러도 타이머가 멈춰서 3초 뒤에 이미지가 사라지지 않게 됨
            console.log('TIMER SET');
            
            // (2) useEffect 작성 후 그 안에 setTimeout 함수와 빈 의존성 배열을 집어넣음
            // (4) timer라는 상수에 setTimeout 함수가 반환하는 참조를 저장
            const timer = setTimeout(() => {
                onConfirm();
            }, 3000);
    
            // (3) cleanUp 함수를 return해서 타이머를 초기화
            // (6) cleanUp 함수는 위의 Effect 함수가 재작동할 때 실행
            // (6-2) 그러나 의존성 배열이 빈 배열이면 Effect 함수 재작동 불가능
            // (6-3) 의존성 배열에 onConfirm을 "일부러" 넣지 않았음
            return () => {
            
                // (7-2) 실행: 타이머 멈출 때마다 정상적으로 콘솔에 출력됨 (Effect 함수보다 이후에 실행됨)
                console.log('Cleaning up timer');
                
                // (5) clearTimeout에 timer 상수를 전달
                clearTimeout(timer);
            };
        }, []);
    
        return (
            <div id="delete-confirmation">
                <h2>Are you sure?</h2>
                <p>Do you really want to remove this place?</p>
                <div id="confirmation-actions">
                    <button onClick={onCancel} className="button-text">
                        No
                    </button>
                    <button onClick={onConfirm} className="button">
                        Yes
                    </button>
                </div>
            </div>
        );
    }
    
    ```
    
    - 콘솔 출력 결과: 타이머가 정상적으로 멈춰 자동 삭제 문제 해결됨
    
    ```jsx
    DeleteConfirmation.jsx:6 TIMER SET
    DeleteConfirmation.jsx:19 Cleaning up timer
    DeleteConfirmation.jsx:6 TIMER SET
    DeleteConfirmation.jsx:19 Cleaning up timer
    DeleteConfirmation.jsx:6 TIMER SET
    DeleteConfirmation.jsx:19 Cleaning up timer
    ```
    

### 객체의 문제점 & 함수 의존성

- 연습 목표: 함수를 의존성 배열에 넣을 때 발생할 수 있는 무한루프 문제에 대해 알아보자.
- 실습
    - Javscript에서 함수는 객체이기 때문에 의존성 배열에 넣을 경우 각별히 주의가 필요
    
    ```jsx
    # DeleteConfirmation.jsx
    
    import { useEffect } from 'react';
    
    export default function DeleteConfirmation({ onConfirm, onCancel }) {
        // (3-1) 그러나 이 함수는 무한루프되지 않는다 (App.jsx 참고)
        useEffect(() => {
            console.log('TIMER SET');
            const timer = setTimeout(() => {
                onConfirm();
            }, 3000);
            return () => {
                clearTimeout(timer);
            };
        }, [onConfirm]); // (1-1) 의존성 배열에 onConfirm(= App.jsx의 handleRemovePlace 함수) 추가
        // (1-2) 다수의 의존성은 쉼표로 구분해줄 것
    
        // (2-1) BUT 의존성으로 함수(=객체)를 추가하면: 무한 루프 위험 가능성!
        // (2-2) 함수 안에 중첩된 함수는 컴포넌트 실행마다 재생성됨
        // (2-3) Javascript에서 같은 모양의 코드여도 다른 객체로 생성되어버림
    
        return (
            <div id="delete-confirmation">
                <h2>Are you sure?</h2>
                <p>Do you really want to remove this place?</p>
                <div id="confirmation-actions">
                    <button onClick={onCancel} className="button-text">
                        No
                    </button>
                    <button onClick={onConfirm} className="button">
                        Yes
                    </button>
                </div>
            </div>
        );
    }
    
    ```
    
    ```jsx
    # App.jsx
    
    import { useRef, useState, useEffect } from 'react';
    
    import Places from './components/Places.jsx';
    import { AVAILABLE_PLACES } from './data.js';
    import Modal from './components/Modal.jsx';
    import DeleteConfirmation from './components/DeleteConfirmation.jsx';
    import logoImg from './assets/logo.png';
    import { sortPlacesByDistance } from './loc.js';
    
    const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
    const storedPlaces = storedIds.map((id) => AVAILABLE_PLACES.find((place) => place.id === id));
    
    function App() {
        const selectedPlace = useRef();
        const [pickedPlaces, setPickedPlaces] = useState(storedPlaces);
        const [availablePlaces, setAvailablePlaces] = useState([]);
        const [modalIsOpen, setModalIsOpen] = useState(false);
    
    		// (중략)
    
        function handleRemovePlace() {
            setPickedPlaces((prevPickedPlaces) => prevPickedPlaces.filter((place) => place.id !== selectedPlace.current));
            setModalIsOpen(false); // (3-2) 이 함수의 작동으로 Modal 컴포넌트가 children을 삭제하게 함 (Modal.jsx 참고)
            // (3-5) 위 코드를 주석처리하면 무한루프
    
            const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
            localStorage.setItem('selectedPlaces', JSON.stringify(storedIds.filter((id) => id !== selectedPlace.current)));
        }
    
        return (
            <>
                <Modal open={modalIsOpen} onClose={handleStopRemovePlace}>
                    <DeleteConfirmation onCancel={handleStopRemovePlace} onConfirm={handleRemovePlace} />
                </Modal>
    
                <header>
                    <img src={logoImg} alt="Stylized globe" />
                    <h1>PlacePicker</h1>
                    <p>Create your personal collection of places you would like to visit or you have visited.</p>
                </header>
                <main>
                    <Places
                        title="I'd like to visit ..."
                        fallbackText={'Select the places you would like to visit below.'}
                        places={pickedPlaces}
                        onSelectPlace={handleStartRemovePlace}
                    />
                    <Places
                        title="Available Places"
                        places={availablePlaces}
                        fallbackText="Sorting places by distance..."
                        onSelectPlace={handleSelectPlace}
                    />
                </main>
            </>
        );
    }
    
    export default App;
    
    ```
    
    ```jsx
    # Modal.jsx
    
    import { useRef, useEffect } from 'react';
    import { createPortal } from 'react-dom';
    
    function Modal({ open, children, onClose }) {
        const dialog = useRef();
    
        useEffect(() => {
            if (open) {
                dialog.current.showModal();
            } else {
                dialog.current.close();
            }
        }, [open]);
    
        return createPortal(
            <dialog className="modal" ref={dialog} onClose={onClose}>
                {/* (3-3) open이 false가 되어버리기 때문에 children이 삭제됨 */}
                {/* (3-4) 자식인 DeleteConfirmation 컴포넌트가 사라져서 무한루프 일어나지 않음! */}
                {open ? children : null}
            </dialog>,
            document.getElementById('modal')
        );
    }
    
    export default Modal;
    
    ```
    

### useCallback 훅

- 연습 목표: 위의 문제를 해결하기 위해 쓰는 훅에 대해 배워보자.
- 실습
    - useCallback 훅을 활용
    
    ```jsx
    # App.jsx
    
    import { useRef, useState, useEffect, useCallback } from 'react'; // (1-1) useCallback을 import
    
        // (2-1) useCallback은 wrap한 함수값을 반환
        // (2-2) wrap한 안쪽의 함수를 재생성되지 않게 하며 메모리에 저장
        // (2-3) 해당 함수가 재실행될 때마다 메모리로 저장된 함수를 재사용
        const handleRemovePlace = useCallback(function handleRemovePlace() {
            setPickedPlaces((prevPickedPlaces) => prevPickedPlaces.filter((place) => place.id !== selectedPlace.current));
            setModalIsOpen(false);
            // (3) 무한루프를 발생시키도록 주석 처리를 해봤는데도 useCallback 덕분에 무한루프되지 않음
    
            // (5-1) 주석을 해제해 모달이 정상적으로 닫히게 돌려놓음
            // (5-2) useCallback은 이에 추가적인 안정성을 부여해주는 역할을 함
    
            const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
            localStorage.setItem('selectedPlaces', JSON.stringify(storedIds.filter((id) => id !== selectedPlace.current)));
        }, []);
        // (1-2) useCallback의 첫 번째 인수로 해당 함수를 wrap해 집어넣는다
        // (1-3) 두 번째 인수로 dependencies 배열을 집어넣는다
    
        // (4-1) 의존성 배열에는 wrap한 함수의 prop 혹은 state 값들 (혹은 그 영향을 받는 다른 값들) 을 집어넣는다
        // (4-2) 빈 배열로 놔두면 useEffect처럼 useCallback 내의 함수가 변하지 않아 재생성도 되지 않는다
    ```
    

### useEffect의 Cleanup 함수: 다른 예시

- 연습 목표: 사용자에게 3초 뒤 모달창이 자동으로 닫힌다고 알려주는 진행 표시줄을 추가해보자.
- 실습
    - clearInterval 함수를 활용해 진행 표시줄의 interval을 멈출 수 있음
    
    ```jsx
    # DeleteConfirmation.jsx
    
    import { useEffect, useState } from 'react'; // (2-1) progress에 활용할 상태 작성용 useState를 import
    
    const TIMER = 3000; // (3) 여러 곳에서 작성된 3초를 상수에 저장해 밖에 빼두고 전부 TIMER 변수로 수정해준다
    
    export default function DeleteConfirmation({ onConfirm, onCancel }) {
        const [remainingTime, setRemainingTime] = useState(TIMER); // (2-2) progress용 상태 작성
    
        // (4-1) 매 10ms마다 실행되는 setInterval 함수 추가 작성
        useEffect(() => {
            // (8-1) interval이라는 상수에 저장
            const interval = setInterval(() => {
                console.log('INTERVAL'); // (6-3) 무한루프 여부 확인용 콘솔 출력
                // (7-1) 실행하면 10ms마다 제대로 INTERVAL이 출력되지만, 타이머 만료 후에도 출력이 계속됨
                // (7-2) interval을 멈출 clearInterval을 작성해줘야 함
                setRemainingTime((prevTime) => prevTime - 10); // (4-2) 10ms마다 10ms씩 감소하도록
            }, 10);
            // (6-1) BUT 그냥 실행하면 무한루프가 발생해 만료가 되어버린 상태로 나타남
            // (6-2) useEffect를 추가 작성해 wrap해줘야 함
    
            // (8-2) 아래의 clearTimeout 때처럼 clearInterval 함수를 반환
            return () => {
                clearInterval(interval);
            };
        }, []);
        // (8-3) 의존성 배열이 비어있으므로 모달창이 닫힐 때 clearInterval 함수 실행됨
    
        useEffect(() => {
            console.log('TIMER SET');
            const timer = setTimeout(() => {
                onConfirm();
            }, TIMER);
            return () => {
                clearTimeout(timer);
            };
        }, [onConfirm]);
    
        return (
            <div id="delete-confirmation">
                <h2>Are you sure?</h2>
                <p>Do you really want to remove this place?</p>
                <div id="confirmation-actions">
                    <button onClick={onCancel} className="button-text">
                        No
                    </button>
                    <button onClick={onConfirm} className="button">
                        Yes
                    </button>
                </div>
                {/* (1) 진행 표시줄을 생성하기 위한 progress 추가 */}
                <progress value={remainingTime} max={TIMER} />
                {/* (5-1) progress의 value 속성에 remainingTime 값을 넣어줌 */}
                {/* (5-2) 진행 표시줄의 찬 정도인 max 속성을 TIMER (3초) 로 설정 */}
            </div>
        );
    }
    
    ```
    
    - 콘솔 출력 결과: 3초 후 모달창이 닫힐 때 정상적으로 interval도 멈춰 INTERVAL 출력도 멈춤
    
    ```jsx
    DeleteConfirmation.jsx:28 TIMER SET
    300 DeleteConfirmation.jsx:12 INTERVAL
    ```
    

### State(상태) 업데이트 최적화

- 연습 목표: 리액트가 10ms마다 의존성 배열 안의 onConfirm 값을 확인하고 Effect 함수 재실행 여부를 알아보는 작업을 더 최적화해보자.
- 실습
    - ProgressBar 컴포넌트를 새로 추가 작성해 최적화
    
    ```jsx
    # ProgressBar.jsx
    
    import { useEffect, useState } from 'react';
    
    // (1-1) 하나의 컴포넌트만 재실행되도록 ProgressBar 파일을 새로 생성
    export default function ProgressBar({ timer }) {
        // (1-2) DeleteConfirmation에서 관련 부분만 잘라온다
        // (1-3) TIMER는 밖에서 timer라는 prop으로 받아온다
        const [remainingTime, setRemainingTime] = useState(timer);
        useEffect(() => {
            const interval = setInterval(() => {
                console.log('INTERVAL');
                setRemainingTime((prevTime) => prevTime - 10);
            }, 10);
    
            return () => {
                clearInterval(interval);
            };
        }, []);
    
        return <progress value={remainingTime} max={timer} />;
    }
    
    ```
    
    ```jsx
    # DeleteConfirmation.jsx
    
    import { useEffect } from 'react';
    import ProgressBar from './ProgressBar.jsx';
    
    const TIMER = 3000;
    
    export default function DeleteConfirmation({ onConfirm, onCancel }) {
        useEffect(() => {
            console.log('TIMER SET');
            const timer = setTimeout(() => {
                onConfirm();
            }, TIMER);
            return () => {
                clearTimeout(timer);
            };
        }, [onConfirm]);
    
        return (
            <div id="delete-confirmation">
                <h2>Are you sure?</h2>
                <p>Do you really want to remove this place?</p>
                <div id="confirmation-actions">
                    <button onClick={onCancel} className="button-text">
                        No
                    </button>
                    <button onClick={onConfirm} className="button">
                        Yes
                    </button>
                </div>
                {/* (2) ProgressBar 컴포넌트로 수정 작성 */}
                <ProgressBar timer={TIMER} />
            </div>
        );
    }
    
    ```

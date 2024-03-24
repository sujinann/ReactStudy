## useEffect가 필요 없는 경우

선택된 장소들을 클릭 시 선택 장소 정보에서 제거하는 기능과

앱이 시작할 때 localStorage에 저장된 장소들을 처음부터 보여주는 기능을 구현할 것임

```jsx
import { useRef, useState, useEffect } from 'react';

import Places from './components/Places.jsx';
import { AVAILABLE_PLACES } from './data.js';
import Modal from './components/Modal.jsx';
import DeleteConfirmation from './components/DeleteConfirmation.jsx';
import logoImg from './assets/logo.png';
import { sortPlacesByDistance } from './loc.js';

// 이 부분은 useEffect() 사용할 필요 없는 부분
// 콜백함수나 promise 등이 없고, 즉시 한줄씩 실행되기 때문
// 여러번 반복 실행할 필요도 없으니 성능 향상을 위해 앱 밖에 빼버리는 것
const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
const storedPlaces = storedIds.map((id) =>
  AVAILABLE_PLACES.find((place) => place.id === id)
);

function App() {
  const modal = useRef();
  const selectedPlace = useRef();
  const [availablePlaces, setAvailablePlaces] = useState([]);
  const [pickedPlaces, setPickedPlaces] = useState(storedPlaces);

  useEffect(() => {
    navigator.geolocation.getCurrentPosition((position) => {
      const sortedPlaces = sortPlacesByDistance(
        AVAILABLE_PLACES,
        position.coords.latitude,
        position.coords.longitude
      );

      setAvailablePlaces(sortedPlaces);
    });
  }, []);

  function handleStartRemovePlace(id) {
    modal.current.open();
    selectedPlace.current = id;
  }

  function handleStopRemovePlace() {
    modal.current.close();
  }

  function handleSelectPlace(id) {
    setPickedPlaces((prevPickedPlaces) => {
      if (prevPickedPlaces.some((place) => place.id === id)) {
        return prevPickedPlaces;
      }
      const place = AVAILABLE_PLACES.find((place) => place.id === id);
      return [place, ...prevPickedPlaces];
    });

    const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
    if (storedIds.indexOf(id) === -1) {
      localStorage.setItem(
        'selectedPlaces',
        JSON.stringify([id, ...storedIds])
      );
    }
  }

  function handleRemovePlace() {
    setPickedPlaces((prevPickedPlaces) =>
      prevPickedPlaces.filter((place) => place.id !== selectedPlace.current)
    );
    modal.current.close();

    // 제거하는 부분. 위의 추가 로직 참고
    const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
    localStorage.setItem(
      'selectedPlaces',
      JSON.stringify(storedIds.filter((id) => id !== selectedPlace.current))
    );
  }

  return (
    <>
      <Modal ref={modal}>
        <DeleteConfirmation
          onCancel={handleStopRemovePlace}
          onConfirm={handleRemovePlace}
        />
      </Modal>

      ...
    </>
  );
}

export default App;
```

<br/>

## useEffect를 활용하는 다른 적용 사례

```jsx
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
      {children}
    </dialog>,
    document.getElementById('modal')
  );
}

export default Modal;
```

- dialog 에 바로 open 인자를 주면 backdrop 기능이 사라지므로 useEffect를 거치는 것으로 해결
- 이번엔 무한 루프를 방지하고자 함이 아니라 특정 값을 동기화 시킨 후에 실행하기 위함
- 이번 useEffect에는 의존성이 있음

<br/>

## Effect Dependencies 이해하기

- useEffect 의 두 번째 인자로 받은 배열 안의 속성이 변경될 때마다 재실행
- 따라서 open 속성이 변경 될 때마다 재실행

<br/>

## 작은 버그 수정하기

`<dialog>` 요소는 키보드의 ESC 키를 눌러 닫을 수 있습니다. 이 경우, 대화창은 사라지지만 ‘`open`’ Prop (즉, `modalIsOpen` 상태)에 전달된 상태가 `false`로 설정되지 않습니다.

따라서 모달을 다시 열 수 없습니다 (`modalIsOpen` 이 여전히 true이므로 UI는 상태와 동기화되어 있지 않습니다).

이 문제를 해결하기 위해, `<dialog>`에 내장된 `onClose`Prop을 추가하여 모달이 닫히는 것을 청취하도록 해야 합니다. 그런 다음 이 이벤트는 모달 컴포넌트의 커스텀 `onClose` Prop을 허용함으로써 App 컴포넌트로 "전달"됩니다.

Modal 컴포넌트는 다음과 같이 보여야 합니다:App컴포넌트에서는 이제 handleStopRemovePlace 함수를 <Modal> 컴포넌트의 onClose Prop에 값으로 지정할 수 있습니다

```jsx

1. import { useRef, useEffect } from 'react';
2. import { createPortal } from 'react-dom';
3.  
4. function Modal({ open, children, onClose }) {
5.   const dialog = useRef();
6.  
7.   useEffect(() => {
8.     if (open) {
9.       dialog.current.showModal();
10.     } else {
11.       dialog.current.close();
12.     }
13.   }, [open]);
14.  
15.   return createPortal(
16.     <dialog className="modal" ref={dialog} onClose={onClose}>
17.       {children}
18.     </dialog>,
19.     document.getElementById('modal')
20.   );
21. }
22.  
23. export default Modal;
```

`App`컴포넌트에서는 이제 `handleStopRemovePlace`함수를 `<Modal>`컴포넌트의 `onClose`Prop에 값으로 지정할 수 있습니다:

```jsx

1. <Modal open={modalIsOpen} onClose={handleStopRemovePlace}>
2.   <DeleteConfirmation
3.     onCancel={handleStopRemovePlace}
4.     onConfirm={handleRemovePlace}
5.   />
6. </Modal>
```

<br/>

## useEffect의 도움으로 고칠 수 있는 다른 문제들

- 일정 시간 뒤에 모달창을 자동으로 닫는 기능 추가
- useEffect를 사용하지 않으면 취소를 누르든 수락을 누르든 타이머가 멈추지 않고, 기능이 계속되는 문제 존재.

```jsx

import { useEffect } from 'react';

export default function DeleteConfirmation({ onConfirm, onCancel }) {
  useEffect(() => {
    console.log('TIMER SET');
    const timer = setTimeout(() => {
      onConfirm();
    }, 3000);

    return () => {
      console.log('Cleaning up timer');
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

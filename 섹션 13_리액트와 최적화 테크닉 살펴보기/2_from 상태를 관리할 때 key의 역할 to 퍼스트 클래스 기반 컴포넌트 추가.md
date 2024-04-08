### 상태를 관리할 때 키의 역할

- 컴포넌트 함수는 컴포넌트를 재사용할때마다 다시 생성됨
    - Counter 함수를 다시 사용할 거임 (initialCount = 0)
    - 두 Counter 함수의 Counter는 별개로 작동함
        - → State는 Component에 연동됨

![Untitled](https://github.com/chomchom96/ReactStudy/assets/112466460/b5c92986-ccdd-4f16-8224-338461fb00fa)


- CounterHistory Component에서 특정 History를 선택해서 강조할 수 있음
    - Counter의 증가 또는 감소 여부를 위에서부터 새롭게 추가해주는 History 컴포넌트임
    - History 강조 표시한SelectedHistory가
    - SelectedHistory 상태가 컴포넌트를 건너뛰고 관리되고 있음
        - → key = index로 설정했기 때문
        - 선택 기준인 key가 index이기 때문에 index=2인 위치에 하이라이트가 고정됨
- 이를 예방하기 위해 Key를 사용
    - key는 고유해야 하는데 비슷한 컴포넌트가 있으면 중복 키 이슈가 있을 수 있음
    - id 값이 unique 하도록 로직을 추가해줌

```jsx
import { useState, memo, useCallback, useMemo } from 'react';

import IconButton from '../UI/IconButton.jsx';
import MinusIcon from '../UI/Icons/MinusIcon.jsx';
import PlusIcon from '../UI/Icons/PlusIcon.jsx';
import CounterOutput from './CounterOutput.jsx';
import { log } from '../../log.js';
import CounterHistory from '../../components/Counter/CounterHistory.jsx'

function isPrime(number) {
  log('Calculating if is prime number', 2, 'other');

  if (number <= 1) {
    return false;
  }

  const limit = Math.sqrt(number);

  for (let i = 2; i <= limit; i++) {
    if (number % i === 0) {
      return false;
    }
  }

  return true;
}

const Counter = memo(function Counter({ initialCount }) {
  log('<Counter /> rendered', 1);

  const initialCountIsPrime = useMemo(
    () => isPrime(initialCount),
    [initialCount]
  );

  // const [counter, setCounter] = useState(initialCount);
  const [counterChanges, setCounterChanges] = useState([{value: initialCount, id:Math.random()*100}]);

  const currentCounter = counterChanges.reduce(
    (prevCounter, counterChange) => prevCounter + counterChange.value,
    0
  );

  const handleDecrement = useCallback(function handleDecrement() {
    // setCounter((prevCounter) => prevCounter - 1);
    setCounterChanges((prevCounterChanges) => [{value:-1, id:Math.random() * 1000}, ...prevCounterChanges]);
  }, []);

  const handleIncrement = useCallback(function handleIncrement() {
    // setCounter((prevCounter) => prevCounter + 1);
    setCounterChanges((prevCounterChanges) => [{value:1, id:Math.random() * 1000}, ...prevCounterChanges]);
  }, []);

  return (
    <section className="counter">
      <p className="counter-info">
        The initial counter value was <strong>{initialCount}</strong>. It{' '}
        <strong>is {initialCountIsPrime ? 'a' : 'not a'}</strong> prime number.
      </p>
      <div>
        <IconButton icon={MinusIcon} onClick={handleDecrement}>
          Decrement
        </IconButton>
        <CounterOutput value={currentCounter} />
        <IconButton icon={PlusIcon} onClick={handleIncrement}>
          Increment
        </IconButton>
        <CounterHistory history={counterChanges} />
      </div>
    </section>
  );
});

export default Counter;
```

### Key(키)가 중요한 이유 부가 설명

- key를 추가할 때 얻는 이점
    - 예전 방식으로 index를 key로 사용해 보겠음
    - 버튼을 누를 때마다 리스트의 모든 항목이 리렌더링 되는 것을 확인
    - 새로운 아이템이 추가되면 이전 리스트를 버리고 새로운 항목이 들어간 인덱스의 리스트를 업데이트함
    - 인덱스 외에 고유한 key를 사용하면 리액트가 리스트의 기존 요소를 식별해서 일부만 렌더링을 해줄 수 있음

### Key(키)를 사용한 컴포넌트 초기화

- 메인 화면에서 setCounter로 initial value를 바꿔줘도 Counter 컴포넌트의 initial value에 즉각 반영이 되지 않음
- 현재는 initial value가 초기화에만 사용되고 있음
    - useEffect를 사용해서 initial count 변화 시 컴포넌트 함수 재실행

```jsx
  useEffect(() => {
    setCounterChanges([{value: initialCount, id:Math.random()*100}]);
  }, [initialCount])

```

- useEffect는 컴포넌트 함수 실행 후에 실행되는데 state 값이 변하므로 두 번의 컴포넌트 함수 실행을 유발
    - 함수에 key를 설정해줌으로써 해결 가능
    - key가 바뀌면 이전 함수 인스턴스를 자동 삭제함

```jsx
// App
        <Counter key={chosenCount} initialCount={chosenCount} />

```

### 상태 스케줄링 & 배칭

```jsx
// App
function handleSetCount(newCount) {
    setChosenCount(newCount);
    console.log(chosenCount);
}
```

- chosenCount 상태 변화 후에도 변화가 적용되지 않음
    - 상태 업데이트에는 자체 스케줄이 있기 때문
    - 이전 상태 값을 사용해서 새로운 상태 값을 파생하는 방식이 일반적
        - 이전 상태 값의 snapshot을 가지고 새로운 상태 snapshot 반환
    - 여러 상태가 동시에 변화되면 내부 스케줄 순서에 따른 상태 업데이트

```jsx
function handleSetCount(newCount) {
  setChosenCount(newCount); // 무시됨
  setChosenCount(chosenCount + 1); // 실제 적용
}

function handleSetCount(newCount) {
  setChosenCount(newCount);
  setChosenCount((prevChosenCount) => prevChosenCount + 1); // 모두 적용
	console.log(chosenCount); // 출력되지 않음
}
```

- set으로 상태 업데이트를 두 번 하기 때문에 컴포넌트 함수가 두 번 실행되지는 않음
    - 리액트의 state batching 작용 때문

### MillionJS로 리액트 최적화하기

- 무려 70%나 빠르게 해준다는 킹갓라이브러리?
- npm i 후 vite config에 plugin 추가해줌

```jsx
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import million from 'million/compiler'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [million.vite({auto:true}), react()],
})

```

- 에러 시 컴포넌트에 ignore 추가

```
// million-ignore

```

## 14강. 클래스 컴포넌트

### 무엇을 & 왜

- 기존 방식은 컴포넌트로 jsx code function을 반환함 (Functional Component)
    - 유일한 컴포넌트 빌드 방식이 아님
- 클래스형 컴포넌트(Class based Component)
    - 자바스크립트 기본기능인 Class를 생성
    - render 메소드가 필수
    - 최근엔 오류 경계의 예외 때문에 잘 사용하지 않음
        - 16.8 버전 이전에 사용(State, Side Effect)
        - React Hook을 사용할 수 없음! (useEffect 등)

```jsx
class User extends Component {
  render() {
    return <li className={classes.user}>{this.props.name}</li>
  }
}
```

- 

### 퍼스트 클래스 기반 컴포넌트 추가

```jsx
// 기존 컴포넌트
const User = (props) => {
  return <li className={classes.user}>{props.name}</li>;
};

// 클래스 변환 컴포넌트
class User extends Component {
  render() {
    return <li className={classes.user}>{this.props.name}</li>
  }
}
```

- render 메소드에 jsx 코드 반환
- prop을 this.prop으로 받아줌
- react의 Component Class를 extend

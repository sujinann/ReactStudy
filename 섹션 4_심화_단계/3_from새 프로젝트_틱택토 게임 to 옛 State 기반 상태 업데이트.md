## 새 프로젝트: 틱택토 게임 첫 단계

```jsx
function App() {
  return (
    <main>
      <div id="game-container">
        <ol id="players">
					<li>
						<span className="plater-name">Player 1</span>
						<span className="plater-symbol">X</span>
					</li>
          <li>
						<span className="plater-name">Player 2</span>
						<span className="plater-symbol">O</span>
					</li>
        </ol>
        GAME BOARD
      </div>
      LOG
    </main>
  );
}

export default App;
```

1. 일단은 구조를 먼저 짜두고
2. 플레이어 부분을 하드코딩으로 완성
3. 이제부터 진행하며 플레이어 부분을 동적인 값으로 대체할 것임

</br>

## 개념 복습: 컴포넌트 분리 & 재사용 가능한 컴포넌트 구축

위의 코드에서 리스트 부분이 반복되기 때문에 오류 방지와 재사용 목적으로 컴포넌트를 분리함

```jsx
export default function Player({ name, symbol }) {
  return (
    <li>
      <span className="player">
        {playerName}
      <span className="player-symbol">{symbol}</span>
      </span>
      <button>Edit</button>
    </li>
  );
}
```

분리한 컴포넌트를 App.jsx 에 import 해주고 가져와서 사용함

```jsx
import Player from './components/Player.jsx';

<ol id="players">
	<Player name="Player 1" symbol="X" />
	<Player name="Player 2" symbol="O" />
</ol>
```

</br>

## 개념 복습: State 활용법

```jsx
import { useState } from 'react';

export default function Player({ name, symbol }) {
  const [ isEditing, setIsEditing ] = useState(false);

  function handleEditClick() {
    setIsEditing(true);
  }

  let playerName = <span className="player-name">{name}</span>;

  if (isEditing) {
    playerName = <input type="text" required />;
  }

  return (
    <li>
      <span className="player">
        {playerName}
        <span className="player-symbol">{symbol}</span>
      </span>
      <button onClick={handleEditClick}>Edit</button>
    </li>
  );
}
```

Edit 버튼을 동작시키기 위해 useState를 import 함

클릭 하면 현재의 플레이어 이름을 출력하던 부분이 텍스트를 입력받을 수 있도록 변경됨

여기서 required 라는 것을 쓰는데 필수 조건 이라는 역할임

</br>

## 컴포넌트 인스턴스의 분리된 동작법

```jsx
<ol id="players">
	<Player name="Player 1" symbol="X" />
	<Player name="Player 2" symbol="O" />
</ol>
```

같은 컴포넌트를 사용하고 있어도 리액트가 각각 다른 인스턴스를 새로 생성하기 때문에 완전히 따로 동작함

따라서 플레이어1 의 Edit 버튼을 누르더라도 플레이어2 의 Edit field가 활성화 되진 않음

</br>

## 조건적 콘텐츠 & State 업데이트를 위한 차선책

```jsx
import { useState } from 'react';

export default function Player({ name, symbol }) {
  const [isEditing, setIsEditing] = useState(false);

  function handleEditClick() {
    setIsEditing(!isEditing);
  }

  let playerName = <span className="player-name">{name}</span>;
  // let btnCaption = 'Edit';

  if (isEditing) {
    playerName = <input type="text" required value={name} />;
    // btnCaption = 'Save';
  }

  return (
    <li>
      <span className="player">
        {playerName}
        <span className="player-symbol">{symbol}</span>
      </span>
      <button onClick={handleEditClick}>{isEditing ? 'Save' : 'Edit'}</button>
    </li>
  );
}
```

삼항 연산자로 isEditing의 상태에 따라 버튼의 출력값을 Save와 Edit으로 나누어 출력하도록 변경함

이때 삼항 연산자 대신에 주석과 같이 처리해도 됨

버튼을 여러번 눌러서 Save와 Edit을 왔다갔다 하는 기능을 원하므로 true고정값으로 바꿔주던 handleEditClick() 함수에 !isEditing 을 사용하여 더 유연하게 동작할 수 있도록 고쳐줌
하지만 이는 아직도 차선책임

</br>

## 실습: 옛 State를 기반으로 올바르게 상태 업데이트 하기

```jsx
function handleEditClick() {
	setIsEditing((editing) => !editing);
}
```

(editing) 여기 부분에서 리액트가 자동으로 이전 상태 값을 기반으로 가져오기 때문에 이 방식을 써야 함

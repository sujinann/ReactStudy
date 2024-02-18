### 계산된 값에서 새로운 값 파생하기

```jsx
for (const combination of WINNING_COMBINATIONS) {
    const firstSquareSymbol = gameBoard[combination[0].row][combination[0].column];
    const secondSquareSymbol = gameBoard[combination[1].row][combination[1].column];
    const thirdSquareSymbol = gameBoard[combination[2].row][combination[2].column];
  
    if (firstSquareSymbol) {
      if (firstSquareSymbol === secondSquareSymbol === thirdSquareSymbol) {
        winner =firstSquareSymbol; 
      }
    }
  }
```

### 게임 오버 화면 꾸미기 & 무승부 여부

- Game over component

```jsx
//GameOver.jsx
export default function GameOver({winner}) {
    return <div id="game-over">
        <h2>Game Over!</h2>
        <p>{winner} won!</p>
        <p><button>Rematch!</button></p>
    </div>
}

// App
{winner && <GameOver winner={winner} />}
```

- 무승부 로직

```jsx
const hasDraw = gameTurns.length === 9 & !winner;
...
{(winner || hasDraw) && <GameOver winner={winner} />}

// GameOver.jsx
{winner && <p>{winner} won!</p>}
{!winner && <p>It&apos;s a draw!</p>}
```

### 불변성이 중요한 이유

- 코드의 모든 요소들이 GameTurns 변수 의존도가 매우 높음 (현재 플레이어, 승자, 무승부 여부 등)
- 게임을 재시작할 때 GameTurns를 비우면 모든 변수가 초기화됨

```jsx
// App.jsx
function handleRestart() {
    setGameTurns([]);
}

{(winner || hasDraw) && <GameOver winner={winner} onRestart={handleRestart}/>}
// GameOver.jsx
<p><button onClick={onRestart}>Rematch!</button></p>
```

→ 제대로 작동하지 않음

- gameBoard는 gameTurns에 의존해 동적 작용
- gameBoard는 Array → JS의 참조형 변수
    - 게임을 재시작해도 배열은 예전 메모리의 참조값을 유지
- solution : 재시작시 배열을 deepcopy해 새로 만들기

```jsx
let gameBoard = [...initialGameBoard];
```

### State를 끌어올리면 안되는 이유

- 승자를 player mark에 따라 정했었음 (X, O)
    - Player의 Name, Symbol은 App의 Player 컴포넌트 내에서만 매핑되어 있음
    - Player State를 App까지 끌어올려서 사용하고 싶음
    - Player.jsx 내에서 입력값에 따라 입력 필드를 업데이트하는 State를 끌어올리면 입력값을 한 번 누를 때마다 전체 App.jsx의 상태가 재평가됨
    
    + 매우 어려움
    
    - 

```jsx
// App.jsx
const [players, setPlayers]= useState({'X' : 'Player 1', 'O' : 'Player 2'})

function handlePlayerNameChange(symbol, newName) {
    setPlayers(prevPlayers => {
      return {
        ...prevPlayers,
        [symbol]:newName
      }
    });
  }
```

- 이름을 변경하는 메서드가 발생할 때만 발생
- player의 예전 state를 가져와서 overwrite 하는 방식으로 처리하면 player 전체를 reevaluate 하는 일이 없음

### State 끌어올리기 대안

- handlePlayerNameChange를 Player component 내에서 작동하도록 하는 게 이상적임
    - 포인터를 Player component 까지 내림

```jsx
// App.jsx
<Player initialName="Player 1" symbol="X" isActive={activePlayer === 'X'} onChangeName={handlePlayerNameChange}/>

// Player.jsx
function handleEditClick() {
    setIsEditing((isEditing) => !isEditing);
    if (isEditing) onChangeName(symbol, playerName)
}

// 다시 App.jsx
winner = firstPlayerSymbol -> winner = players[firstSquareSymbol];
```

### 마무리 및 개선

1. 전체 winner 결정 로직을 함수로 outsourcing 할 수 있음
    - setGameBoard도 동일

```jsx
let winner;
  // 무승부 : 모든 보드가 채워짐 -> max turn = 9
  for (const combination of WINNING_COMBINATIONS) {
    const firstSquareSymbol = gameBoard[combination[0].row][combination[0].column];
    const secondSquareSymbol = gameBoard[combination[1].row][combination[1].column];
    const thirdSquareSymbol = gameBoard[combination[2].row][combination[2].column];
  
    if (firstSquareSymbol) {
      if (firstSquareSymbol === secondSquareSymbol === thirdSquareSymbol) {
        winner = players[firstSquareSymbol]; 
      }
    }
    const hasDraw = gameTurns.length === 9 & !winner;
}
```

1. function App 위에 초기 게임 상태를 상수로 저장할 수 있음

```jsx
const PLAYERS = {
  X: 'Player 1',
  O: 'Player 2'
}

const initialGameBoard = [[null, null, null],[null, null, null],[null, null, null],]
```

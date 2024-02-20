## 계산된 값 권장 및 불필요한 상태 관리 방지

게임보드를 계속 상태 관리할 필요 없이, 꼭 필요한 게임 턴에서 부터 계산이 가능함.

GameBoard.jsx

```jsx
// import { useState } from 'react';

const initialGameBoard = [
  [null, null, null],
  [null, null, null],
  [null, null, null],
];

export default function GameBoard({ onSelectSquare }) { // activePlayerSymbol 제거
  // const [gameBoard, setGameBoard] = useState(initialGameBoard);

  // function handleSelectSquare(rowIndex, colIndex) {
  //   setGameBoard((prevGameBoard) => {
  //     const updatedBoard = [...prevGameBoard.map(innerArray => [...innerArray])];
  //     updatedBoard[rowIndex][colIndex] = activePlayerSymbol;
  //     return updatedBoard;
  //   });

  //   onSelectSquare();
  // }

  return (
    <ol id="game-board">
      {gameBoard.map((row, rowIndex) => (
        <li key={rowIndex}>
          <ol>
            {row.map((playerSymbol, colIndex) => (
              <li key={colIndex}>
                <button onClick={onSelectSquare}>{playerSymbol}</button>
	              // handleSelectSquare(rowIndex, colIndex)
							</li>
            ))}
          </ol>
        </li>
      ))}
    </ol>
  );
}
```

App.jsx

```jsx
import { useState } from 'react';

import Player from './components/Player.jsx';
import GameBoard from './components/GameBoard.jsx';
import Log from './components/Log.jsx'; // 추가

function App() {
  const [gameTurns, setGameTurns] = useState([]); // 추가
  const [activePlayer, setActivePlayer] = useState('X');

//	function handleSelectSquare() {
//    setActivePlayer((curActivePlayer) => curActivePlayer === 'X' ? 'O' : 'X');
//  }
// 아래로 변경

  function handleSelectSquare(rowIndex, colIndex) {
    setActivePlayer((curActivePlayer) => (curActivePlayer === 'X' ? 'O' : 'X'));
		// 여기 부분 추가
		setGameTurns((prevTurns) => {
      let currentPlayer = 'X';

      if (prevTurns.length > 0 && prevTurns[0].player === 'X') {
        currentPlayer = 'O';
      }

      const updatedTurns = [
        { square: { row: rowIndex, col: colIndex }, player: currentPlayer },
        ...prevTurns,
      ];

      return updatedTurns;
    });
  }

  return (
    <main>
      <div id="game-container">
        <ol id="players" className="highlight-player">
          <Player
            initialName="Player 1"
            symbol="X"
            isActive={activePlayer === 'X'}
          />
          <Player
            initialName="Player 2"
            symbol="O"
            isActive={activePlayer === 'O'}
          />
        </ol>
        <GameBoard
          onSelectSquare={handleSelectSquare}
          activePlayerSymbol={activePlayer}
        />
      </div>
      <Log />
    </main>
  );
}

export default App
```

<br/>

## props에서 state 파생하기

Apps.jsx 에서 gameTurns를 넘겨주도록 변경

```jsx
<GameBoard
  onSelectSquare={handleSelectSquare}
  turns={gameTurns}
/>
```

```jsx
const initialGameBoard = [
  [null, null, null],
  [null, null, null],
  [null, null, null],
];

// turns prop 추가
export default function GameBoard({ onSelectSquare, turns }) {
  // 게임 턴을 이용해 계산된 게임판 구성 추가
	let gameBoard = initialGameBoard;

  for (const turn of turns) {
    const { square, player } = turn;
    const { row, col } = square;

    gameBoard[row][col] = player;
  }

  return (
    <ol id="game-board">
      {gameBoard.map((row, rowIndex) => (
        <li key={rowIndex}>
          <ol>
            {row.map((playerSymbol, colIndex) => (
              <li key={colIndex}>
								// 행열 인자 다시 넘겨주기
                <button onClick={() => onSelectSquare(rowIndex, colIndex)}>{playerSymbol}</button>
              </li>
            ))}
          </ol>
        </li>
      ))}
    </ol>
  );
}
```

<br/>

## 컴포넌트 간의 상태 공유

Log.jsx

```jsx
export default function Log({ turns }) {
  return (
    <ol id="log">
      {turns.map((turn) => (
        <li key={`${turn.square.row}${turn.square.col}`}>
          {turn.player} selected {turn.square.row},{turn.square.col}
        </li>
      ))}
    </ol>
  );
}
```

App.jsx

```jsx
<Log turns={gameTurns} />
```

</br>

## 상태 관리 간소화

App.jsx

```jsx
import { useState } from 'react';

import Player from './components/Player.jsx';
import GameBoard from './components/GameBoard.jsx';
import Log from './components/Log.jsx';

// 상태관리 간소화를 위해 새로운 함수 추가. (gameTurns값으로 나머지 파생. 중복도 제거.)
function deriveActivePlayer(gameTurns) {
  let currentPlayer = 'X';

  if (gameTurns.length > 0 && gameTurns[0].player === 'X') {
    currentPlayer = 'O';
  }

  return currentPlayer;
}

function App() {
  const [gameTurns, setGameTurns] = useState([]);
  // const [activePlayer, setActivePlayer] = useState('X'); 이 부분 간소화

  const activePlayer = deriveActivePlayer(gameTurns); // 새 함수 사용

  function handleSelectSquare(rowIndex, colIndex) {
    // setActivePlayer((curActivePlayer) => (curActivePlayer === 'X' ? 'O' : 'X'));
    setGameTurns((prevTurns) => {
      const currentPlayer = deriveActivePlayer(prevTurns); // 새 함수 사용

      const updatedTurns = [
        { square: { row: rowIndex, col: colIndex }, player: currentPlayer },
        ...prevTurns,
      ];

      return updatedTurns;
    });
  }

  return (
    <main>
      <div id="game-container">
        <ol id="players" className="highlight-player">
          <Player
            initialName="Player 1"
            symbol="X"
            isActive={activePlayer === 'X'}
          />
          <Player
            initialName="Player 2"
            symbol="O"
            isActive={activePlayer === 'O'}
          />
        </ol>
        <GameBoard onSelectSquare={handleSelectSquare} turns={gameTurns} />
      </div>
      <Log turns={gameTurns} />
    </main>
  );
}

export default App;
```

<br/>

## 조건적 버튼 비활성화

GameBoard.jsx

```jsx
return (
  <ol id="game-board">
    {gameBoard.map((row, rowIndex) => (
      <li key={rowIndex}>
        <ol>
          {row.map((playerSymbol, colIndex) => (
            <li key={colIndex}>
              <button
                onClick={() => onSelectSquare(rowIndex, colIndex)}
                disabled={playerSymbol !== null} // 추가.
              >
                {playerSymbol}
              </button>
            </li>
          ))}
        </ol>
      </li>
    ))}
  </ol>
);
```

<br/>

## 분리된 파일로 데이터 아웃소싱

이기는 조건을 파일로 정리해두고 앱으로 불러오기

App.jsx

```jsx
import { WINNING_COMBINATIONS } from './winning-combinations.js';
```

winning-combinations.js

```jsx
export const WINNING_COMBINATIONS = [
	[
    { row: 0, column: 0 },
    { row: 0, column: 1 },
    { row: 0, column: 2 },
  ],

	...   // 생략

];
```

<br/>

## 계산된 값 끌어올리기

이제 게임 보드를 앱에서 관리. 게임보드 컴포넌트에 넘겨줌. 

게임 보드 상태에 접근 용이하여 승리 분석 가능.

App.jsx

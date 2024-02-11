# 사용자 입력 & 양방향 바인딩 ~ 교차 State(상태) 방지하기

### 사용자 입력 & 양방향 바인딩

- 편집 모드에 들어가면 수정이 되지 않는 상황을 해결해보자.
    - defaultValue={초기값}
        - value 대신 defaultValue로 초기 값을 설정해 값을 덮어쓰는 것을 방지. ⇒ 수정은 가능.
        
        ```jsx
        # Player.jsx
        
        if (isEditing) {
            playerName = <input type="text" required defualtValue={name} />;
        }
        ```
        
        - 수정 값이 아직 저장되지 않고 Player 1으로만 저장되는 상태. ⇒ 문제가 해결되지 않음.
    - 양방향 바인딩: 입력값의 변화에 반응하고 변경 값을 다시 입력값에 전달.
        
        ```jsx
        # App.jsx
        
        import Player from "./components/Player.jsx";
        
        function App() {
          return (
            <main>
              <div id="game-container">
                <ol id="players">
                  {/* 이름 속성명을 initialName으로 수정 */}
                  <Player initialName="Player 1" symbol="X" />
                  <Player initialName="Player 2" symbol="O" />
                </ol>
                GAME BOARD
              </div>
              LOG
            </main>
          );
        }
        
        export default App;
        ```
        
        ```jsx
        # Player.jsx
        
        import { useState } from "react";
        
        export default function Player({ initialName, symbol }) {
          // 이름이 변할 때마다 그 값을 {playerName}에 반환하기 위해 또 하나의 state가 필요함
          // 구조 분해 할당을 이용해 useState 안에 이름 속성(initialName)을 넣어 초기 값으로 사용
          // playerName 이름 상태를 동적 값으로 사용
          const [playerName, setPlayerName] = useState(initialName); // 제어하려는 상태가 많으면 동일 컴포넌트 내에서 useState를 여러 번 사용 가능
          const [isEditing, setIsEditing] = useState(false);
        
          function handleEditClick() {
            setIsEditing((editing) => !editing);
          }
        
          function handleChange(event) {
            // onChange 이벤트 객체를 event로 명명
            console.log(event); // 이벤트 객체는 리액트가 자동으로 넘어오게 한다
            setPlayerName(event.target.value); // .target 속성을 작성해 이벤트를 방출한 요소(하단의 input 요소)를 참조하게 된다. .value 속성은 이용자가 작성한 값을 저장한다.
          } // 작성하는 이름의 상태 변화를 감지하는 함수를 추가로 작성
        
          // 변수명 충돌 방지를 위해 editablePlayerName으로 변수명 수정
          let editablePlayerName = <span className="player-name">{playerName}</span>; // 동적 값인 playerName으로 수정
        
          if (isEditing) {
            editablePlayerName = // 충돌 방지용 변수명 수정
              (
                <input
                  type="text"
                  required
                  value={playerName}
                  onChange={handleChange}
                /> // 동적 값인 playerName으로 수정 + 변화를 감지하는 이벤트로 handleChange 함수를 실행
                // onChange 이벤트는 매 작성값, 매 입력(키보드를 누를 때)에 의해 발동. 이벤트 객체(event로 명명)를 제공.
              );
          }
        
          return (
            <li>
              <span className="player">
                {editablePlayerName} {/* 충돌 방지용 변수명 수정 */}
                <span className="player-symbol">{symbol}</span>
              </span>
              <button onClick={handleEditClick}>{isEditing ? "Save" : "Edit"}</button>
            </li>
          );
        }
        ```
        
        - 제어하려는 상태가 많으면 동일 컴포넌트 내에서 useState를 여러 번 사용 가능.
        - playerName 이름 state(상태)를 동적 값으로 사용.
            - 이 동적 값을 사용할 모든 {name}을 {playerName}으로 수정.
        - 구조 분해 할당을 이용해 useState 안에 이름 속성(initialName)을 넣어 초기 값으로 사용.
            - 바깥 component에서도 이름 속성명인 name을 initialName으로 수정.
        - 이름이 변할 때마다 그 값을 {playerName}에 반환하기 위해 또 하나의 state가 필요.
        - 변수명 충돌 방지를 위해 playerName을 editablePlayerName으로 수정.
            - 편집 모드일 때 화면에서 나타낼 모든 플레이어 이름 변수명들도 playerName에서 editablePlayerName으로 수정.
        - input 요소에 onChange 이벤트를 부여해 매 작성값, 매 입력(키보드를 누를 때)에 의해 발동하도록 함.
            - 이 때, 리액트 측에서 이벤트 객체(여기서는 event로 명명함)를 자동으로 제공한다.
        - 제공된 이벤트 객체를 받을 handleChange(event) 함수를 작성.
            - 이 함수는 작성하는 이름의 상태 변화를 감지하는 함수로 활용된다.
            - 함수 내에 setPlayerName(event.target.value);를 작성.
                - .target 속성: 이벤트를 방출한 요소(하단의 input 요소)를 참조하게 된다.
                - .value 속성: 이용자가 작성한 값을 저장한다.
    - 동작 순서:
        - handleChange에 입력한 값이 onChange 이벤트에 의해 반영되어 저장.
        - 그 입력값이 playerName으로 넘어가 저장되고, 화면에는 그 값이 반영됨.

### 다차원 리스트 렌더링

- 3x3 그리드 게임판 모양을 만들어보자.
    
    ```jsx
    # App.jsx
    
    import Player from "./components/Player.jsx";
    import GameBoard from "./components/GameBoard.jsx";
    
    function App() {
      return (
        <main>
          <div id="game-container">
            <ol id="players">
              <Player initialName="Player 1" symbol="X" />
              <Player initialName="Player 2" symbol="O" />
            </ol>
            {/* 아직 아무 속성도 받고 있지 않지만, GameBoard를 import해온다. */}
            <GameBoard />
          </div>
          LOG
        </main>
      );
    }
    
    export default App;
    ```
    
    ```jsx
    # GameBoard.jsx
    // 3x3 그리드 게임판이 화면에 나타남
    // 버튼 위에 두면 커서 모양은 변하지만, 눌러도 아무 변화가 없는 상태
    
    const initialGameBoard = [
      // 안의 세 배열은 바깥 배열의 각 아이템(row로 명명)에 속함
      [null, null, null],
      [null, null, null],
      [null, null, null],
    ];
    
    export default function GameBoard() {
      // map 방식을 이용. map은 함수를 입력으로 받고, 그 함수는 해당 배열의 각 아이템에 대해 자동으로 호출.
      return (
        <ol id="game-board">
          {initialGameBoard.map(
            (
              row,
              rowIndex, // rowIndex라는 인수를 추가해 key로 활용. 원래는 data가 아닌 data의 위치를 나타내기 때문에 변동될 수 있어 잘 쓰이지 않음.
            ) => (
              <li key={rowIndex}>
                {/* 각 칸의 null 등 값을 동적으로 가져오기 위해 col과 그 인덱스인 colIndex를 활용 */}
                <ol>
                  {row.map(
                    (
                      playerSymbol,
                      colIndex, // col은 적절치 않아 playerSymbol로 이름을 수정
                    ) => (
                      <li key="colIndex">
                        {/* col(playerSymbol)은 동적 값으로 초기값인 null, 플레이어의 기호인 X 혹은 O를 가져오게 함. */}
                        {/* col은 적절치 않아 playerSymbol로 이름을 수정 */}
                        <button>{playerSymbol}</button>
                      </li>
                    ),
                  )}
                </ol>
              </li>
            ),
          )}
        </ol>
      ); // 각 목록 아이템은 식별을 위한 고유값인 key(= 속성명)이 필요.
    }
    ```
    
    - GameBoard.jsx라는 새 파일을 생성한다.
        - initialGameBoard라는 배열을 만들어 안에 세 배열을 집어넣는다.
            - 각 배열은 해당 배열의 각 아이템(row로 명명)에 속한다.
        - map을 활용해 동적으로 보드판을 형성한다.
            - map 방식: 함수를 입력으로 받고, 그 함수는 해당 배열의 각 아이템에 대해 자동으로 호출.
            - row로 명명한 아이템을 활용하기 위해 row와 rowIndex 두 인수를 map에 넣는다.
                - rowIndex라는 인수를 추가해 key로 활용.
                    - 각 목록 아이템은 식별을 위한 고유값인 key(= 속성명)이 필요.
                    - 원래는 data가 아닌 data의 위치를 나타내기 때문에 변동될 수 있어 인덱스는 잘 쓰이지 않음.
            - rowIndex로 나열된 목록 안에 또 다른 목록을 작성한다.
            - col(의미에 맞도록 playerSymbol로 수정됨)과 colIndex 두 인수를 map에 넣는다.
                - playerSymbol은 동적 값.
                - 초기값인 null, 플레이어의 상징 기호인 X, O를 가져오게 함.
            - 각 칸 안에 버튼을 작성하고 그 값으로 playerSymbol을 동적으로 반영하게 함.
    - 실행 결과:
        - 3x3 그리드 게임판이 화면에 나타남.
        - 각 버튼 위에 두면 커서 모양은 변하지만, 눌러도 아무 변화가 없는 상태.

### 추천 실습: 불변의 객체 State(상태)로 업데이트하기

- 칸을 누르면 X 표시가 나타나도록 상태 변화를 감지하는 함수를 작성해보자.
    
    ```jsx
    # GameBoard.jsx
    
    import { useState } from "react";
    
    const initialGameBoard = [
      [null, null, null],
      [null, null, null],
      [null, null, null],
    ];
    
    export default function GameBoard() {
      const [gameBoard, setGameBoard] = useState(initialGameBoard); // 상태 변화 감지를 위해 useState 이용: initialGameBoard를 초기값으로 활용.
    
      // 이렇게 작성하는 건 추천되지 않음.
      // 리액트가 실행하는 예정된 상태 업데이트보다 이전에 일어날 수 있기 때문.
      // 위와 같은 상황은 앱 내부의 한 상태에 대해 여러 개의 상태 업데이트가 예정되어 있을 때 발생.
      // 객체와 배열은 참조값이므로 직접 변경하기보다 (deep) copy를 만들어야!
      // 객체와 배열의 원본은 불변하는 방식으로 수정 작업을 거치게 해야.
    	//
      //   function handleSelectSquare(rowIndex, colIndex) {
      //     setGameBoard((prevGameBoard) => {
      //       prevGameBoard[rowIndex][colIndex] = "X";
      //       return prevGameBoard;
      //     });
      //   }
    
      // (deep) copy를 만들 때는 Javascript의 스프레드 연산자(...)를 활용한다.
      function handleSelectSquare(rowIndex, colIndex) {
        // 버튼 클릭 이벤트로 실행될 함수 작성.
        // 정확히 어느 row번호, col번호에 있는 칸의 null을 상징 기호로 바꿔야 하는지를 알아야.
        setGameBoard((prevGameBoard) => {
          const updatedBoard = [
            ...prevGameBoard.map((innerArray) => [...innerArray]),
          ];
          updatedBoard[rowIndex][colIndex] = "X"; // 플레이어에 따라 다른 기호가 나타나는 건 추후 작성 (일단 X로 통일).
          return updatedBoard; // 상태 변화에 따라 새로운 3x3 배열로 갈아끼우기 위한 함수 실행.
        });
      }
    
      return (
        <ol id="game-board">
          {gameBoard.map( // initialGameBoard 대신 gameBoard로 수정
            (
              row,
              rowIndex,
            ) => (
              <li key={rowIndex}>
                <ol>
                  {row.map((playerSymbol, colIndex) => (
                    <li key="colIndex">
                      <button
                        onClick={() => handleSelectSquare(rowIndex, colIndex)}
                      >
                        {/* onClick 이벤트로 handleSelectSquare 함수 실행. */}
                        {playerSymbol}
                      </button>
                    </li>
                  ))}
                </ol>
              </li>
            ),
          )}
        </ol>
      );
    }
    ```
    
    - 상태 변화 감지를 위해 useState 이용.
        - initialGameBoard를 초기값으로 한 번만 활용.
    - 버튼의 onClick 이벤트로 handleSelectSquare 함수 실행.
        - 상태 변화에 따라 새로운 3x3 배열로 갈아끼우기 위한 함수를 작성해 실행.
        - 정확히 어느 row번호, col번호에 있는 칸의 null을 상징 기호로 바꿔야 하는지를 알아야.
        - rowIndex와 colIndex 두 값을 함수에 집어넣는다.
        - 플레이어에 따라 다른 기호가 나타나는 건 추후 작성 (일단 X로 통일).
    - 객체와 배열을 직접 수정하는 것은 추천되지 않음. 원본은 불변하는 방식으로 수정 작업을 거치게 해야 함.
        - 리액트가 실행하는 예정된 상태 업데이트보다 이전에 일어날 수 있기 때문.
            - 이 상황은 앱 내부의 한 상태에 대해 여러 개의 상태 업데이트가 예정되어 있을 때 주로 발생.
        - 객체와 배열은 참조값이므로 직접 변경하기보다 (deep) copy를 만들어야 함.
    - 배열의 (deep) copy를 만들 때는 Javascript의 스프레드 연산자(...)를 활용한다.
    - 실행 결과:
        - 각 칸을 누르면 X 표가 잘 나타남.
        - 아직 플레이어의 턴 교체가 없고, X 표만 나타나고 있어 누가 이기고 있는지 알 수 없음.

### State(상태) 끌어올리기 [핵심 개념]

- 현재 플레이어 차례에 따라 플레이어에 하이라이트 효과를 부여하고, 해당 플레이어의 기호가 칸에 번갈아 표시되도록 해보자.
    
    ```jsx
    # App.jsx
    
    import { useState } from "react"; // 부모 컴포넌트인 App.jsx에서 prop을 보내기 위해 useState를 활용
    
    import Player from "./components/Player.jsx";
    import GameBoard from "./components/GameBoard.jsx";
    
    function App() {
      const [activePlayer, setActivePlayer] = useState("X"); // 현재 플레이 차례에 따라 기호를 변경
    
      // 격자판의 버튼을 클릭한 후에 차례가 변경되기 때문에 handleSelectSquare 함수를 사용
      function handleSelectSquare() {
        setActivePlayer((curActivePlayer) => (curActivePlayer === "X" ? "O" : "X")); // 현재 차례가 X일 경우 O로, 아닐 경우 X로 변경
      }
    
      return (
        <main>
          <div id="game-container">
            {/* CSS 디자인 적용을 위해 className을 작성해준다. */}
            <ol id="players" className="highlight-player">
              {/* Player 1은 X일 때 isActive가 true가 되도록 */}
              <Player
                initialName="Player 1"
                symbol="X"
                isActive={activePlayer === "X"}
              />
              <Player
                initialName="Player 2"
                symbol="O"
                isActive={activePlayer === "O"}
              />
              {/* Player 2는 O일 때 isActive가 true가 되도록 */}
            </ol>
            {/* GameBoard로 onSelectSquare라는 prop으로 넘겨준 뒤, 그 안에서 함수를 호출하면 바깥의 App.jsx에 작성된 handleSelectSquare 함수가 실행되게 함. */}
            {/* activePlayerSymbol이라는 prop으로 activePlayer 값(= 현재 플레이어의 기호)을 넘겨준다. */}
            <GameBoard
              onSelectSquare={handleSelectSquare}
              activePlayerSymbol={activePlayer}
            />
          </div>
          LOG
        </main>
      );
    }
    
    export default App;
    ```
    
    ```jsx
    # GameBoard.jsx
    
    import { useState } from "react";
    
    const initialGameBoard = [
      [null, null, null],
      [null, null, null],
      [null, null, null],
    ];
    
    export default function GameBoard({ onSelectSquare, activePlayerSymbol }) {
      // 버튼에 표시할 activePlayerSymbol이라는 prop을 받아 집어넣는다.
      // App.jsx에서 넘겨받은 onSelectSquare라는 prop을 집어넣는다.
      const [gameBoard, setGameBoard] = useState(initialGameBoard);
    
      function handleSelectSquare(rowIndex, colIndex) {
        setGameBoard((prevGameBoard) => {
          const updatedBoard = [
            ...prevGameBoard.map((innerArray) => [...innerArray]),
          ];
          updatedBoard[rowIndex][colIndex] = activePlayerSymbol; // prop으로 받아온 activePlayerSymbol로 수정해 'O'도 제대로 뜰 수 있도록 함.
          return updatedBoard;
        });
    
        onSelectSquare(); // 이 함수를 호출하고 나면, App.jsx에 작성된 해당 함수가 실행된다.
      }
    
      return (
        <ol id="game-board">
          {gameBoard.map((row, rowIndex) => (
            <li key={rowIndex}>
              <ol>
                {row.map((playerSymbol, colIndex) => (
                  <li key="colIndex">
                    <button onClick={() => handleSelectSquare(rowIndex, colIndex)}>
                      {playerSymbol}
                    </button>
                  </li>
                ))}
              </ol>
            </li>
          ))}
        </ol>
      );
    }
    ```
    
    ```jsx
    # Player.jsx
    
    import { useState } from "react";
    
    export default function Player({ initialName, symbol, isActive }) {
      // isActive이라는 prop을 바깥으로부터 받아온다.
      const [playerName, setPlayerName] = useState(initialName);
      const [isEditing, setIsEditing] = useState(false);
    
      function handleEditClick() {
        setIsEditing((editing) => !editing);
      }
    
      function handleChange(event) {
        console.log(event);
        setPlayerName(event.target.value);
      }
    
      let editablePlayerName = <span className="player-name">{playerName}</span>;
    
      if (isEditing) {
        editablePlayerName = (
          <input type="text" required value={playerName} onChange={handleChange} />
        );
      }
    
      return (
        // isActive의 T/F 여부에 따라 active라는 CSS 디자인을 적용할지, 안 할지를 결정하도록 작성한다.
        <li className="isActive ? 'active' : undefined">
          <span className="player">
            {editablePlayerName}
            <span className="player-symbol">{symbol}</span>
          </span>
          <button onClick={handleEditClick}>{isEditing ? "Save" : "Edit"}</button>
        </li>
      );
    }
    ```
    
    - 다른 component들에 동일한 값이 필요할 때: Lifting the state up(= 상태 끌어올리기) 활용.
        - 부모 component에 해당하는 App.vue에서 두 자식 component 모두에게 속성(prop)을 보낼 수 있다는 점을 이용한다.
    - 부모 컴포넌트인 App.jsx에서 prop을 보내기 위해 useState를 활용.
        - 현재 플레이 차례에 따라 기호를 변경하기 위함.
    - 격자판의 버튼을 클릭한 후에 차례가 변경되기 때문에 handleSelectSquare 함수를 사용.
        - 현재 차례가 X일 경우 O로, 아닐 경우 X로 변경.
        - GameBoard로 onSelectSquare라는 prop으로 넘겨준 뒤, 그 안에서 함수를 호출하면 바깥의 App.jsx에 작성된 handleSelectSquare 함수가 실행되게 함.
        - GameBoard 함수에 App.jsx에서 넘겨받은 onSelectSquare라는 prop을 집어넣는다.
        - GameBoard 안에서 이 함수를 호출하고 나면, App.jsx에 작성된 해당 함수가 실행된다.
    - 본인 차례일 때 하이라이트 효과 CSS 디자인을 적용하기 위해 ol에 className을 작성해준다.
        - Player에서 사용할 isActive이라는 prop을 바깥으로부터 받아온다.
        - isActive의 T/F 여부에 따라 active라는 CSS 디자인을 적용할지 여부를 결정하도록 작성.
            - active CSS 디자인을 적용하면 하이라이트 효과가 부여된다.
        - Player 1은 X일 때 isActive가 true가 되도록, Player 2는 O일 때 isActive가 true가 되도록.
    - activePlayerSymbol이라는 prop으로 activePlayer 값(= 현재 플레이어의 기호)을 GameBoard로 넘겨준다.
        - GameBoard에서 버튼에 표시할 activePlayerSymbol이라는 prop을 받아 집어넣는다.
        - ‘X’로만 뜨던 것을 prop으로 받아온 activePlayerSymbol로 수정해서 'O'도 제대로 뜰 수 있도록 함.
    - 실행 결과:
        - 플레이어 차례에 따라 그 플레이어에 하이라이트 효과가 부여됨.
        - 현재 차례인 플레이어의 기호가 칸에 제대로 표시됨. (O / X)
        - 아직은 같은 버튼을 여러 번 누를 수 있어 기호를 덮어씌울 수 있음.
        - 아직 승리했을 때의 기능이 따로 없음.

### 교차 State(상태) 방지하기

- 로그를 추가해보자. (Log.jsx 파일 새로 추가함)
    
    ```jsx
    # App.jsx
    
    import { useState } from "react";
    
    import Player from "./components/Player.jsx";
    import GameBoard from "./components/GameBoard.jsx";
    import Log from "./components/Log.jsx";
    
    function App() {
      // 버튼 클릭마다 아래 배열에 순서를 하나씩 추가되게끔 하자.
      const [gameTurns, setGameTurns] = useState([]); // 순서 데이터 배열 상태를 보기 위한 useState
      const [activePlayer, setActivePlayer] = useState("X");
    
      function handleSelectSquare() {
        setActivePlayer((curActivePlayer) => (curActivePlayer === "X" ? "O" : "X"));
      }
    
      return (
        <main>
          <div id="game-container">
            <ol id="players" className="highlight-player">
              <Player
                initialName="Player 1"
                symbol="X"
                isActive={activePlayer === "X"}
              />
              <Player
                initialName="Player 2"
                symbol="O"
                isActive={activePlayer === "O"}
              />
            </ol>
            <GameBoard
              onSelectSquare={handleSelectSquare}
              activePlayerSymbol={activePlayer}
            />
          </div>
          {/* 새로 작성한 Log.jsx를 이 곳에 불러온다. */}
          <Log />
        </main>
      );
    }
    
    export default App;
    ```
    
    ```jsx
    # GameBoard.jsx
    
    import { useState } from "react";
    
    const initialGameBoard = [
      [null, null, null],
      [null, null, null],
      [null, null, null],
    ];
    
    export default function GameBoard({ onSelectSquare, activePlayerSymbol }) {
      // 필요 없어진 부분 => 주석 처리. App.jsx에서 gameTurns로 대체.
      //   const [gameBoard, setGameBoard] = useState(initialGameBoard);
    
      //   function handleSelectSquare(rowIndex, colIndex) {
      //     setGameBoard((prevGameBoard) => {
      //       const updatedBoard = [
      //         ...prevGameBoard.map((innerArray) => [...innerArray]),
      //       ];
      //       updatedBoard[rowIndex][colIndex] = activePlayerSymbol;
      //       return updatedBoard;
      //     });
    
      //     onSelectSquare();
      //   }
    
      return (
        <ol id="game-board">
          {gameBoard.map((row, rowIndex) => (
            <li key={rowIndex}>
              <ol>
                {row.map((playerSymbol, colIndex) => (
                  <li key="colIndex">
                    <button onClick={() => handleSelectSquare(rowIndex, colIndex)}>
                      {playerSymbol}
                    </button>
                  </li>
                ))}
              </ol>
            </li>
          ))}
        </ol>
      );
    }
    ```
    
    - 유사한 정보를 가진 상태를 또 관리할 필요는 없다.
        - 게임판 작동을 통해 몇 번 클릭되었는지, 누구 차례인지는 파악할 수 있기 때문.
    - 누구 차례인지 등의 상태를 App.jsx에서 처리하도록 하자.
        - GameBoard.jsx에서 게임판 상태에 대한 코드는 필요 없어졌으니 주석 처리한다.
        - 상태를 끌어올려 App.jsx에 작성한 gameTurns 배열을 통해 GameBoard와 Log 둘 다에 각각 필요한 정보를 알맞게 넘겨주는 방식으로 전환.

## 모듈 소개

기존에 배운 리액트 개념들을 복습하며 Investment Calculator Wep app 만들기

컴포넌트와 상태관리,

리스트 출력과 조건관리 등

<br/>

## 헤더 컴포넌트 추가하기

Header.jsx

```jsx
import logo from '../assets/investment-calculator-logo.png';

export default function Header() {
  return (
    <header id="header">
      <img src={logo} alt="Logo showing a money bag" />
      <h1>Investment Calculator</h1>
    </header>
  );
}
```

src안의 index.jsx

```jsx
import ReactDOM from 'react-dom/client';

import App from './App.jsx';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

앱 컴포넌트가 루트라는 html요소로 ReactDom을 이용하여 렌더링 됨.

프로젝트 안의 index.html

```jsx
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/investment-calculator-logo.png" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React Investment Calculator</title>
  </head>
  <body>
    <div id="root"></div> // 여기부분 루트 연결
    <script type="module" src="/src/index.jsx"></script>
  </body>
</html>
```

<br/>

## 사용자 입력 컴포넌트

UserInput.jsx

```jsx
export default function UserInput() {
  return (
    <section id="user-input">
      <div className="input-group">
        <p>
          <label>Initial Investment</label>
          <input
            type="number"
            required
          />
        </p>
        <p>
          <label>Annual Investment</label>
          <input
            type="number"
            required
          />
        </p>
      </div>
      <div className="input-group">
        <p>
          <label>Expected Return</label>
          <input
            type="number"
            required
          />
        </p>
        <p>
          <label>Duration</label>
          <input
            type="number"
            required
          />
        </p>
      </div>
    </section>
  );
}
```

양방향 바인딩 필요.

<br/>

## 이벤트 핸들링 & 양방향 바인딩 활용

유저가 입력한 값을 얻어 저장.

UserInput.jsx

```jsx
import { useState } from 'react';

export default function UserInput() {

  //네 가지 값들을 하나의 객체로 합쳐서 관리.
  const [userInput, setUserInput] = useState({
    initialInvestment: 10000,
    annualInvestment: 1200,
    expectedReturn: 6,
    duration: 10,
  });

  function handleChange(inputIdentifier, newValue) {
    setUserInput((prevUserInput) => {
      return {
        ...prevUserInput,
        [inputIdentifier]: newValue,
      };
    });
  }

  return (
    <section id="user-input">
      <div className="input-group">
        <p>
          <label>Initial Investment</label>
          <input
            type="number"
            required
	    // 인풋에 보이는 값
            value={userInput.initialInvestment}
 	    // target == input 임 target.value는 input의 value와 같음
            onChange={(event) =>
              handleChange('initialInvestment', event.target.value)
            }
          />
        </p>
        <p>
          <label>Annual Investment</label>
          <input
            type="number"
            required
            value={userInput.annualInvestment}
            onChange={(event) =>
              handleChange('annualInvestment', event.target.value)
            }
          />
        </p>
      </div>
      <div className="input-group">
        <p>
          <label>Expected Return</label>
          <input
            type="number"
            required
            value={userInput.expectedReturn}
            onChange={(event) =>
              handleChange('expectedReturn', event.target.value)
            }
          />
        </p>
        <p>
          <label>Duration</label>
          <input
            type="number"
            required
            value={userInput.duration}
            onChange={(event) =>
              handleChange('duration', event.target.value)
            }
          />
        </p>
      </div>
    </section>
  );
}
```

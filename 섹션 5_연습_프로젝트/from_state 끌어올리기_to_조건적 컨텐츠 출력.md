### State 끌어올리기

- UserInput.jsx의 userInput Object를 App.jsx로 끌어올려서 Investment.js 등 다른 컴포넌트로 전달해야함

```jsx
// App.jsx
import { useState } from "react";

import Header from "./components/Header.jsx";
import UserInput from "./components/UserInput.jsx";
import Result from "./components/Result.jsx";

function App() {
	// Result.jsx의 state, function 을 끌어옴
  const [userInput, setUserInput] = useState({
    initialInvestment: 10000,
    annualInvestment: 12000,
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
    <>
      <Header />
			</** userInput, Result 에 prop 전달 **/>
      <UserInput onChange={handleChange} userInput={userInput}/>
      <Result input={userInput}/>
    </>
  );
}

export default App;
```

### 값 계산하기, 숫자 올바르기 다루는 법

- 함수에 변수를 전달할 때 함수가 사용하는 변수명에 맞게 전달할 것
- 값을 input에서 여러 개를 따로 가져오면 JS에서는 input.value를 string으로 처리를 해버림
    - 1 + 1 = 11
- 문자열 → 숫자 변환

```jsx
function handleChange(inputIdentifier, newValue) {
    setUserInput((prevUserInput) => {
      return {
        ...prevUserInput,
				// newValue -> +newValue
        [inputIdentifier]: +newValue,
      };
    });
  }
```

### 리스트에 결과 출력 및 파생값 만들기

```jsx
import {calculateInvestResult, formatter} from '../utils/investment'

export default function Result({ input }) {
    const resultData = calculateInvestResult(input);
    const initialInvestment = resultData[0].valueEndOfYear - resultData[0].interest - resultData[0].annualInvestment;

    return <table id='result'>
        <thread>
            <tr>
                <th>Year</th>
                <th>Investment Value</th>
                <th>Interest (Year)</th>
                <th>Total Interest</th>
                <th>Invested Capital</th>
            </tr>
        </thread>
        <tbody>
            {resultData.map(yearData => {
                const totalInterest = yearData.valueEndOfYear - yearData.annualInvestment * yearData.year;
                const  totalAmountInvested = yearData.valueEndOfYear - totalInterest;
                return <tr key={yearData.year}>
                    <td>{yearData.year}</td>
                    <td>{formatter.format(yearData.valueEndOfYear)}</td>
                    <td>{formatter.format(yearData.interest)}</td>
                    <td>{formatter.format(totalInterest)}</td>
                    <td>{formatter.format(totalAmountInvested)}</td>
                </tr>
            })}
        </tbody>
    </table>
}
```

### 조건적 컨텐츠 출력

- duration에 음수값이 들어가거나, ratio = 0 .. 등등 시 investment 함수가 에러
    - 대신 에러 메시지 출력
    - valid input이 있어야만 결과값이 나오도록 필터링

```jsx
// App.jsx
const inputIsValid = userInput.duration >= 1 && ratio != 0;

{inputIsValid && <Result input={userInput} />}
{!input && <p>Please enter a duration greater than zero</p>}
```

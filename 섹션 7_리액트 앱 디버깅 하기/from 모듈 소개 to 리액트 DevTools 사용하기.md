## 모듈 소개

디버깅 하는 법을 배울 것.

- 리액트 에러 메세지의 이해
- 메세지에 나타나지 않는 오류(논리적 오류) 해결하기
- 리액트 Strict Mode 알아보기
- React DevTools 사용해 보기

<br/>

## 리액트 오류 메세지 이해하기

![111](https://github.com/sujinann/ReactStudy/assets/139312979/91801a17-7c0e-4e57-9cac-d94f839b9249)

오류 내용, 오류 위치, 오류로 이어지는 코드의 실행 목록들 제공 stack-trace

Results.jsx

```jsx
import { calculateInvestmentResults, formatter } from '../util/investment.js';

export default function Results({ input }) {
  const results = [];
  calculateInvestmentResults(input, results); //여기 함수를 볼 필요가 있음
  // 따라서 밑에 언급할 로직 추가 필요
  const initialInvestment =
    results[0].valueEndOfYear -
    results[0].interest -
    results[0].annualInvestment;

  return (
    <table id="result">
      <thead>
        <tr>
          <th>Year</th>
          <th>Investment Value</th>
          <th>Interest (Year)</th>
          <th>Total Interest</th>
          <th>Invested Capital</th>
        </tr>
      </thead>
```

investment.jsx

```jsx
// This function expects a JS object as an argument
// The object should contain the following properties
// - initialInvestment: The initial investment amount
// - annualInvestment: The amount invested every year
// - expectedReturn: The expected (annual) rate of return
// - duration: The investment duration (time frame)
export function calculateInvestmentResults({
  initialInvestment,
  annualInvestment,
  expectedReturn,
  duration,
}, results) {
  let investmentValue = initialInvestment;

  for (let i = 0; i < duration; i++) { // 여기가 원인인 것을 알 수 있음
    const interestEarnedInYear = investmentValue * (expectedReturn / 100);
    investmentValue += interestEarnedInYear + annualInvestment;
    results.push({
      year: i + 1, // year identifier
      interest: interestEarnedInYear, // the amount of interest earned in this year
      valueEndOfYear: investmentValue, // investment value at end of year
      annualInvestment: annualInvestment, // investment added in this year
    });
  }
}
```

```jsx
if (results.length === 0) {
  return <p>Invalid input data provided.</p>
};
```

Results.jsx 에 위 코드를 추가하는 것으로 해결.

<br/>

## 코드 흐름 및 경고 분석

![22222.PNG](https://prod-files-secure.s3.us-west-2.amazonaws.com/ca254afe-fecd-4396-92df-1ab2cb1215ad/dd9978c4-cef5-4256-b30c-4dac86339874/22222.png)

오류 메세지가 출력 되지 않는 논리적 오류 등과 같은 경우에는

인터넷 브라우저의 개발자 도구 옵션의 소스 탭에서 도움을 얻을 수 있음

코드를 클릭하면 파란 부분으로 변하고 그 부분의 로직에서 동작을 멈추고

아래 버튼들과 마우스를 변수에 올림으로 prop 분석을 가능하게 함

![333.PNG](https://prod-files-secure.s3.us-west-2.amazonaws.com/ca254afe-fecd-4396-92df-1ab2cb1215ad/2c2cf80a-c7ad-4b3e-a4d6-d52c681e29e8/333.png)

19번 째 줄 newValue 앞에 + 를 붙여 숫자로 변환하는 과정을 거쳐 오류를 해결 할 수 있음

<br/>

## React Strict Mode 이해하기

index.jsx

```jsx
import { StrictMode } from 'react'; // 스트릭트 모드 추가
import ReactDOM from 'react-dom/client';

import App from './App.jsx';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <StrictMode> // 스트릭트 모드 적용
    <App />
  </StrictMode>
);
```

Strict Mode 는 리액트의 내장 컴포넌트

위의 예시처럼 앱에 사용하면 모든 컴포넌트를 두 번씩 실행

부분적으로 적용 가능

개발 단계에서만 두 번씩 실행 배포 때는 성능 저하 고려

컴포넌트들이 두 번씩 실행되면 즉각적인 에러 발견에 용이해서 사용

<br/>

## React DevTools 사용하기

확장 프로그램의 하나

개발자 도구의 Components 탭에서 이용 가능 

![4444.PNG](https://prod-files-secure.s3.us-west-2.amazonaws.com/ca254afe-fecd-4396-92df-1ab2cb1215ad/30aba75a-d586-406b-bbd7-b133e422a1dc/4444.png)

컴포넌트를 클릭하면 컴포넌트의 내용들에 대해 자세히 알 수 있음

받는 prop 값들과 그 prop들의 속성 등을 화면과 같이 표시해 줌

저기서 변화를 주면 고친 prop이 어떤 UI에 어떤 변화를 끼치는지 즉시 확인 가능

만약 컴포넌트가 상태 관리를 하고 있다면 hooks 에서 자세한 정보를 얻을 수 있음

또한 컴포넌트 트리 구조 확인에도 용이함

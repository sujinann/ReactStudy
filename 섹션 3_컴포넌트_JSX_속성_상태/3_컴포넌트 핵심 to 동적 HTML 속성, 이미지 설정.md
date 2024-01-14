# 컴포넌트 핵심 ~ 동적 HTML 속성, 이미지 설정
<br/>

## 소개

2주차는 Components, Data, Events, State 내용을 학습하여  

정적인 리액트 앱과 동적인 리액트 앱을 생성하는 것에 대해 배움

<br/>

## 컴포넌트 핵심 개념

- UI 구성 단위
- HTML, CSS, JavaScript 등 언어의 결합으로 React Component 구성
- 장점 - 코드 재활용 높아짐, 보기 편해짐, 수정 편해짐, 관심사의 분리

<br/>

## JSX

- JacaScript Syntax Extention
- 자바스크립트 언어에 다른 언어를 섞어 사용
- 브라우저에서 사용 불가능, 개발 서버에서 변환 후 전달

<br/>

## 커스텀 컴포넌트 생성 및 활용

```jsx
function Header() {
  return (
    <header>
      <img src="src/assets/react-core-concepts.png" alt="Stylized atom" />
      <h1>React Essentials</h1>
      <p>
        Fundamental React concepts you will need for almost any app you are going to build!
      </p>
    </header>
  );
}

function App() {
  return (
    <div>
      <Header />
      <main>
        <h2>Time to get started!</h2>
      </main>
    </div>
  );
}

export default App;
```

커스텀 컴포넌트 명은 항상 첫글자 대문자로 사용. 내장 요소와 중복 방지.

<br/>

## 동적 값 출력 및 활용

```jsx
const reactDescriptions = ['Fundamental', 'Crucial', 'Core'];

function genRandomInt(max) {
  return Math.floor(Math.random() * (max + 1));
}

function Header() {
  return (
    <header>
      <img src="src/assets/react-core-concepts.png" alt="Stylized atom" />
      <h1>React Essentials</h1>
      <p>
        {reactDescriptions[genRandomInt(2)]} Fundamental React concepts you will need for almost any app you are going to build!
      </p>
    </header>
  );
}

function App() {
  return (
    <div>
      <Header />
      <main>
        <h2>Time to get started!</h2>
      </main>
    </div>
  );
}

export default App;
```

13번째 줄 {reactDescriptions[genRandomInt(2)]} 가 핵심

const discription = {reactDescriptions[genRandomInt(2)]} 로 밖으로 빼서 정리하면 더 깔끔

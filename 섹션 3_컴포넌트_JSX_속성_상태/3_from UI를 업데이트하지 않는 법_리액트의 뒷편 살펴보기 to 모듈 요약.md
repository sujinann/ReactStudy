# UI를 업데이트하지 않는 법 - 리액트의 뒷편 살펴보기 [핵심 개념] ~ 모듈 요약

### UI를 업데이트하지 않는 법 - 리액트의 뒷편 살펴보기 [핵심 개념]

- App.jsx에 새로운 일반 변수를 선언
    
    ```jsx
     	// App() 함수 안에 있음
      // tabContent라는 일반 변수를 선언해보자
      let tabContent = "Please click a button"; // 버튼을 눌러도 문구가 바뀌지 않음
    
      function handleSelect(selectedButton) {
        // selectedButton => 'Components', 'JSX', 'Props', 'State'
        // console.log(selectedButton);
        tabContent = selectedButton;
        console.log(tabContent); // 콘솔에 정상적으로 버튼명이 출력됨
      }
    
      console.log("App Component Rendering"); // 실행 시 1번 출력
    ```
    
    ```jsx
     // App()의 return 안에 있는 section	
     <section id="examples">
        <h2>Examples</h2>
        <menu>
          <TabButton onSelect={() => handleSelect("components")}>
          Components
          </TabButton>
          <TabButton onSelect={() => handleSelect("jsx")}>JSX</TabButton>
          <TabButton onSelect={() => handleSelect("props")}>Props</TabButton>
          <TabButton onSelect={() => handleSelect("state")}>State</TabButton>
        </menu>
        {tabContent} // 재평가되지 않아서 맨 처음 값으로만 유지됨
      </section>
    ```
    
    - handleSelect 함수는 실행됨 ⇒ 변수는 업데이트되지만, UI는 업데이트되지 않음
    - App 컴포넌트 함수가 재실행되고 있지 않기 때문 ⇒ tabContent의 변화를 감지할 수 없음
    - 리액트에 의해 코드가 재평가되어야 UI가 업데이트될 수 있음
    - 컴포넌트 함수는 리액트가 단 한 번만 실행한다!
    
    ```jsx
    // TabButton.jsx
    
    export default function TabButton({ children, onSelect }) {
    	console.log("Tabbutton Component Executing"); // 버튼이 4개 = 실행 시 4번 출력
      return (
        <li>
          <button onClick={onSelect}>{children}</button>
        </li>
      );
    }
    ```
    
    - App Component Rendering은 한 번만 출력, 
    Tabbutton Component Executing은 버튼 수만큼 출력
    
    ![Untitled](https://github.com/serethia/git-test/assets/137035446/43bd1f41-422c-48c4-ace3-fd4cbe431f73)
    
    - 위의 두 로그 모두 버튼을 눌러봐도 출력되지 않음 = 일반 변수를 선언했기 때문에 재실행 X
    
- 리액트에게 “앱 컴포넌트 함수를 재실행하라”고 전달하기 위해서는 State를 활용!

### State(상태) 관리 & Hooks(훅) 사용법 [핵심 개념]

- useState 함수를 리액트에서 가져오기 ⇒ import문
    
    ```jsx
    // App.jsx
    import { CORE_CONCEPTS } from "./data.js";
    import Header from "./components/Header/Header.jsx";
    import CoreConcept from "./components/CoreConcept.jsx";
    import TabButton from "./components/TabButton.jsx";
    import { useState } from "react"; // state를 사용하기 위해 임포트할 것!
    ```
    
    - React Hooks = 리액트 프로젝트에서 use로 시작하는 모든 함수의 총칭
        - 리액트 컴포넌트 함수 or 다른 리액트 훅 안에서만 호출 가능
        
        ```jsx
        // ( O )
        // 리액트 컴포넌트 함수 안 최상위에서 리액트 훅 호출
        function App() {
          useState("Please click a button"); // 괄호 안에 적힌 값 = 초기(default)값
        
          function handleSelect(selectedButton) {
            tabContent = selectedButton;
            console.log(tabContent);
          }
        
        	// (이하 코드 생략)
        ```
        
        - 다른 코드 안에 중첩시켜 호출해서는 안 됨
        
        ```jsx
        // ( X )
        function App() {
        
          function handleSelect(selectedButton) {
        	  // 호출 시 내부 함수 안에 중첩하면 안 됨
        	  useState("Please click a button");
            tabContent = selectedButton;
            console.log(tabContent);
          }
        ```
        
    - 훅의 규칙:
        - 컴포넌트 함수 (혹은 커스텀 훅) 안에서만 훅을 호출 = 밖에서는 X
        - 최상위에서만 훅을 호출 = 내부 코드 (함수, if문, loop 등) 안에 중첩 X
    - useState 함수를 변수에 저장 가능 (단, 항상 2칸짜리 배열 형태임)
        
        ```jsx
        // 첫 번째 요소: 해당 컴포넌트 실행 주기의 현 데이터 스냅샷
        // 두 번째 요소: state를 업데이트하는 함수 + 리액트가 함수를 재실행하도록 전달
        
        // 두 코드 다 같은 기능!
        const stateArray = useState("Please click a button");
        
        // 구조 분해 할당을 이용해서도 가능 (두 요소명은 임의로 정해도 무방)
        const [ selectedTopic, setSelectedTopic ] = useState("Please click a button");
        ```
        
        - 첫 번째 요소: 실행 직후에는 default 값이 저장됨, 재실행 시 업데이트된 값이 저장됨
        - 두 번째 요소: 첫 번째 요소를 계속 업데이트된 값으로 새로 저장시킴
                                  (그래서 굳이 변수를 쓸 필요가 없음)
            
- 다른 곳에서도 state 요소를 호출해 이용
    - 함수 실행 후에 콘솔 출력을 해도 업데이트 이전 값이 출력됨 ⇒ 리액트 작동 원리 이해해야!
    - 앱 컴포넌트 함수를 전부 실행하고 난 뒤에야 업데이트 후 값을 사용할 수 있게 됨
        
        (아직 앱 컴포넌트 함수 도중에 있는 상태라서 이전 값밖에 이용할 수 없는 것)
        
    
    ```jsx
      function handleSelect(selectedButton) {
        setSelectedTopic(selectedButton); // setSelectedTopic 함수에 원하는 값을 넣음
        console.log(selectedTopic); // 업데이트되기 이전 값이 콘솔에 출력
      }
    ```
    
    ```jsx
    <section id="examples">
      <h2>Examples</h2>
      <menu> // Tabbutton Component Executing은 그대로 4번씩 재출력됨
        <TabButton onSelect={() => handleSelect("components")}>
        Components
        </TabButton>
        <TabButton onSelect={() => handleSelect("jsx")}>JSX</TabButton>
        <TabButton onSelect={() => handleSelect("props")}>Props</TabButton>
        <TabButton onSelect={() => handleSelect("state")}>State</TabButton>
      </menu>
      {selectedTopic} // selectedTopic 값(버튼명)으로 화면에 정상 출력됨
    </section>
    ```
    

### 데이터 기반 State(상태) 가져오기 및 출력

- 변경된 data.js로 출력
    - 변경 부분: EXAMPLES 상수 추가됨
    
    ```jsx
    // data.js에서 추가된 부분
    export const EXAMPLES = {
      components: {
        title: "Components",
        description:
          "Components are the building blocks of React applications. A component is a self-contained module (HTML + optional CSS + JS) that renders some output.",
        code: `
    function Welcome() {
      return <h1>Hello, World!</h1>;
    }`
      },
      jsx: {
        title: "JSX",
        description:
          "JSX is a syntax extension to JavaScript. It is similar to a template language, but it has full power of JavaScript (e.g., it may output dynamic content).",
        code: `
    <div>
      <h1>Welcome {userName}</h1>
      <p>Time to learn React!</p>
    </div>`
      },
      props: {
        title: "Props",
        description:
          "Components accept arbitrary inputs called props. They are like function arguments.",
        code: `
    function Welcome(props) {
      return <h1>Hello, {props.name}</h1>;
    }`
      },
      state: {
        title: "State",
        description:
          "State allows React components to change their output over time in response to user actions, network responses, and anything else.",
        code: `
    function Counter() {
      const [isVisible, setIsVisible] = useState(false);
    
      function handleClick() {
        setIsVisible(true);
      }
    
      return (
        <div>
          <button onClick={handleClick}>Show Details</button>
          {isVisible && <p>Amazing details!</p>}
        </div>
      );
    }`
      }
    };
    ```
    
    - EXAMPLES 상수를 import
    
    ```jsx
    import { EXAMPLES } from "./data.js";
    ```
    
    - 각 속성(components, jsx, props, state)을 가져와 화면에 출력하고 싶은 것을 . 이후에 작성
    
    ```jsx
    <div id="tab-content">
      <h3>{EXAMPLES[selectedTopic].title}</h3>
      <p>{EXAMPLES[selectedTopic].description}</p>
      <pre>
        <code>{EXAMPLES[selectedTopic].code}</code>
      </pre>
    </div>
    ```
    
    - default 값을 알맞은 형식(data.js에서 찾을 수 있는 속성)으로 수정
    
    ```jsx
    const [selectedTopic, setSelectedTopic] = useState("components");
    ```
    

### 퀴즈 4: State(상태)와 계산된 값

![정답1 PNG](https://github.com/serethia/git-test/assets/137035446/10122abd-5b6e-4ed0-808f-8939e491e043)

![정답2 PNG](https://github.com/serethia/git-test/assets/137035446/b165e8f9-86ef-4daf-80c2-f1fb373890ad)

![정답3 PNG](https://github.com/serethia/git-test/assets/137035446/1ab26268-6c62-4948-9c0c-a666678f9b7d)

![정답4 PNG](https://github.com/serethia/git-test/assets/137035446/803819b4-011f-4775-b54a-c59473dc27f7)

- 채점 결과
    
    ![채점결과 PNG](https://github.com/serethia/git-test/assets/137035446/5a49bbe3-bf4d-4e72-8ee3-d96237abbcbb)
    
    - 4번 문제가 잘 이해가 안 가서 추가 검색을 해봤습니다
        
        ![Untitled (1)](https://github.com/serethia/git-test/assets/137035446/8f1b07ad-5f6c-491c-a1c4-b9ac32322fd0)
        
        - 파생된 state = 계산된 값 
        = 이전에 정의된 다른 state의 영향을 받아서 해당 state을 통해 계산할 수 있는 값
        - 오직 데이터와의 동기화만을 위해 불필요한 useState를 남용하지 말라는 의미
        = state에 저장할 변수의 수를 줄여 최적화 및 단순화하라는 뜻
        - 예시:
            - 노래 장르라는 대분류가 있고, 각 장르 안에 곡들이 소분류로 있다고 하자
            - 소분류가 모두 선택되었을 경우 대분류 체크박스도 체크되도록 한다
            - 반대로 대분류 체크박스를 체크하면 소분류가 모두 선택되도록 한다
            
              ⇒ 대분류와 소분류가 서로 영향을 주고받는 경우, useState를 남용하지 않는다!
            

### 조건적 콘텐츠 렌더링

- 화면에 출력할 default값을 ‘Please select a topic’으로 변경하고 싶다면
    - useState 안의 default값을 빈 String 값 혹은 null로 바꿔준다
    
    ```jsx
    const [selectedTopic, setSelectedTopic] = useState("");
    // 혹은
    const [selectedTopic, setSelectedTopic] = useState(null);
    // 혹은
    const [selectedTopic, setSelectedTopic] = useState();
    ```
    
    - 조건적인 출력으로 변경
        - 동적 내용을 넣기 위한 중괄호 { } 작성
        - 삼항(조건) 연산자 활용 (null은 아무것도 출력하지 않음)
        
        ```jsx
        // 선택한 버튼이 없다면 Please select a topic. 출력        
        {selectedTopic === undefined ? <p>Please select a topic.</p> : null} 
        // 혹은
        {!selectedTopic ? <p>Please select a topic.</p> : null} 
        ```
        
        ```jsx
        // 선택한 버튼의 내용을 알맞은 형식으로 출력        
        {selectedTopic !== undefined ? <div id="tab-content">
        			             <h3>{EXAMPLES[selectedTopic].title}</h3>
        			             <p>{EXAMPLES[selectedTopic].description}</p>
        			             <pre>
        			               <code>{EXAMPLES[selectedTopic].code}</code>
        			             </pre>
        			           </div> : null}
        // 혹은
        {selectedTopic ? <div id="tab-content">
        			             <h3>{EXAMPLES[selectedTopic].title}</h3>
        			             <p>{EXAMPLES[selectedTopic].description}</p>
        			             <pre>
        			               <code>{EXAMPLES[selectedTopic].code}</code>
        			             </pre>
        			           </div> : null}
        ```
        
        - 조건이 서로 정반대인 경우 null을 빼고 두 삼항연산자 조건을 묶어도 됨
        
        ```jsx
        {!selectedTopic ? (<p>Please select a topic.</p>) : (<div id="tab-content">
        					         <h3>{EXAMPLES[selectedTopic].title}</h3>
        					         <p>{EXAMPLES[selectedTopic].description}</p>
        					         <pre>
        					           <code>{EXAMPLES[selectedTopic].code}</code>
        					         </pre>
        					       </div>})
        ```
        
        - 삼항연산자를 묶지 않는 대신, 삼항연산자와 else문을 지우고 && (AND) 로 대체해도 됨
        (자바스크립트에서 조건이 true일 경우, AND 뒤의 값을 출력한다는 꼼수를 이용한 것)
        
        ```jsx
        {!selectedTopic && <p>Please select a topic.</p>}
        
        {selectedTopic && <div id="tab-content">
        		<h3>{EXAMPLES[selectedTopic].title}</h3>
        		<p>{EXAMPLES[selectedTopic].description}</p>
        		<pre>
        		  <code>{EXAMPLES[selectedTopic].code}</code>
        		</pre>
        	      </div>}
        ```
        
        - 변수를 사용해서 JSX 코드를 저장해도 됨
        (단, 변수에 state를 저장하면 오류가 났었음!)
        
        ```jsx
        // return문 위에 변수 선언
        let tabContent = <p>Please select a topic.</p>;
        if (selectedTopic) { // 조건 만족 시 값을 덮어씌운다
          tabContent = (
            <div id="tab-content">
              <h3>{EXAMPLES[selectedTopic].title}</h3>
              <p>{EXAMPLES[selectedTopic].description}</p>
              <pre>
                <code>{EXAMPLES[selectedTopic].code}</code>
              </pre>
            </div>
          );
        }
        ```
        
        ```jsx
        // return문 안에서 출력
        <section id="examples">
          <h2>Examples</h2>
          <menu>
            <TabButton onSelect={() => handleSelect("components")}>Components</TabButton>
            <TabButton onSelect={() => handleSelect("jsx")}>JSX</TabButton>
            <TabButton onSelect={() => handleSelect("props")}>Props</TabButton>
            <TabButton onSelect={() => handleSelect("state")}>State</TabButton>
          </menu>
          {tabContent} // 변수 값 출력
        </section>
        ```
        

### CSS 스타일링 및 동적 스타일링

- 선택된 버튼(탭)에 효과를 부여하자
    - 리액트에서는 className이라는 prop으로 클래스 설정 가능 (class 아님! 혼동 주의)
    (index.css 파일에 있는 스타일링에 명시된 클래스명을 가져오는 것)
    
    ```jsx
    // TabButton.jsx
    return (
      <li>
        <button className="active" onClick={onSelect}>{children}</button>
      </li>
    );
    
    // 이렇게만 작성하면 모든 버튼이 활성화된 상태로 변함. 일단 스타일링은 가능하구나!
    ```
    
    ```css
    #examples menu button.active {
      background-color: #7925d3;
      color: #ebe7ef;
    }
    ```
    
    - 속성도 동적으로 부여할 수 있다
    
    ```jsx
    // TabButton.jsx
    export default function TabButton({ children, onSelect, isSelected }) {
      console.log("Tabbutton Component Executing");
      return (
        <li>
          <button className={isSelected ? 'active' : ''} onClick={onSelect}>
            {children}
          </button> // 삼항연산자에서 else문을 undefined로 적어도 ''와 동일!
        </li>
      );
    }
    ```
    
    ```jsx
    // App.jsx
    <TabButton isSelected={selectedTopic === "components"} // true 혹은 false로 덮어씀
               onSelect={() => handleSelect("components")}>
               Components
    </TabButton>
    <TabButton isSelected={selectedTopic === "jsx"}
               onSelect={() => handleSelect("jsx")}>
               JSX
    </TabButton>
    <TabButton isSelected={selectedTopic === "props"}
               onSelect={() => handleSelect("props")}>
               Props
    </TabButton>
    <TabButton isSelected={selectedTopic === "state"}
               onSelect={() => handleSelect("state")}>
               State
    </TabButton>
    
    // 두 코드를 수정해주면 누를 때만 버튼이 정상적으로 효과가 바뀌게 됨
    ```
    

### List(리스트) 데이터 동적 출력

- 기존 자바스크립트 객체 배열은 요소의 수를 통일해야 한다는 단점이 있음.
입력 개수가 다른 것이 있을 경우, 해당 요소가 뜨지 않아서 앱이 정상적으로 나오지 않게 됨
    
    ```jsx
    // 기존 자바스크립트 객체 배열
    <ul>
      <CoreConcept {...CORE_CONCEPTS[0]} />
      <CoreConcept {...CORE_CONCEPTS[1]} />
      <CoreConcept {...CORE_CONCEPTS[2]} />
      <CoreConcept {...CORE_CONCEPTS[3]} />
    </ul>
    ```
    
    ```jsx
    // data.js에서 state를 주석 처리해보자
    
    export const CORE_CONCEPTS = [
      {
        image: componentsImg,
        title: "Components",
        description:
          "The core UI building block - compose the user interface by combining multiple components."
      },
      {
        image: jsxImg,
        title: "JSX",
        description:
          "Return (potentially dynamic) HTML(ish) code to define the actual markup that will be rendered."
      },
      {
        image: propsImg,
        title: "Props",
        description:
          "Make components configurable (and therefore reusable) by passing input data to them."
      }
      // {
      //   image: stateImg,
      //   title: "State",
      //   description:
      //     "React-managed data which, when changed, causes the component to re-render & the UI to update."
      // }
    ];
    ```
    
    - JSX로 데이터 배열 동적 출력 가능!
    - 자바스크립트 객체 배열 ⇒ JSX 요소 배열로 전환 (map 메소드 이용)
    
    ```jsx
    // conceptItem은 이미 CORE_CONCEPTS 배열의 인덱스로 쓰이는 중
    <ul>
      {CORE_CONCEPTS.map((conceptItem) => (<CoreConcept {...conceptItem} />))}
    </ul>
    
    // 실행 시 주석 처리한 state는 아예 뜨지 않고 3개의 주제만 자연스럽게 화면에 출력됨
    ```
    
    - 개발자 도구에 Warning이 뜨는데 key에 대해서는 추후 학습할 예정
    
    > Warning: Each child in a list should have a unique "key" prop.
    > 
    > 
    > Check the render method of `App`. See [https://reactjs.org/link/warning-keys](https://reactjs.org/link/warning-keys) for more information.
    > 
    > at CoreConcept ([https://txfznm.sse.codesandbox.io/src/components/CoreConcept.jsx:18:3](https://txfznm.sse.codesandbox.io/src/components/CoreConcept.jsx:18:3))
    > 
    > at App ([https://txfznm.sse.codesandbox.io/src/App.jsx?t=1705144600707:26:45](https://txfznm.sse.codesandbox.io/src/App.jsx?t=1705144600707:26:45))
    > 
    
    ```jsx
    // key를 직접 쓰지 않더라도 꼭 작성해주자
    // key를 쓰는 것은 리액트
    <ul>
      {CORE_CONCEPTS.map((conceptItem) => (
       <CoreConcept key={conceptItem.title} {...conceptItem} />
      ))}
    </ul>
    
    // 리스트 안의 아이템들을 구분할 수 있는 고유값으로 key에 넣어준다
    // key를 작성해주면 Warning이 사라진다!
    ```
    

### 퀴즈 5: 조건적 콘텐츠와 동적인 리스트

![질문1-1 PNG](https://github.com/serethia/git-test/assets/137035446/f43dda0c-4042-487d-9425-a57e6284398b)

![질문1-2 PNG](https://github.com/serethia/git-test/assets/137035446/acdf77a6-0bde-4e7b-988e-25ab2b77c890)

![정답1-1 PNG](https://github.com/serethia/git-test/assets/137035446/28bf76a2-a106-4aba-bcdc-403fb370c814)

![정답1-2 PNG](https://github.com/serethia/git-test/assets/137035446/81b6ca81-0801-47ca-acd0-436d22cde38b)

![질문2 PNG](https://github.com/serethia/git-test/assets/137035446/8c6bfb02-72f1-4af6-92fb-552494459451)

![정답2 PNG](https://github.com/serethia/git-test/assets/137035446/e2bd2c73-c715-4883-adbe-cc32b37153a3)

![질문3 PNG](https://github.com/serethia/git-test/assets/137035446/a84232eb-74da-4653-970f-ae28768ea7b3)

![정답3 PNG](https://github.com/serethia/git-test/assets/137035446/2ac07bad-3a45-4b52-b547-bbc708a99dd1)

- 채점 결과
    
    ![채점결과 PNG](https://github.com/serethia/git-test/assets/137035446/b0ae7af4-a109-44dd-94a3-86098cbcd80d)

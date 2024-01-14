### Props 개념

- 컴포넌트는 여러 번 재사용하는 경우가 많음
- Props
    - 데이터를 jsx 문법에 맞게 컴포넌트로 전달 후 사용
    - 모든 종류의 데이터 값을 Props로 전달 가능
        - img, number, array, object 등
    - 

```jsx
import imageProps from "../img주"

function CoreConcept(props) {
	return (
	<li>
		<img src={props.image} alt={props.title}/>
		<h3>{props.title}</h3>
		<p>{props.desc}</p>
	</li>
	);
}

function App() {
	<main>
		<section id="core-concepts>
		<h2>Core Concepts</h2>
		<ul>
			<CoreConcept title="제목" desc="설명" image="imageProps"/>
			<CoreConcept title="제목2" ... />
			<CoreConcept />
			<CoreConcept />
		**</ul>
	</main>
};
```

### Props 대체 문법

```jsx
// 이미지 소스 JSX
import componentsImg from './assets/components.png';
import propsImg from './assets/config.png';
import jsxImg from './assets/jsx-ui.png';
import stateImg from './assets/state-mgmt.png';

export const CORE_CONCEPTS = [
  {
    image: componentsImg,
    title: 'Components',
    description:
      'The core UI building block - compose the user interface by combining multiple components.',
  },
  {
    image: jsxImg,
    title: 'JSX',
    description:
      'Return (potentially dynamic) HTML(ish) code to define the actual markup that will be rendered.',
  },
  {
    image: propsImg,
    title: 'Props',
    description:
      'Make components configurable (and therefore reusable) by passing input data to them.',
  },
  {
    image: stateImg,
    title: 'State',
    description:
      'React-managed data which, when changed, causes the component to re-render & the UI to update.',
  },
];
```

```jsx
import {CORE_CONCEPTS} from "../data.jsx"

function CoreConcept(props) {
	return (
	<li>
		<img src={props.image} alt={props.title}/>
		<h3>{props.title}</h3>
		<p>{props.desc}</p>
	</li>
	);
}

...
<CoreConcept 
title={CORE_CONCEPTS[0].title}
description={CORE_CONCEPTS[0].description}
image={CORE_CONCEPTS[0].image}
/>
...

// JS Spread 문법
<CoreConcept {..CORE_CONCEPTS[0]/}
/>
...

// JS destructuring 활용한 함수 수정
 function CoreConcept({image, title, description}) {
	return (
	<li>
		<img src={image} alt={title}/>
		<h3>{title}</h3>
		<p>{description}</p>
	</li>
	);
}
```

- 추천 실습 → CoreConcept function과 CoreConcept 활용 컴포넌트를 다른 파일에 분리하기
    - 컴포넌트 jsx 파일은 components 서브폴더에 모아놓기
        
        → export 키워드로 jsx 파일에서 컴포넌트 export 해주기
        
        ! component에서 import한 파일의 주소는 최상위 경로부터 상세히 정의하기
        

```jsx
// components/Header.jsx
// default 키워드 export면 import 시 키워드 지정 필요없음
// 기본 export 값으로 생성
import reactImg from "./assets/img.png"

export default function Header() { 
	return {
		<header>
			<img src={reactImg}>
		
		}

}

//App.jsx
import Header from "../components/Header.jsx"

function App() {
	return {
		<div>
			<Header />
		</div>
	}
}

```

- 컴포넌트 스타일 파일 저장
    - .css 파일 별도 저장
    - jsx 파일에서 css 파일 import
    - jsx와 css를 components 밑에 하위 폴더로 묶어서 관리 가능
        - jsx에서 import한 파일 단계가 한단계씩 올라감을 유의
        
        ex: import “../MyComponent.jsx” → “../../MyComponent.jsx” 
        
![Untitled (1)](https://github.com/chomchom96/ReactStudy/assets/112466460/54f663a4-1cda-4f47-b1db-9417e4f899c6)


- Children Prop
    - 컴포넌트 context 사이 출력할 텍스트를 정의
    - 필요에 따라 복잡한 jsx 구조가 들어갈 수도 있음 (나중)
    - 이런 식으로 컴포넌트에서 다른 컴포넌트의 내용을 감싸서 구축하는 것 컴포넌트 합성

```jsx

...
<section id="example">
	<h2>Examples</h2>
	<menu>
		<TabButton>BUTTON</TabButton> // props.children은 컴포넌트 사이 출력할 텍스트를 정의
	</menu>
</section>

// TabButton.jsx
export default function tabButton(props) {
		return <li><button>{props.children}</button></li>; // 리액트가 지정한 children
}

// or
export default function tabButton({children}) {
		return <li><button>{children}</button></li>; // 리액트가 지정한 children
}
```

- 이벤트 처리하기
    - 리액트에서 JS식 DOM  접근자를 직접 사용하면 안됨!
    - 이벤트 처리용 prop 추가(onClick 등)
        - prop에 이벤트 발생시 처리할 함수를 값으로 전달

```jsx
export default function tabButton({children}) {
		function handleClick = () => {
			console.log("click")
		}

		return {
			<li>
				<button onClick={handleClick}>{children}</button>
			</li>;
		}
}
```

- 함수를 Prop의 값으로 전달하기
    - 커스텀 컴포넌트는 Native JSX 요소를 감쌈
    - 리액트에서 제공하는 내장 컴포넌트
    - 내장 버튼은 onClick과 같은 내장 props를 가짐
    - 외부에서 버튼에 들어갈 함수를 Props로 전달할 수 있음
    - 최종적으로 TabButton 컴포넌트의 내부까지 외부 함수가 전달되어야 함(Forwarding Pointer)

```jsx
function handleSelect = () => {
	console.log("Click");
}

<TabButton onSelect={handleSelect}>Button</TabButton> // onClick시 발생할 함수를 Prop 전달하고 싶음
```

```jsx
// destructuring 문법으로 해당 prop에 접근하기
// children prop은 리액트에서 제공하는 기본 prop으로 변경 불가
// 다른 prop은 사용자 커스텀 가능
export default function tabButton({children, onSelect}) {
		
		return {
			<li>
				<button onClick={onSelect}>{children}</button>
			</li>;
		}
}
```

- 이벤트 함수에 커스텀 인자 전달
    - 동적 Content를 버튼 클릭에 따라 변경하고 싶음
    - handleSelect 함수에 다른 식별자를 전달하도록 onSelect에 커스텀 함수 전달
    - 이벤트에 따라 실행되는 함수를 정의 + 함수의 인자를 통제할 때 사용할 수 있는 패턴

```jsx
function handleSelect = (selectedButton) => {
	// selectedButton에 따라 문자열 값을 동적 변화
	console.log(selectedButton)
}

<TabButton onSelect={() => handleSelect('components')}>Button</TabButton> // onClick시 발생할 함수를 Prop 전달하고 싶음
```

### 퀴즈 2 : Dynamic & Props

1. JSX 내에 동적인 값 또는 JS 표현 출력을 위한 문법 : 중괄호 {}
2. JSX 내에서 동적으로 출력 가능한 것은? : 유효한 자바스크립트 표현문
3. 리액트에서 이미지를 로딩할 때 주로 사용하는 방법 : <img>src를 import 문으로 생성된 path로 지정
4. HTML 요소에 동적인 값을 부여하는 방법 : JSX과 같이 중괄호
5. 리액트 컴포넌트 재사용은 Prop(속성)
6. Prop의 동작 방식 : Prop을 컴포넌트에 지정, 전달 받는 컴포넌트에서 추출(import)
7. 에러 나는 코드

```jsx
<MyComponent priority={5}/>
function MyComponent(priority) {
	return <p>Priority: {priority}</p>
}

=> Decomposing을 하지 않으면 props 객체에서 priority prop을 추출할 수 없음

수정 코드
function MyComponent(props) {
	return <p>Priority: {props.priority}</p>
}

OR

function MyComponent(priority) {
	return <p>Priority: ({priority})</p>
}
```

### 퀴즈 3 : 이벤트 핸들링 실습

1. 컴포넌트 함수를 주로 저장하는 방법 : 한 파일당 한 컴포넌트!
2. children Prop의 목적 : HTML 요소 열림과 닫힘 태그 사이에 있는 내용을 전달 및 사용 → Vue의 emit Prop과 성격이 완전히 다름
3. 사용자 이벤트 핸들링 방법 : 내장 prop 사용(onClick 등) → onClick(’’) 과 같이 함수 실행문을 사용하지 말고 함수 이름만 사용해서 해당 함수에 포인터를 전달할 것
4. 이벤트 이후 코드를 실행시킬 때 onClick 등 이벤트 prop에 전달해야 하는 값 → 실행되어야 하는 함수의 포인터 (onClilck = {handleClick})
5. 이벤트에서 독립적인 함수의 구성 및 설정법 → 함수를 다른 함수로 감싼다(onClick={() ⇒ handleClick(5)}). 함수의 실행을 다른 함수로 감싸면 그 다른 함수가 이벤트 핸들링의 prop 값으로 전달


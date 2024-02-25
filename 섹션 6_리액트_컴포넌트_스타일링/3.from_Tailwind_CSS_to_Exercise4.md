# 6주차 6강

### Tailwind CSS 소개

- 매우매우 인기있는 웹 앱 스타일링 솔루션
- 클래스에 스타일 지정
- 공식 docs 따라 설치할 것

[Install Tailwind CSS with Vite - Tailwind CSS](https://tailwindcss.com/docs/guides/vite)

- tailwind.config.js 파일은 직접 설치해야함
- ++ Tailwlind Intellisense
- 기존의 css 파일 + styled components를 제거해도 스타일 적용 가능

```jsx
import logo from '../assets/logo.png';

export default function Header() {
  return (
    <header className="flex flex-col items-center mt-8 mb-16">
      <img src={logo} alt="A canvas" className="mb-8 w-44 object-contain"/>
      <h1 className='text-4xl font-semibold tracking-widest text-center uppercase text-amber-800 font-title'>ReactArt</h1>
      <p>A community of artists and art-lovers.</p>
    </header>
  );
}

```

![Untitled](https://github.com/chomchom96/ReactStudy/assets/112466460/ceb14cad-f059-4cfd-af61-210a2b47d0dd)


### 리액트에서 Tailwind 추가 및 사용법

- 기존 index.css 위에 @tailwind 설정 세 줄만 추가해도 정상 작동
- tailwind.config.js에서 다양한 설정을 할 수 있음

```jsx

<h1 className='.. font-title'>ReactArt</h1>
 
// tailwind.config
/** @type {import('tailwindcss').Config} */
export default {
    ...
    theme: {
      extend: {
        fontFamily: {
          title: ["'Pacifico'", 'cursive'] // title font custom
        }
      },
    },
  }
```
![Untitled 1](https://github.com/chomchom96/ReactStudy/assets/112466460/df006be9-af52-4494-90f6-ab01a85864fc)

### 미디어 쿼리 + 가상 선택자

- 스크린이 특정 너비일 때만 추가할 수 있는 접두사가 있음
    - ex : md:w-40 → screen이 medium일 때만 w-40
- hover, focus 시 비슷하게 지정 가능
    - ex: hover:bg-white : hover 시 background-white
    
    ```jsx
    export default function Button({children}) {
        return <button className='px-4 py-2 font-semibold uppercase rounded text-stone-900 bg-amber-400 hover:bg-amber-500 ' {...props}>{children}</button>
    }
    ```


![Untitled 2](https://github.com/chomchom96/ReactStudy/assets/112466460/3b287491-2084-4586-beec-15c672cf0e6c)


### 조건 및 동적 스타일링

```jsx
export default function Input({ label, invalid, ...props }) {
    let labelClasses = "block mb-2 text-xs font-bold tracking-wide uppercase"
    let inputClasses = "w-full px-3 py-2 leading-tight text-gray-700 border rounded shadow"
    if (invalid) {
        labelClasses += "text-red-400"
        inputClasses += 'text-red-500 bg-red-100 border-red-300'
    }

    else {
        labelClasses +=  "text-stone-300"
        inputClasses += 'text-gray-700 bg-stone-300'
    }
    return (
      <p>
        <label className={labelClasses}>{label}</label>
        <input className={inputClasses} {...props} />
      </p>
    );
}
```

### Tailwind로 데모 앱 옮기기

```
<div id="auth-inputs" className="w-full max-w-sm p-8 rounded shadow-md bg-gradient-to-b from-stone-700 to-stone-800 mx-auto">
```

- w-full : 가능한 한 최대 width
- max-w-sm : 최대 24rem
- bg-gradient-to-b : top → bottom gradient  + from ~ to color 지정
- mx-auto : 가운데 정렬

```jsx
<div className="flex flex-col gap-2">
<div className="flex justify-end gap-4">
```

- flex flex-col : column 방향 flex
- gap-2 : 2 rem gap
- justify-end : 양끝부터 시작하는 flex 배치

### Tailwind 장단점

- 클래스 네임이 매우 길어짐
- JSX 코드 내에서 직접 바꿈 → 스타일 코드가 분리 안됨
- 재사용하는 컴포넌트 내에 스타일을 직접 지정 가능 → 재사용성 높음
- 코드 길이가 짧아짐 → 시간절약
    - intellisense로 필요한 스타일도 추천
    - 같은 css 스타일이 겹치지 않음
    - 커스텀 자유도 높음
    

### Coding Exercise 4

```jsx
import React from 'react';

// don't change the Component name "App"
export default function App() {
    const [toggle, setToggle] = React.useState(false);
    const customStyle = toggle? { color: 'red' } : { color: 'white' };
    const handleToggle = () => setToggle(!toggle);
    return (
        <div>
            <p style={{ ...customStyle }}>Style me!</p>
            <button onClick={handleToggle}>Toggle style</button>
        </div>
    );
}

```

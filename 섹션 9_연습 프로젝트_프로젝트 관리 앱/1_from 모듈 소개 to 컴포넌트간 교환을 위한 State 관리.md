## 모듈 소개

- 프로젝트 관리 앱 만들기
    - 컴포넌트와 상태 관리
    - ref와 portal 사용
    - 테일윈드

<br/>

## 프로젝트 사이드바 컴포넌트 추가하기

![sidebar](https://github.com/sujinann/ReactStudy/assets/139312979/9e2670ab-d625-442b-93f3-54d76cedff10)

- 프로젝트 추가 버튼, 프로젝트 전환 가능 리스트 기능의 사이드 바

<br/>

## 테일윈드로 사이드바 & 버튼 스타일링

```jsx
// App.jsx

import ProjectsSidebar from './components/ProjectsSidebar.jsx';

function App() {
  return (
    // 세로 전체 화면, 상하 패딩 2rem
    <main className="h-screen my-8">
      <ProjectsSidebar />
    </main>
  );
}

export default App;
```

- h-screen 화면의 세로 길이 전부 차지

```jsx
// ProjectsSidebar.jsx

export default function ProjectsSidebar() {
  return (
    // 넓이 1/3, 좌우 패딩2rem, 상하 패딩 4rem, 배경색, 글자색, 프리픽스 넓이 18rem, 둥근모서리
    <aside className="w-1/3 px-8 py-16 bg-stone-900 text-stone-50 md:w-72 rounded-r-xl">
      <h2 className="mb-8 font-bold uppercase md:text-xl text-stone-200">
        Your Projects
      </h2>
      <div>
        <button className="px-4 py-2 text-xs md:text-base rounded-md bg-stone-700 text-stone-400 hover:bg-stone-600 hover:text-stone-100">
          + Add Project
        </button>
      </div>
      <ul></ul>
    </aside>
  );
}
```

- md: 사이즈가 너무 큰 스크린을 고려하여 최대 넓이 설정
- hover: 마우스가 위에 위치했을 때만 적용되는 속성

<br/>

## 재사용 가능한 입력 컴포넌트 추가하기

```jsx
// Input.jsx

// ...props 로 속성을 가져와서 이 컴포넌트 밖에서도 스타일을 부여 가능, 재사용성 높임
export default function Input({ label, textarea, ...props }) {
  // 반복되어 상수로 지정
  const classes =
    // 꽉 찬 가로, 모든 방향 패딩, 2픽셀 경계선, 입력 중일때 스타일 변화
    'w-full p-1 border-b-2 rounded-sm border-stone-300 bg-stone-200 text-stone-600 focus:outline-none focus:border-stone-600';

  return (
    <p className="flex flex-col gap-1 my-4">
      <label className="text-sm font-bold uppercase text-stone-500">
        {label}
      </label>
      {textarea ? (
        <textarea className={classes} {...props} />
      ) : (
        <input className={classes} {...props} />
      )}
    </p>
  );
}
```

- focus: 입력 중일 때 스타일 지정

```jsx
// NewProject.jsx

import Input from './Input.jsx';

export default function NewProject() {
  return (
    // [] 사용하여 커스텀 값 입력하기. 넓이, 위 여백
    <div className="w-[35rem] mt-16">
      <menu className="flex items-center justify-end gap-4 my-4">
        <li>
          <button className="text-stone-800 hover:text-stone-950">Cancel</button>
        </li>
        <li>
          <button className="px-6 py-2 rounded-md bg-stone-800 text-stone-50 hover:bg-stone-950">Save</button>
        </li>
      </menu>
      <div>
        <Input label="Title" />
        <Input label="Description" textarea />
        <Input label="Due Date" />
      </div>
    </div>
  );
}
```

- “[]” 사용하여 내장되지 않은 커스텀 속성 값 입력

![style](https://github.com/sujinann/ReactStudy/assets/139312979/07469b8a-c5e4-4e9b-9b5b-aedff82074a5)

<br/>

## 스타일을 위한 컴포넌트 분리

```jsx
// Button.jsx

export default function Button({ children, ...props }) {
  return (
    <button
      className="px-4 py-2 text-xs md:text-base rounded-md bg-stone-700 text-stone-400 hover:bg-stone-600 hover:text-stone-100"
      {...props}
    >
      {children}
    </button>
  );
}
```

- 버튼 스타일 재사용을 위한 분리

```jsx
// NoProjectSelected.jsx

import noProjectImage from '../assets/no-projects.png';
import Button from './Button.jsx';

export default function NoProjectSelected({ onStartAddProject }) {
  return (
    <div className="mt-24 text-center w-2/3">
      <img
        src={noProjectImage}
        alt="An empty task list"
        // object-contain 속성으로 이미지 깨지지 않게 조정, 중앙정렬
        className="w-16 h-16 object-contain mx-auto"
      />
      <h2 className="text-xl font-bold text-stone-500 my-4">
        No Project Selected
      </h2>
      <p className="text-stone-400 mb-4">
        Select a project or get started with a new one
      </p>
      <p className="mt-8">
        <Button onClick={onStartAddProject}>Create new project</Button>
      </p>
    </div>
  );
}
```

![noproject](https://github.com/sujinann/ReactStudy/assets/139312979/5b688f02-4df2-41b1-a747-528a0baa5523)

<br/>

## 컴포넌트 간 교환을 위한 상태 관리법

```jsx
// App.jsx

import { useState } from 'react';

import NewProject from './components/NewProject.jsx';
import NoProjectSelected from './components/NoProjectSelected.jsx';
import ProjectsSidebar from './components/ProjectsSidebar.jsx';

function App() {
  const [projectsState, setProjectsState] = useState({
    selectedProjectId: undefined,
    projects: []
  });

  function handleStartAddProject() {
    setProjectsState(prevState => {
      return {
        ...prevState,
        selectedProjectId: null,
      };
    });
  }

  let content;

  if (projectsState.selectedProjectId === null) {
    content = <NewProject />
  } else if (projectsState.selectedProjectId === undefined) {
    content = <NoProjectSelected onStartAddProject={handleStartAddProject} />;
  }

  return (
    <main className="h-screen my-8 flex gap-8">
      <ProjectsSidebar onStartAddProject={handleStartAddProject} />
      {content}
    </main>
  );
}
```

- 최소한의 상태만 사용하기 위해 SelectedProjectId 속성을 재사용
- undefined 프로젝트를 추가 한 적 없고, 선택 된 프로젝트가 없음
- 프로젝트를 추가하기 시작하면 null 값으로 변경

```jsx
export default function NoProjectSelected({ onStartAddProject }) {
  return (
      // ...
      <p className="mt-8">
        <Button onClick={onStartAddProject}>Create new project</Button>
      </p>
  );
}
```

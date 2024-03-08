# Refs(참조)와 전달된 Refs(참조)로 사용자 입력 받아오기~테일윈드 CSS로 모달 스타일링

### Refs(참조)와 전달된 Refs(참조)로 사용자 입력 받아오기

- 연습 목표: 제대로 유효하게 작성한 입력값을 저장하고 그 값을 사이드바에 표시해보자.
    - ref를 활용하자.
        - ref: HTML 요소와 연결 및 상호작용 가능
- (1)
    
    ```jsx
    # NewProject.jsx
    
    import { useRef } from "react"; // useRef 추가로 import
    import Input from "./Input.jsx";
    
    export default function NewProject() {
      // useRef를 활용하자.
      const title = useRef();
      const description = useRef();
      const dueDate = useRef();
    
      return (
        <div className="w-[35rem] mt-16">
          <menu className="flex items-center justify-end gap-4 my-4">
            <li>
              <button className="text-stone-800 hover:text-stone-950">
                Cancel
              </button>
            </li>
            <li>
              <button className="px-6 py-2 rounded-md bg-stone-800 text-stone-50 hover:bg-stone-950">
                Save
              </button>
            </li>
          </menu>
          <div>
            {/* Input이 커스텀 컴포넌트여서 내부의 input에 ref 속성이 설정되지 않아 작동하지 않는다. */}
            <Input label="Title" ref={title} />
            <Input label="Description" textarea />
            <Input label="Due Date" />
          </div>
        </div>
      );
    }
    ```
    
    - useRef를 import해서 활용한다.
    - ref 속성에 담아 컴포넌트 내부로 보낸다.
        - 빌트인이 아닌 커스텀 컴포넌트여서 내부 input에 참조 속성이 설정(작동)되지 않음
        - Cross-component 참조 연결 구조 완성이 필요
            - 새로 들어오는 ref를 전달해 이를 사용할 수 있게 해야 함
- (2)
    
    ```jsx
    # Input.jsx
    
    import { forwardRef } from "react"; // forwardRef 추가로 import
    
    // forwardRef로 컴포넌트 함수를 모두 wrap한다.
    // props 다음에 두 번째 parameter(매개변수)로 받아올 ref를 추가 작성해준다.
    const Input = forwardRef(function Input({ label, textarea, ...props }, ref) {
      const classes =
        "w-full p-1 border-b-2 rounded-sm border-stone-300 bg-stone-200 text-stone-600 focus:outline-none focus:border-stone-600";
    
      return (
        <p className="flex flex-col gap-1 my-4">
          <label className="text-sm font-bold uppercase text-stone-500">
            {label}
          </label>
          {textarea ? (
            <textarea className={classes} {...props} ref={ref} />
          ) : (
            <input className={classes} {...props} ref={ref} />
          )}
          {/* 두 렌더링 영역들에 위에서 받아온 ref값을 추가 작성한 ref 속성에 넣어준다. */}
        </p>
      );
    });
    
    export default Input; // wrap한 함수를 저장한 변수 혹은 상수를 내보낸다.
    
    ```
    
    - ref 속성을 보낸 컴포넌트 코드에서 forwardRef를 import해서 활용한다.
        - forwardRef로 컴포넌트 함수를 모두 wrap
        - wrap한 함수를 저장한 Input 상수(변수)를 export해 내보냄
        - props 다음에 두 번째 parameter로 받아올 ref를 추가 작성
        - textarea 혹은 input 영역들에 위에서 받아온 ref값을 추가 작성한 ref 속성에 넣어줌
- (3)
    
    ```jsx
    # NewProject.jsx
    
    import { useRef } from "react";
    import Input from "./Input.jsx";
    
    export default function NewProject() {
      const title = useRef();
      const description = useRef();
      const dueDate = useRef();
    
      // ref 속성을 가지고 저장 버튼을 누르면 그 값으로 갱신해주는 함수를 작성해주자.
      function handleSave() {
        const enteredTitle = title.current.value;
        const enteredDescription = description.current.value;
        const enteredDueDate = dueDate.current.value;
    
        // validation ...
      }
    
      return (
        <div className="w-[35rem] mt-16">
          <menu className="flex items-center justify-end gap-4 my-4">
            <li>
              <button className="text-stone-800 hover:text-stone-950">
                Cancel
              </button>
            </li>
            <li>
              <button className="px-6 py-2 rounded-md bg-stone-800 text-stone-50 hover:bg-stone-950" onClick={handleSave}>
                Save
              </button>
            </li>
          </menu>
          <div>
            <Input label="Title" ref={title} />
            <Input label="Description" textarea ref={description} />
            <Input label="Due Date" ref={dueDate} />
            {/* cross-component 참조 연결 구조 완성 후 나머지 Input 컴포넌트에도 각각 ref
            속성에 담아 보낸다. */}
          </div>
        </div>
      );
    }
    
    ```
    
    - 모든 Input 컴포넌트들에 ref 속성 안에 각각의 값(title, description, dueDate)을 담아 보낸다.
    - ref 속성을 가지고 저장 버튼을 누르면 그 값으로 갱신해주는 handleSave 함수를 추가해주자.
        - Save 버튼에 onClick 이벤트 시 handleSave를 실행하도록 작성
        - 각각에 알맞은 ref 값으로 갱신해주되, .current.value를 붙여 작성
        - 유효성 검사(validation)은 추후 작성할 예정
- (4)
    
    ```jsx
    # App.jsx
    
    import { useState } from "react";
    import NewProject from "./components/NewProject.jsx";
    import NoProjectSelected from "./components/NoProjectSelected.jsx";
    import ProjectsSidebar from "./components/ProjectsSidebar.jsx";
    
    function App() {
      const [projectsState, setProjectsState] = useState({
        selectedProjectId: undefined,
        projects: [],
      });
    
      // 프로젝트 추가 작업을 시작하는 함수
      function handleStartAddProject() {
        setProjectsState((prevState) => {
          return {
            ...prevState,
            selectedProjectId: null,
          };
        });
      }
    
      // 프로젝트 추가 작업을 끝내는 함수
      function handleAddProject(projectData) {
        setProjectsState((prevState) => {
          const newProject = {
            ...projectData,
            id: Math.random(),
          };
    
          return {
            ...prevState,
            projects: [...prevState.projects, newProject],
          };
        });
      }
    
      let content;
      if (projectsState.selectedProjectId === null) {
        content = <NewProject />;
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
    
    export default App;
    ```
    
    - 모든 프로젝트를 생성 및 관리하는 App.jsx로 올라와서 갱신 값을 사이드바에 반영해주자.
        - 유일하게 NewProject.jsx에 접근할 수 있기 때문
    - 프로젝트 추가 작업을 끝내는 함수 handleAddProject 함수를 추가하자.
        - 새 프로젝트 newProject를 작성해서 projects 배열에 추가해주는 함수
            - projectData를 받아와서 스프레드연산자를 이용해 펼침 (title, description, dueDate)
            - 프로젝트 선택에 유용하도록 id도 추가해줌 (미니 프로젝트이므로 랜덤으로 값 지정)
            - projects 배열에 newProject 추가
- (5)
    
    ```jsx
    # NewProject.jsx
    
    import { useRef } from "react";
    import Input from "./Input.jsx";
    
    // onAdd라는 prop을 밖에서 받아온다
    export default function NewProject({ onAdd }) {
      const title = useRef();
      const description = useRef();
      const dueDate = useRef();
    
      function handleSave() {
        const enteredTitle = title.current.value;
        const enteredDescription = description.current.value;
        const enteredDueDate = dueDate.current.value;
    
        // validation ...
    
        // onAdd를 활용해서 새로 작성한 각 내용이 저장되도록 하자.
        onAdd({
          title: enteredTitle,
          description: enteredDescription,
          dueDate: enteredDueDate,
        });
      }
    
      return (
        <div className="w-[35rem] mt-16">
          <menu className="flex items-center justify-end gap-4 my-4">
            <li>
              <button className="text-stone-800 hover:text-stone-950">
                Cancel
              </button>
            </li>
            <li>
              <button
                className="px-6 py-2 rounded-md bg-stone-800 text-stone-50 hover:bg-stone-950"
                onClick={handleSave}
              >
                Save
              </button>
            </li>
          </menu>
          <div>
            <Input type="text" label="Title" ref={title} />
            <Input label="Description" textarea ref={description} />
            <Input type="date" label="Due Date" ref={dueDate} />
            {/* date picker 설정(캘린더에서 고르는 기능)을 위해 type="date"를 추가 작성해준다 */}
          </div>
        </div>
      );
    }
    
    ```
    
    ```jsx
    # App.jsx
    
    import { useState } from "react";
    import NewProject from "./components/NewProject.jsx";
    import NoProjectSelected from "./components/NoProjectSelected.jsx";
    import ProjectsSidebar from "./components/ProjectsSidebar.jsx";
    
    function App() {
      const [projectsState, setProjectsState] = useState({
        selectedProjectId: undefined,
        projects: [],
      });
    
      function handleStartAddProject() {
        setProjectsState((prevState) => {
          return {
            ...prevState,
            selectedProjectId: null,
          };
        });
      }
    
      function handleAddProject(projectData) {
        setProjectsState((prevState) => {
          const newProject = {
            ...projectData,
            id: Math.random(),
          };
    
          return {
            ...prevState,
            projects: [...prevState.projects, newProject],
          };
        });
      }
    
      // 프로젝트 목록이 제대로 바뀌고 있는지 확인하기 위한 콘솔 출력
      console.log(projectsState);
    
      let content;
    
      if (projectsState.selectedProjectId === null) {
        content = <NewProject onAdd={handleAddProject} />; // onAdd라는 prop에 handleAddProject 함수를 넣어줌
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
    
    export default App;
    
    ```
    
    - NewProject.jsx에서 onAdd라는 이름으로 새 프로젝트를 작성 및 저장하는 함수를 받아오자.
        - 저장 버튼을 눌러 실행되는 handleSave 함수 안에 onAdd를 활용
            - 새로 작성한 값을 각 key에 맞게 저장시킴 (title, description, dueDate)
        - 하단의 Input 컴포넌트에 type를 구체적으로 지정
            - title이 들어가는 Input은 text로 지정
            - dueDate가 들어가는 Input은 date로 지정
                - 리액트에 내장된 type이어서 date picker 설정 가능 (캘린더에서 고르는 기능)
    - App.jsx로 올라가 NewProject 컴포넌트에 handleAddProject 함수를 전달하자.
        - onAdd라는 prop의 값으로 전달
        - 콘솔로 projectsState를 출력해 프로젝트 목록이 제대로 갱신되는지 확인
            - 두 번씩 콘솔에 출력되는 이유: 리액트의 Strict Mode 때문 (컴포넌트 함수들을 두 번씩 실행)
- 실행 결과
    - 콘솔에 정상적으로 추가한 프로젝트가 저장됨
    - 저장 버튼을 눌렀을 때 새 프로젝트 작성 컴포넌트가 닫히지 않음
    - 사이드바에 새 프로젝트가 렌더링되지 않음

### 프로젝트 생성 핸들링 & UI 업데이트

- 연습 목표 1: Save 버튼을 누르면 프로젝트 생성 컴포넌트가 사라지게 해보자.
    - selectedProjectId의 상태 변화를 확인해야 한다.
        - undefined일 때: NewProject 컴포넌트가 닫혀 있어야 함
        - null일 때: NewProject 컴포넌트가 열려 있어야 함
- (1)
    - 2가지 방법
        
        ```jsx
        # App.jsx
        
          function handleAddProject(projectData) {
            setProjectsState((prevState) => {
              const projectId = Math.random(); // 밖으로 빼둠
              const newProject = {
                ...projectData,
                id: projectId, // 선택된 프로젝트의 id가 될 예정
              };
        
              return {
                ...prevState,
                selectedProjectId: undefined, // 지금은 undefined로 직접 변경
                projects: [...prevState.projects, newProject],
              };
            });
          }
        ```
        
        - handleAddProject 함수에서 return 시 undefined로 직접 변환
            - 일단 이 방법으로 진행
        - projectId 변수(상수)를 따로 만들어 return 시 projectId로 설정
            - 아직 프로젝트 선택 단계까지 가지 않아서 이 방법은 보류
- 연습 목표 2: Save 버튼을 누르면 사이드바에 갱신된 프로젝트 목록이 뜨게 해보자.
    - App.jsx에서 ProjectsSidebar.jsx로 prop을 전달해야 한다.
        - projectsState.projects 배열을 넘겨 갱신된 프로젝트 목록을 띄울 수 있게 함
    - ProjectsSidebar.jsx에서 prop을 받아온다.
- (2)
    
    ```jsx
    # App.jsx
    
    function App() {
    
    // 중략
    
      return (
        <main className="h-screen my-8 flex gap-8">
          {/* prop으로 projectsState 안의 projects 배열을 넘겨준다. */}
          <ProjectsSidebar
            onStartAddProject={handleStartAddProject}
            projects={projectsState.projects}
          />
          {content}
        </main>
      );
    }
    ```
    
    ```jsx
    # ProjectsSidebar.jsx
    
    import Button from "./Button.jsx";
    
    // 밖에서 projects를 받아온다.
    export default function ProjectsSidebar({ onStartAddProject, projects }) {
      return (
        <aside className="w-1/3 px-8 py-16 bg-stone-900 text-stone-50 md:w-72 rounded-r-xl">
          <h2 className="mb-8 font-bold uppercase md:text-xl text-stone-200">
            Your Projects
          </h2>
          <div>
            <Button onClick={onStartAddProject}>+ Add Project</Button>
          </div>
          {/* map을 사용할 떄는 항상 root 항목에 고유 식별자인 key 속성을 받아와야 한다. */}
          {/* mt: 위쪽 margin 설정 */}
          <ul className="mt-8">
            {projects.map((project) => (
              <li key={project.id}>
                {/* w-full:사용 가능한 너비를 모두 사용 
                text-left: 텍스트를 왼쪽 정렬
                px: 좌우 padding 설정
                py: 상하 padding 설정
                rounded-sm: 둥근 모서리
                my: 상하 margin 설정
                hover: 마우스를 hover할 때의 이벤트
                text: 글자색 설정
                bg: 배경색 설정  */}
                <button className="w-full text-left px-2 py-1 rounded-sm my-1 text-stone-400 hover:text-stone-200 hover:bg-stone-800">
                  {project.title}
                </button>
              </li>
            ))}
          </ul>
        </aside>
      );
    }
    ```
    
    - projects라는 prop으로 넘겨주고 그걸 받아온다.
    - map 기능을 사용해서 projects를 펼쳐 보여준다.
        - map: 항상 root 항목에 고유 식별자인 key 속성을 받아와야
        - 겹치지 않는 project.id를 key로 설정
    - 사이드바에 바로 보여주기 위해 project.title을 버튼에 동적으로 작성해준다.
    - 테일윈드 스타일을 적용해주자. (위 코드 참고)
- 실행 결과
    - 새 프로젝트를 작성하고 Save를 누르면 사이드바에 그 프로젝트가 추가됨
    - 새 프로젝트를 작성하는 컴포넌트가 정상적으로 닫힘

### 사용자 입력 유효성 검사 & useImperativeHandle로 에러 모달 띄우기

- 연습 목표: 유효하지 않은 값일 때 alert로 오류 메세지를 띄워보자.
- (1)
    
    ```jsx
    # NewProject.jsx
    
    function handleSave() {
        const enteredTitle = title.current.value;
        const enteredDescription = description.current.value;
        const enteredDueDate = dueDate.current.value;
    
        if (
          enteredTitle.trim() === "" ||
          enteredDescription.trim() === "" ||
          enteredDueDate.trim() === ""
        ) {
          // 에러 alert를 띄우기 위함
        }
    
        onAdd({
          title: enteredTitle,
          description: enteredDescription,
          dueDate: enteredDueDate,
        });
      }
    ```
    
    - NewProject.jsx에 유효성 검사 if문 조건을 작성한다.
        - 각 입력값을 trim한 결과가 빈 값일 경우 유효하지 않다고 인식
- (2)
    
    ```jsx
    # Modal.jsx
    
    import { createPortal } from "react-dom"; // 포털을 사용하자.
    import { forwardRef, useImperativeHandle, useRef } from "react"; // 함수 호출에 사용할 useRef, forwardRef, useImperativeHandle 훅을 사용하자.
    
    const Modal = forwardRef(function Modal({ children }, ref) {
      const dialog = useRef();
      useImperativeHandle(ref, () => {
        return {
          // 임의로 지정한 함수명 => 바깥에서 호출 가능해짐
          open() {
            // 현재 모달을 보여주는 역할
            dialog.current.showModal();
          },
        };
      });
      return createPortal(
        // ref 속성에 dialog 값을 넣어준다.
        <dialog ref={dialog}>{children}</dialog>,
        document.getElementById("modal-root"),
      );
      // modal-root라는 id를 가진 div에 모달을 포탈시키자.
    });
    
    export default Modal;
    
    ```
    
    ```jsx
    # index.html
    
      <body class="bg-stone-50">
        <!-- Modal.jsx를 포탈시킬 위치 -->
        <div id="modal-root"></div>
        <div id="root"></div>
        <script type="module" src="/src/main.jsx"></script>
      </body>
    ```
    
    - Modal.jsx라는 새 파일을 생성한다.
        - 모달 내용으로 보여줄 children을 동적으로 받아옴
            - dialog라는 태그에 chileren을 동적으로 보여줌
        - 모달창을 열기 위한 함수를 바깥에서 호출할 수 있도록 useRef, forwardRef, useImperativeHandle 훅을 활용
            - forwardRef로 컴포넌트 함수를 모두 wrap한 뒤 밖으로 export
            - ref도 밖에서 두 번째 인수로 받아와 useImperativeHandle 훅에서도 활용
            - dialog라는 변수를 useRef로 선언
            - 임의로 지정한 open이라는 함수 안에 dialog.current.showModal 함수(모달창을 보여주는 기능)를 작성해 바깥에서 open 함수를 호출하면 모달창을 열게끔 함
            - dialog라는 태그 안의 ref 속성에 dialog 값을 넣어줌
        - 포탈로 index.html 파일의 modal-root라는 id를 가진 div 위치에 Modal을 텔레포트
            - createPortal로 return 값을 모두 wrap한 뒤, 포탈시킬 위치를 document.getElementById()로 지정
- (3)
    
    ```jsx
    # NewProject.jsx
    
    import { useRef } from "react";
    import Input from "./Input.jsx";
    import Modal from "./Modal.jsx";
    
    export default function NewProject({ onAdd }) {
      // 새로운 참조를 추가한다.
      const modal = useRef();
    
      const title = useRef();
      const description = useRef();
      const dueDate = useRef();
    
      function handleSave() {
        const enteredTitle = title.current.value;
        const enteredDescription = description.current.value;
        const enteredDueDate = dueDate.current.value;
    
        if (
          enteredTitle.trim() === "" ||
          enteredDescription.trim() === "" ||
          enteredDueDate.trim() === ""
        ) {
          modal.current.open(); // 유효하지 않은 값일 경우 에러 alert를 띄움
          return; // 하단의 onAdd 실행을 막기 위함
        }
    
        onAdd({
          title: enteredTitle,
          description: enteredDescription,
          dueDate: enteredDueDate,
        });
      }
    
      return (
        <>
          {/* Modal 창에 띄울 내용을 추가 작성해준다. */}
          <Modal ref={modal} buttonCaption="Okay">
            <h2>Invalid Input</h2>
            <p>Oops... looks like you forgot to enter a value.</p>
            <p>Please make sure you provide a valid value for every input field.</p>
          </Modal>
          <div className="w-[35rem] mt-16">
            <menu className="flex items-center justify-end gap-4 my-4">
              <li>
                <button className="text-stone-800 hover:text-stone-950">
                  Cancel
                </button>
              </li>
              <li>
                <button
                  className="px-6 py-2 rounded-md bg-stone-800 text-stone-50 hover:bg-stone-950"
                  onClick={handleSave}
                >
                  Save
                </button>
              </li>
            </menu>
            <div>
              <Input type="text" label="Title" ref={title} />
              <Input label="Description" textarea ref={description} />
              <Input type="date" label="Due Date" ref={dueDate} />
            </div>
          </div>
        </>
      );
    }
    
    ```
    
    ```jsx
    # Modal.jsx
    
    import { createPortal } from "react-dom";
    import { forwardRef, useImperativeHandle, useRef } from "react";
    
    // 버튼에 사용할 문구도 buttonCaption이라는 prop으로 받아오자.
    const Modal = forwardRef(function Modal({ children, buttonCaption }, ref) {
      const dialog = useRef();
      useImperativeHandle(ref, () => {
        return {
          open() {
            dialog.current.showModal();
          },
        };
      });
      return createPortal(
        <dialog ref={dialog}>
          {children}
          <form method="dialog">
            <button>{buttonCaption}</button>
          </form>
          {/* 버튼을 누르면 모달창을 닫을 수 있도록 form을 추가 작성 */}
        </dialog>,
        document.getElementById("modal-root"),
      );
    });
    
    export default Modal;
    ```
    
    - NewProject.jsx의 유효성 검사 코드를 완성한다.
        - modal이라는 새로운 참조 ref를 추가
        - if 조건문에 해당할 시 modal.current.open() 함수를 호출해 에러 메세지를 띄움
            - return을 적어 하단의 onAdd 실행을 방지함
        - 아까 export한 Modal 컴포넌트를 return 문에 포함시킴
            - 아까 forwardRef로 받아오려 했던 ref 속성에 modal 값을 넣어줌
    - 모달창에 띄울 내용을 추가 작성해준다.
        - NewProject.jsx의 Modal 태그 사이에 띄울 메세지를 모두 작성
    - 모달창을 닫을 버튼을 만들어준다.
        - Modal.jsx에서 dialog 태그 사이에 form 태그를 dialog라는 method로 작성
            - form 태그 사이에 버튼을 만듦
                - form 안의 버튼을 누를 시 리액트에서 자동으로 모달창을 닫아줌
            - buttonCaption이라는 prop을 바깥에서 받아옴 (컴포넌트 함수에 인수 추가)
            - button 태그 안에 버튼에 띄울 문구로 쓸 buttonCaption 값을 동적으로 보여줌
- 실행 결과
    - 원시적인 형태이긴 하지만, 유효하지 않을 경우 모달창이 정상적으로 뜸
    - Okay 버튼을 누르면 모달창이 정상적으로 닫힘

### 테일윈드 CSS로 모달 스타일링

- 연습 목표 1: 모달창을 스타일링해보자.
    - 모달창 및 배경색 스타일링
    - 모달창 문구 스타일링
- (1)
    
    ```jsx
    # Modal.jsx
    
        // backdrop: dialog 뒤쪽에 렌더링시킬 클래스를 작성
        // 투명도를 표기할 경우: 숫자 뒤에 /숫자로 추가 표기 (예: 900/90) (숫자 = 불투명도)
        // p: 모든 면에 패딩 추가
        // rounded-md: 둥근 모서리
        // shadow-md: 뒤에 그림자 추가
        <dialog
          ref={dialog}
          className="backdrop:bg-stone-900/90 p-4 rounded-md shadow-md"
        >
    ```
    
    - Modal.jsx에서 모달창 및 배경색을 스타일링한다. (각 스타일은 주석 참고)
- (2)
    
    ```jsx
    # NewProject.jsx
    
          <Modal ref={modal} buttonCaption="Okay">
            <h2 className="text-xl font-bold text-stone-500 my-4">Invalid Input</h2>
            <p className="text-stone-400 mb-4">
              Oops... looks like you forgot to enter a value.
            </p>
            <p className="text-stone-400 mb-4">
              Please make sure you provide a valid value for every input field.
            </p>
          </Modal>
    ```
    
    ```jsx
    # Modal.jsx
    
    import Button from "./Button.jsx";
    
    // 중략
    
        <dialog
          ref={dialog}
          className="backdrop:bg-stone-900/90 p-4 rounded-md shadow-md"
        >
          {children}
          {/* mt: margin top */}
          {/* text-right: 우측 정렬 */}
          <form method="dialog" className="mt-4 text-right">
            {/* 일반 button 태그 대신 커스텀으로 작성했던 Button으로 수정 */}
            <Button>{buttonCaption}</Button>
          </form>
        </dialog>,
    ```
    
    - NoProjectSelected.jsx에서 모달창 문구를 스타일링할 클래스들(여기서는 h2와 p의 className들)을 복사해서 NewProject.jsx에 붙여넣는다.
    - Modal.jsx에서 버튼을 스타일링한다. (각 스타일은 주석 참고)
- (3)
    
    ```jsx
    # NewProject.jsx
    
          <Modal ref={modal} buttonCaption="Okay">
            <h2 className="text-xl font-bold text-stone-700 my-4">Invalid Input</h2>
            <p className="text-stone-600 mb-4">
              Oops... looks like you forgot to enter a value.
            </p>
            <p className="text-stone-600 mb-4">
              Please make sure you provide a valid value for every input field.
            </p>
          </Modal>
    ```
    
    - NewProject.jsx에서 문구를 더 진하게 스타일링한다.
- 연습 목표 2: Cancel 버튼이 작동되게 해보자.
    - Cancel 버튼을 누르면 projectsState.selectedProjectId가 undefined로 돌아가도록 한다.
- (4)
    
    ```jsx
    # App.jsx
    
      function handleStartAddProject() {
        setProjectsState((prevState) => {
          return {
            ...prevState,
            selectedProjectId: null,
          };
        });
      }
    
      // 새 프로젝트를 생성하는 작업을 취소하는 함수 추가 작성
      function handleCancelAddProject() {
        setProjectsState((prevState) => {
          return {
            ...prevState,
            selectedProjectId: undefined, // null이 아닌 undefined로 돌아오도록
          };
        });
      }
      
      if (projectsState.selectedProjectId === null) {
        // onCancel이라는 prop에 handleCancelAddProject 함수를 담아 보낸다.
        content = (
          <NewProject onAdd={handleAddProject} onCancel={handleCancelAddProject} />
        );
      } else if (projectsState.selectedProjectId === undefined) {
        content = <NoProjectSelected onStartAddProject={handleStartAddProject} />;
      }
    ```
    
    - App.jsx에 새로운 함수 handleCancelAddProject를 추가 작성한다.
        - selectedProjectId를 null에서 undefined로 돌려놓음
        - 나머지는 handleStartAddProject 함수와 똑같이 작동
        - NewProject 컴포넌트에 onCancel이라는 prop으로 함수를 담아 보냄
- (5)
    
    ```jsx
    # NewProject.jsx
    
    // 밖에서 onCancel이라는 prop을 받아온다.
    export default function NewProject({ onAdd, onCancel }) {
    
    // 중략
    
              <li>
                {/* 클릭 시 onCancel이 실행되도록 한다. */}
                <button
                  className="text-stone-800 hover:text-stone-950"
                  onClick={onCancel}
                >
                  Cancel
                </button>
              </li>
    ```
    
    - NewProject.jsx에서 onCancel을 받아온다.
        - Cancel 버튼 클릭 시 onCancel이 호출되도록 작성
- 실행 결과
    - 모달창이 그럴싸한 모양으로 나타남
    - Cancel 버튼을 누르면 초기 화면으로 잘 돌아감
    - 아직 사이드바에 생성된 프로젝트의 상세 화면이 없는 상태 ⇒ 추후 작성 예정

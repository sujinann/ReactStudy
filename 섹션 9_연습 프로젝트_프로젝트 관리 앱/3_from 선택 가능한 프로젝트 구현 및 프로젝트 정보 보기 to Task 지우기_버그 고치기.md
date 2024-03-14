# 8주차 9강

### 선택 가능한 프로젝트 구현 및 프로젝트 정보 보기

- 왼쪽에 등록된 프로젝트 목록에서 제목을 누를 때 프로젝트 세부 사항이 렌더링되도록 구현할것임
- 목록에서 제목 클릭에 따른 세부 사항 props 전달을 해줘야함

```jsx
// SelectedProject.jsx
// project 객체를 전달받아 예쁘게 표시
export default function SelectedProject({project}) {
		// date formatter
    const formattedDate = new Date(project.dueDate).toLocaleDateString('ko-KR', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
    });
    
    return (
        <div className="w-[35rem] mt-16">
            <header className="pb-4 mb-4 border-b-2 border-stone-300">
                <div className="flex items-center justify-between">
                    <h1 className="text-3xl font-bold text-stone-600 mb-2.5">{project.title}</h1>
                    <button className="text-stone-600 hover:text-stone-950">Delete</button>
                </div>
                <p className="mb-4 text-stone-400">{formattedDate}</p>
								// whitespace prewrap : 여러 줄에 걸친 텍스트를 줄바꿈할 때 단어가 깨지지 않음
                <p className="text-stone-600 whitespace-pre-wrap">{project.description}</p>
            </header>
            TASKS
        </div>
    )
}

// App.jsx
...

function App() {
	...
  function handleSelectProject(id) {
    setProjectState((prevState) => {
      return {
        ...prevState,
        selectedProjectId: id,
      }
    })
  }

  ...
  // Array에서 id가 일치하는 객체를 반환하는 함수
  const selectedProject = projectsState.projects.find(project => project.id === projectsState.selectedProjectId);

  // SelectedProject에 id로 선택한 project 전달
  let content = <SelectedProject project={selectedProject}/>;

  if (projectsState.selectedProjectId === null) {
    content = <NewProject onAdd={handleAddProject} onCancel={handleCancelAddProject} />
  }
  else if (projectsState.selectedProjectId === undefined) {
    content = <NoProjectSelected onStartAddProject={handleStartAddProject} />
  }
  return (
    <main className="h-screen my-8 flex gap-8">
      <ProjectSidebar 
        onStartAddProject={handleStartAddProject} 
        projects={projectsState.projects}
        onSelectProject={handleSelectProject} />
      {content}
    </main>
  );
}

export default App;

//ProjectSidebar.jsx
import Button from "./Button";
import SelectedProject from "./SelectedProject";

export default function ProjectSidebar({
    onStartAddProject,
    projects,
    onSelectProject,
    selectedProjectId
}) {
    return (
        <aside className="w-1/3 px-8 py-16 bg-stone-900 text-stone-50 md:w-72 rounded-r-xl">
            <h2 className="mb-8 font-bold uppercase md:text-xl text-stone-200">
                Your Projects
            </h2>
            <div>
                <Button onClick={onStartAddProject}>+ Add Project</Button>
            </div>
            <ul className="mt-8">
								// 배열 map 함수
                {projects.map((project) => {
                    let cssClasses = "w-full text-left px-2 py-1 rounded-sm my-1 hover:text-stone-200 hover:bg-stone-800"
                    if (project.id === selectedProjectId) {
                        cssClasses += " bg-stone-100 text-stone-200"
                    } else {
                        cssClasses += " text-stone-400"
                    }

                    return (
                        <li key={project.id}>
                            <button
                                className={cssClasses}
                                onClick={onSelectProject}                        
														>{project.title}</button>
                        </li>
                    )
                })}
            </ul>
        </aside >
    );
}
```

- dueDate를 읽을 수 없다는 에러 발생(Cannot read properties of undefined(reading ’DueDate’)
    - App 컴포넌트에 HandleSelectProject 함수를 추가했음(SelectedProjectId 설정해주는 함수)
    - 이후 onSelect로 ProjectSidebar에 전달
    - ProjectSidebar에서는 onSelectProject 속성으로 같은 함수가 내장되어 있음

```jsx
return (<li key={project.id}>
                        <button className={cssClasses}
                            onClick=~~{onSelectProject}~~
														onClick={() => onSelectProject(project.id)}
														
                        >{project.title}</button>
</li>)
```

![Untitled 1](https://github.com/chomchom96/ReactStudy/assets/112466460/edef2998-ae0f-404f-9612-bcdface69004)

- 아직 Delete 버튼이 작동하지 않음

### 프로젝트 삭제 핸들링

- 간단한 filter 함수로 배열에서 선택된 프로젝트를 지울 수 있음

```jsx
 function handleDeleteProject() {
    setProjectState((prevState) => {
      return {
        ...prevState,
        selectedProjectId: undefined,
        projects: prevState.projects.filter((project) => {
					// false가 return된 객체를 삭제함
          project.id !== prevState.selectedProjectId
        })
      }
    })
  }

  let content = <SelectedProject project={selectedProject} onDelete={handleDeleteProject}/>;

// SelectedProject.jsx
// props로 onDelete 함수 전달
export default function SelectedProject({project, onDelete}) {

...
						<button
                    onClick={onDelete}>Delete</button>

```

### 태스크 컴포넌트 추가

- TASKS(할 일 목록) 작동하게 만들기

```jsx
// Tasks.jsx
import NewTask from "./NewTask"

export default function Tasks() {
    return (
        <section>
            <h2 className="text-2xl font-bold text-stone-700 mb-4">Tasks</h2>
            <NewTask />
            <p className="text-stone-800 mb-4">This project does not have any tasks yet.</p>
            <ul>

            </ul>

        </section>
    )
}

// NewTask.jsx
export default function NewTask () {
  return (
    <div>
        <input type="text" className="w-64 px-2 py-1 rounded-sm bg-stone-200" />
        <button className="text-stone-700 hover:text-stone-950 mx-4">Add Task</button>
    </div>
  )
}

```

![Untitled 2](https://github.com/chomchom96/ReactStudy/assets/112466460/a6553062-98fa-41e0-8e48-1e5733cef00b)

### Task 관리 & Props Drilling

- App.jsx에 id를 key로 프로젝트와 Task List를 묶을 수 있음
    - App → Tasks까지 거치는 컴포넌트의 개수가 매우 많아서 번거로움
        - NewTask ← Tasks ← SelectedProject ← ProjectSidebar ← App
    - 나중에는 전역 상태 관리를 사용할 예정
- Property Drilling 을 사용할거임
    - App의 HandleAddTask, HandleDeleteTask를 Tasks.jsx까지 Drillling

```jsx
// App.jsx
let content = <SelectedProject
    project={selectedProject}
    onDelete={handleDeleteProject}
    onAddTask={handleAddTask}
    onDeleteTask={handleDeleteTask}
/>;

// SelectedProject.jsx

export default function SelectedProject({project, onDelete, onAddTask, onDeleteTask}) {
...
            <Tasks onAdd={onAddTask} onDelete={onDeleteTask}/>

// Tasks.jsx
export default function Tasks({onAdd, onDelete}) {
...
            <NewTask onAdd={onAdd} />
// NewTask.jsx
export default function NewTask ({onAdd}) {
...
		function handleClick() {
        onAdd(enteredTask);
        setEnteredTask(''); // 입력 후 빈칸 초기화
    }
```

- Drilling이 끝나면 시작점에서 prop을 다룰 수 있게됨

```jsx
// 로직 재활용
function handleAddTask(text) { 
    setTasksState(prevState => {
      const taskId = Math.random();
      const newTask = {
        text: text,
        id: taskId,
        projectId: projectId
      }
      return {
        ...prevState,
        tasks: [
          newTask,
          ...prevState.tasks,
        ]
      }
    })
  }
```

- 이제 프로젝트에 속하는 Tasks 목록을 가져와야함

```jsx
// App.jsx
 let content = <SelectedProject
    project={selectedProject}
    onDelete={handleDeleteProject}
    onAddTask={handleAddTask}
    onDeleteTask={handleDeleteTask}
    tasks={projectsState.tasks}
  />;

.. 같은 방법으로 Tasks.jsx 까지 Drilling 

// Tasks.jsx
import NewTask from "./NewTask"

export default function Tasks({ tasks, onAdd, onDelete }) {
    return (
        <section>
            <h2 className="text-2xl font-bold text-stone-700 mb-4">Tasks</h2>
            <NewTask onAdd={onAdd} onDelete={onDelete} />
            {tasks.length === 0 && (<p
                className="text-stone-800 my-4">
                This project does not have any tasks yet.
            </p>
            )}
            {tasks.length > 0 && (
                <ul className="p-4 mt-8 rounded-md bg-stone-100">
                    {tasks.map((tasks) => (
                        <li keys={tasks.id}
                        className="flex justify-between my-4">
                        <span>{tasks.text}</span>
                        <button className="text-stone-700 hover:text-red-500"
                        onClick={onDelete}
                        >Clear</button>
                    </li>
                    ))}
                </ul>
                )}
        </section>
    )
}

```

![Untitled 3](https://github.com/chomchom96/ReactStudy/assets/112466460/69559596-264b-46cf-b987-945d1ffc50be)

### Task 지우기 + 버그 고치기

- handleDeleteTask 로직 만들기 + Clear 버튼에 연결

```jsx
function handleDeleteTask(id) {
    setProjectState((prevState) => {
      return {
        ...prevState,
        tasks: prevState.tasks.filter((task) =>
          task.id !== id
        )
      }
    })
  }

// Tasks.jsx
<button className="text-stone-700 hover:text-red-500"
                        onClick={() => onDelete(tasks.id)}
                        >Clear</button>
// 포인터 전달하지 말고 직접 실행햐아함(id param 전달)
```
![Untitled 4](https://github.com/chomchom96/ReactStudy/assets/112466460/73e411ff-2dc5-4e24-8380-acdb621e85ed)

- 작동은 하지만 개발자 도구에서 WARNING 문구 확인 가능


- NewTask에서 파생한 에러(로그에서도 보임)
- enteredTask가 초기값이 undefined이므로 발생
    - input value 안의 값이 undefined이므로 react의 관리 대상에서 제외됨

```jsx
    const [enteredTask, setEnteredTask] = useState(''); // 초기값 넣어주면 해결

```

- 빈 값인 Task도 추가됨

```jsx
 		function handleClick() {
        if (enteredTask.trim() === '') {
            alert('빈 칸 넣지마 임마')
        }
        onAdd(enteredTask);
        setEnteredTask(''); // 입력 후 빈칸 초기화
    }
```

![Untitled](https://github.com/chomchom96/ReactStudy/assets/112466460/0adecd2a-98e7-4f9c-ae3d-66359cb2ec64)

- 선택된 프로젝트 강조 css가 없음

```jsx
return (
    <main className="h-screen my-8 flex gap-8">
      <ProjectSidebar
        ...
// 추가해주세요
        selectedProjectId={projectsState.selectedProjectId}
        />
      {content}
    </main>
  );

```

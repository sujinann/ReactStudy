# 여러 JSX 슬롯 활용법 ~ 세부 과정: 이미지 저장소는 public/ VS assets/

### 여러 JSX 슬롯 활용법

-   컴포넌트의 구조가 공통된 래퍼(wrapper) 구조를 띨 경우:
    -   새로운 wrapper component를 생성한다.
        ```jsx
        export default function Tabs() {
            // 모든 종류의 탭에 적용하고자 함
            return (
                <>   // Fragment
                    <menu>   // menu 탭: 버튼을 모두 감싸도록
                        BUTTONS // menu 바 안: TabButtons 같은 버튼들이 들어감
                    </menu>
                    CONTENT // menu 바 밖: 누르는 탭 버튼에 따라 다르게 보여줄 실제 내용
                </>
            );
        }
        ```
    -   속성 구조 분해 할당(prop destructuring)으로 자식 속성(children, buttons)을 가져온다.
        ```jsx
        export default function Tabs({ children, buttons }) {
            return (
                <>   // Fragment
                    <menu>   // menu 탭: 버튼을 모두 감싸도록
                        {buttons}   // menu 바 안: TabButtons 같은 버튼들이 들어감
                    </menu>
                    {children}   // menu 바 밖: 누르는 탭 버튼에 따라 다르게 보여줄 실제 내용
                </>
            );
        }
        ```
    -   wrapper component를 사용할 코드에서 import해준다.
        ```jsx
        import Tabs from "./Tabs.jsx";   // wrapper component를 import

        <Tabs>{tabContent}<Tabs/>   // 작성한 wrapper component를 사용
        ```
    -   공통으로 사용되지 않는 부분은 wrapper component 안으로 끌고 오지 않는다. (예를 들어, {children}에 들어갈 {tabContent} 등)
    -   buttons에 들어갈 모든 코드를 복사해서 Tabs 태그에 버튼 속성(buttons={ })을 추가한 뒤 붙여 넣는다. 이 때, 다수의 형제 요소가 들어갈 수는 없으므로 ```<div>```로 감싸주거나, Fragment(<>)로 감싸줘야 한다.
        ```jsx
        <Tabs
            buttons={
                <>
                    <TabButton isSelected={selectedTopic === 'components'} onClick={() => handleSelect('components')}>
                        Components
                    </TabButton>
                    <TabButton isSelected={selectedTopic === 'jsx'} onClick={() => handleSelect('jsx')}>
                        JSX
                    </TabButton>
                    <TabButton isSelected={selectedTopic === 'props'} onClick={() => handleSelect('props')}>
                        Props
                    </TabButton>
                    <TabButton isSelected={selectedTopic === 'state'} onClick={() => handleSelect('state')}>
                        State
                    </TabButton>
                </>
            }
        >
            {tabContent}
        </Tabs>
        ```

### 컴포넌트 타입 동적으로 설정하기

-   공통 wrapper component를 더욱 유연하게 만들어 재사용도를 높이고 싶다면:
    -   식별자인 menu 탭도 속성 값으로 설정한다. 만약 내장된 menu가 아닌, custom된 component를 동적으로 작성하고 싶다면 “menu”를 {Section}으로 바꿔 써도 된다.
        -   커스텀된 속성 값은 중괄호 안에 (괄호 없이) 함수/변수 명을 작성하고, 내장된 속성 값은 따옴표 안에 문자열처럼 작성해야 한다.
            ```jsx
            <Tabs
                buttonsContainer="menu" // "menu"를 {Section}으로 바꾸는 것도 가능.
                // 중략
            >
                {tabContent}
            </Tabs>
            ```
        -   밖에 작성한 속성 값을 집어넣을, 대문자로 시작하는 변수를 선언해서 그 값을 넣어준다.
            ```jsx
            export default function Tabs({ children, buttons, buttonsContainer }) {
                const ButtonsContainer = buttonsContainer; // 반드시 대문자로 시작해야!
                return (
                    <>
                        <ButtonsContainer>{buttons}</ButtonsContainer>
                        {children}
                    </>
                );
            }
            ```
        -   개발자 도구에서 확인해보면, ```<menu>```가 버튼들을 감싸고 있는 것을 확인할 수 있다.
        -   menu 외에도 div, ul 등 다른 내장 식별자도 사용 가능하다.
    -   더 빠른 지름길: 처음부터 속성 명을 대문자로 시작하도록 작성한다. (동일한 결과가 뜬다!)
        ```jsx
        export default function Tabs({ children, buttons, ButtonsContainer }) {
            //   const ButtonsContainer = buttonsContainer;
            // 처음부터 대문자로 시작하는 속성명을 써서 바로 활용할 수도 있다.
            return (
                <>
                    <ButtonsContainer>{buttons}</ButtonsContainer>
                    {children}
                </>
            );
        }
        ```
        ```jsx
        <Tabs
            ButtonsContainer="menu" // 밖의 속성명도 대문자로 바꿔준다.
            // 중략
        >
            {tabContent}
        </Tabs>
        ```

### 기본 Prop(속성) 값 설정

-   “menu”를 default(기본)값으로 설정해두면 Tabs가 더욱 간략해진다.
    -   ButtonsContainer = “menu” 혹은 ButtonsContainer = Section 과 같은 형태로 작성한다. 만약 커스텀된 Section을 기본값으로 설정할 경우 해당 함수를 import해줘야 한다.
    ```jsx
    <Tabs
    // ButtonsContainer="menu"   밖에서 이 속성을 작성할 필요가 없어진다.
    // 중략
    >
        {tabContent}
    </Tabs>
    ```
    ```jsx
    export default function Tabs({ children, buttons, ButtonsContainer = 'menu' }) {
        // "menu" 대신 (함수를 import한다는 전제 하에) Section으로 작성할 수도 있다.
        return (
            <>
                <ButtonsContainer>{buttons}</ButtonsContainer>
                {children}
            </>
        );
    }
    ```

### 모든 콘텐츠가 컴포넌트에 보관될 필요가 없는 이유

-   정적인 마크업: props나 state에 의해 변하지 않는다.
    -   따라서, 정적인 마크업들까지 컴포넌트 안에 넣지 않아도 된다.
    ```jsx
    // index.html

    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <link rel="icon" type="image/svg+xml" href="/game-logo.png" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>React Tic-Tac-Toe</title>
      </head>
      <body>
    		// 정적 마크업 시작되는 부분
        <header>
          <img src="" alt="" />
          <h1>Tic-Tac-Toe</h1>
        </header>
    		// 정적 마크업 끝나는 부분
        <div id="root"></div>
        <script type="module" src="/src/index.jsx"></script>
      </body>
    </html>
    ```
-   public 폴더 안의 이미지 파일들은 index.html과 함께 사이트 방문자들에게 공유된다.
    -   따로 경로 지정이 필요 없고, 이름만으로도 참조가 가능하다.
    ```jsx
    <header>
        <img src="game-logo.png" alt="Hand-drawn tic tac toe game board" />
        <h1>Tic-Tac-Toe</h1>
    </header>
    ```

### 세부 과정: 이미지 저장소는 public/ VS assets

-   public/ 폴더
    -   이 폴더에 저장된 이미지는 index.html 또는 index.css 내에서 직접 참조 가능하다.
    -   프로젝트 개발 서버 및 빌드 프로세스에 의해 공개적으로 제공되기 때문.
    -   브라우저 내에서 직접 방문 가능, 다른 파일에 의해 요청되는 것도 가능.
    -   예를 들어, localhost:5173/image.jpg를 불러오면 public 폴더 안의 해당 이미지를 볼 수 있다.
-   src/assets/ 폴더
    -   공개적으로 제공되지 않아 웹사이트 방문자가 접근 불가능하다.
    -   예를 들어, localhost:5173/image.jpg를 불러오려고 하면 오류가 발생한다.
    -   대신, 코드 파일에서 사용 가능.
        -   코드 파일에 가져온 이미지는 빌드 프로세스에 의해 웹사이트에 제공하기 직전에 public/ 폴더에 삽입되게 한다.
        -   이 이미지는 참조 위치에서 자동으로 링크가 생성되어 사용된다.
-   정리:
    -   빌드 프로세스에 의해 처리되지 않는 이미지 = public/ 폴더
        -   예를 들어, index.html 파일, favicon 이미지 등
    -   컴포넌트 내에서 사용되는 이미지 = src/ 폴더 (예를 들어, src/assets/ 폴더)

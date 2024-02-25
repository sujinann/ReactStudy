# CSS 파일과 CSS 클래스를 사용한 동적 및 조건적 스타일링 ~ 재사용 가능 컴포넌트 생성 및 컴포넌트 조합

### CSS 파일과 CSS 클래스를 사용한 동적 및 조건적 스타일링

- 동적 인라인 스타일은 필요가 없으므로 주석 처리 후 기존의 클래스명 및 독립된 CSS 파일을 사용하도록 하자.
    - 삼항 연산자로 조건 설정을 할 때에는 undefined를 추가해야 한다.
    
    ```jsx
    # AuthInputs.jsx
    
     <input type="email"
            // style={{ backgroundColor: emailNotValid ? "#fed2d2" : "#d1d5db" }}
            className={emailNotValid ? "invalid" : undefined}
            onChange={(event) => handleInputChange("email", event.target.value)}
            />
    ```
    
    - 만약 && 단축 방법으로 작성한다면
        - emailNotValid가 false인 경우 false를 className으로 설정하게 된다.
        - 클래스가 유효하지 않은 값으로 인식되어 콘솔에 오류가 뜨게 된다.
    
    ```jsx
    className={emailNotValid && "invalid"}
    ```
    
    ![캡처1](https://github.com/serethia/streamlit_docs/assets/137035446/9e1497b8-9c49-4e42-969c-f513999dd163)
    
- 만약 아래의 클래스를 조건 설정 없이 그대로 적용한다면
    - label과 invalid를 둘 다 작성해줘야 한다.
    - invalid 클래스를 작성했기 때문에 EMAIL이라는 label이 항상 빨간색으로 나타나게 된다.
    
    ```jsx
    # index.css
    
    .label.invalid {
      color: #f87171;
    }
    ```
    
    ```jsx
    # AuthInputs.jsx
    
    <label className="label invalid">Email</label>
    ```
    
    ![캡처2](https://github.com/serethia/streamlit_docs/assets/137035446/5032693a-419a-408a-960e-b981bcf6181f)
    
- 유효하지 않은 값인 조건에서만 invalid 스타일이 적용되게 하고 싶다면
    - className의 값을 동적으로 하기 위해 중괄호 안에 템플릿 리터럴(백틱들)로 설정한다.
    - 동적 값을 삽입할 때는 템플릿 리터럴 안에 ```${}```을 집어넣는다.
    - 영구적으로 적용할 스타일의 클래스는 ```${}``` 없이 그대로 작성한다.
    - ```${}``` 안에 삼항연산자로 조건을 설정해서 그에 따른 클래스 적용 여부를 작성한다.
    - 유효하지 않은 값을 입력한 후 SIGN IN 버튼을 누르면 EMAIL 레이블이 빨간색으로 변한다.
    
    ```jsx
    <label className={`label ${emailNotValid ? 'invalid' : ''}`}>Email</label>
    ```
    
    ![캡처3](https://github.com/serethia/streamlit_docs/assets/137035446/b7d76a7b-104c-4e5e-b10f-f85d36656709)
    

### 코딩 연습 3: 동적 CSS 클래스

- 결과 성공 & 작성한 코드
    
    ![코딩 연습 3](https://github.com/serethia/streamlit_docs/assets/137035446/e98e38c7-0a39-47ac-94c9-386eebb8adb2)
    
    ![코딩 연습 3 코드](https://github.com/serethia/streamlit_docs/assets/137035446/06995fd6-ff7b-424b-82ea-0bbb76fce87c)
    

- 해답에 나온 설명 (내 코드와의 차이점 비교)
    - 해답에서는 function을 따로 빼서 작성하고 onClick에 함수명만 작성하는 방식 사용.
    - 내 코드에서는 onClick에 함수를 전부 작성하는 방식 사용.
        
        ![코딩 연습 3 해답 코드_차이점](https://github.com/serethia/streamlit_docs/assets/137035446/8a01f574-9c82-45f0-88df-68a97b4ddcff)
        
    

### CSS 모듈로 CSS 규칙 스코핑하기

- CSS 모듈:
    - 바닐라 CSS로 작성한 코드와 규칙을 특정한 곳에 스코프 지정 가능
    - 기본 브라우저나 자바스크립트 기능이 아님
    - 빌드 도구가 CSS 클래스명을 변환하고 파일당 고유한 클래스명만을 사용
- 만약 paragraph로 클래스명을 바꿔 각각 작성해주면 스타일이 지정되어 버린다.
    - 헤더와 EMAIL 레이블이 모두 중앙 정렬이 되어버림

```jsx
# Header.css

.paragraph {
  text-align: center;
  color: #a39191;
  margin: 0;
}
```

```jsx
# Header.jsx

<p className="paragraph">A community of artists and art-lovers.</p>
```

```jsx
# AuthInputs.jsx

<p className="paragraph">
```

![캡처4](https://github.com/serethia/streamlit_docs/assets/137035446/cd7dec90-7e45-452a-87cc-6374f51bbfa7)

- Header.css를 Header.module.css로 파일명을 변경하자.
    - 모듈에서 import할 경우 from을 작성해주고 classes, styles 등 임의의 클래스명을 붙여준다.
    - 개발자 도구에서 확인해보면 지정하지 않은 클래스명으로 변환되어 있음을 확인 가능
    
    ```jsx
    # Header.jsx
    
    import classes from "./Header.module.css";
    
    <p className={classes.paragraph}>
    ```
    
    ![캡처5](https://github.com/serethia/streamlit_docs/assets/137035446/d4dec0bd-2f25-4b43-ac8c-52f46b48e71f)
    
    - 개발자 도구의 헤더 안의 text/css 타입의 스타일을 들어가보면 특수한 스타일 클래스명이 작성되어 있음을 확인 가능
        
        ![캡처6](https://github.com/serethia/streamlit_docs/assets/137035446/a1348b52-f70c-4877-91cd-bbf57e8c0b3e)
        
- CSS 모듈의 장단점
    - 장점
        - CSS 코드와 JSX 코드가 독립되어 있음
        - CSS로 작성 가능
        - CSS 코드 작성자와 JSX 코드 작성자의 역할 분담 가능
        - CSS 클래스가 충돌 없이 스코프가 가능해짐
    - 단점
        - CSS를 알아야 함
        - 작고 많은 CSS 파일들을 프로젝트에 생성해야 할 수 있음
            - 컴포넌트마다 그에 해당하는 CSS 모듈 파일을 생성한다고 가정
            

### “Styled Components” 소개 (서드 파티 패키지)

- Styled Components:
    - CSS 파일 혹은 인라인 스타일 정의 없이 CSS 규칙이나 스타일 적용 가능
    - 별도의 설치 필요 (로컬)
    
    ```jsx
    npm install styled-components
    ```
    
    - 의존성 추가 필요 (샌드박스)
        
        ![캡처7](https://github.com/serethia/streamlit_docs/assets/137035446/75cd92b6-e14a-4511-8399-045a1cd0015e)
        
- AuthInputs.jsx에 Styled Components를 적용해보자.
    - import할 때 from을 작성하고, styled를 가져온다.
    - (강의에서는 {styled}로 객체를 작성했지만 콘솔 오류가 떴음)
    - (검색 결과대로 중괄호를 없앤 styled로 수정하니 정상적으로 실행됨)
    - 참고한 공식 docs 사이트: [https://styled-components.com/docs/basics](https://styled-components.com/docs/basics)
    
    ```jsx
    // 강의대로 {styled}로 작성했을 때 뜨는 콘솔 오류 : export를 제공하지 않음
    
    Uncaught SyntaxError: The requested module '/node_modules/.vite/deps/styled-components.js?v=20ec3044' does not provide an export named 'styled'
    ```
    
    ```jsx
    import styled from "styled-components";
    ```
    
    - styled를 쓰고 그 안에 매핑이 가능한 html, div 등의 속성을 가져와 작성한다.
        - 이렇게 작성하면 해당 속성을 하나의 컴포넌트로 만들어 스타일 지정 가능
    - 해당 속성 뒤에 백틱 2개(템플릿 리터럴)를 작성한다.
    
    ```jsx
    styled.div``;
    ```
    
    - Tagged-template:
        - 정규 자바스크립트 기능.
        - 함수처럼 템플릿 리터럴을 입력값으로 받는다.
        - 문자열을 깨지 않고 여러 줄의 코드를 입력할 수 있다.
        - 카멜 케이스로 스타일을 작성할 필요가 없다. (예) flexDirection 등)
    
    ```jsx
    const ControlContainer = styled.div`
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
      margin-bottom: 1.5rem;
    `;
    ```
    
    ```jsx
    // 수정 전
    
    <div className="controls">
    </div>
    
    // 수정 후
    
    <ControlContainer>
    </ControlContainer>
    ```
    
    - 다른 컴포넌트에서도 스타일을 적용할 거라면 따로 분리된 파일을 만들어 작성하는 게 좋다.
    - 개발자 도구를 보면 styled components를 적용한 클래스명 또한 변환되어 있음 (그리고 div로 설정했기 때문에 태그는 div로 되어있음)
        
        ![캡처8](https://github.com/serethia/streamlit_docs/assets/137035446/b1d3e189-f9e2-4aed-b406-ed507f9864fe)
        
- 적용 후 현재 상황:
    
    ![캡처9](https://github.com/serethia/streamlit_docs/assets/137035446/0c5f9056-5dcb-4af7-8254-bfcd49f4cc27)
    

### Styled Components로 유동적 컴포넌트 생성

- label 스타일 클래스도 styled components로 지정해주자.

```jsx
const Label = styled.label`
  display: block;
  margin-bottom: 0.5rem;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #6b7280;
`;

          <Label className={`label ${emailNotValid ? "invalid" : ""}`}>
            Email
          </Label>

          <Label className={`label ${passwordNotValid ? "invalid" : ""}`}>
            Password
          </Label>
```

- label에는 스타일이 잘 적용되었지만, styled components에 추가 작성된 className 때문에 input에는 스타일이 제대로 작동하지 않는다.
    - 추가 설정한 모든 속성은 해당 컴포넌트의 기본 jsx 요소로 전달하게 됨
    - 내장 ```<label></label>```을 생성해 그 내부 레이블에 className 속성을 전달하게 되어버림
        
        ![캡처10](https://github.com/serethia/streamlit_docs/assets/137035446/e9e2940a-b6f7-4b3d-b54e-aedd79d8822e)
        
- input에도 styled components를 적용해주자.
    - type, className(삼항 연산자 조건), onChange 이벤트는 삭제할 필요 없다.
    - input에도 정상적으로 스타일이 적용되었고, 삼항 연산자 조건 및 이벤트도 정상 작동한다.
    
    ```jsx
    const Input = styled.input`
      width: 100%;
      padding: 0.75rem 1rem;
      line-height: 1.5;
      background-color: #d1d5db;
      color: #374151;
      border: 1px solid transparent;
      border-radius: 0.25rem;
      box-shadow:
        0 1px 3px 0 rgba(0, 0, 0, 0.1),
        0 1px 2px 0 rgba(0, 0, 0, 0.06);
    `;
    
              <Input
                type="email"
                // style={{ backgroundColor: emailNotValid ? "#fed2d2" : "#d1d5db" }}
                className={emailNotValid ? "invalid" : undefined}
                onChange={(event) => handleInputChange("email", event.target.value)}
              />
    
              <Input
                type="password"
                className={passwordNotValid ? "invalid" : undefined}
                onChange={(event) =>
                  handleInputChange("password", event.target.value)
                }
              />
    ```
    
    ![캡처11](https://github.com/serethia/streamlit_docs/assets/137035446/3ecc399e-6d06-4b77-8dbe-9595530d6c3e)
    

### Styled Components로 동적 및 조건적 스타일링

- 동적 Styled Components 적용
    - 지난 번에 작성했던 className 속성을 모두 제거한다.
    - styled components에 추가된 모든 속성이 styled components가 자동 실행될 때 인식된다. (tagged templates 참고)
    - label에 동적으로 작성해두면 invalid의 T/F 여부에 따른 스타일 변화 가능
    
    ```jsx
    const Label = styled.label`
      display: block;
      margin-bottom: 0.5rem;
      font-size: 0.75rem;
      font-weight: 700;
      letter-spacing: 0.1em;
      text-transform: uppercase;
      color: ${({ invalid }) => (invalid ? "#f87171" : "#6b7280")};
    `;
    
    // color: ${(props) => (props.invalid ? "#f87171" : "#6b7280")}; 로도 작성 가능
    
    const emailNotValid = submitted && !enteredEmail.includes("@");
    
              <Label invalid={emailNotValid}>Email</Label>
              <Input
                type="email"
                // style={{ backgroundColor: emailNotValid ? "#fed2d2" : "#d1d5db" }}
                // className={emailNotValid ? "invalid" : undefined}
                onChange={(event) => handleInputChange("email", event.target.value)}
              />
    ```
    
    ![캡처12](https://github.com/serethia/streamlit_docs/assets/137035446/c47bee23-0286-4c64-bcb3-ca8a005b7535)
    
    - input에도 invalid 속성을 추가 작성한 뒤, 동적 styled components를 적용해보자.
    
    ```jsx
    const Input = styled.input`
      width: 100%;
      padding: 0.75rem 1rem;
      line-height: 1.5;
      background-color: ${({ invalid }) => (invalid ? "#fed2d2" : "#d1d5db")};
      color: ${({ invalid }) => (invalid ? "#ef4444" : "#374151")};
      border: 1px solid ${({ invalid }) => (invalid ? "#f73f3f" : "transparent")};
      border-radius: 0.25rem;
      box-shadow:
        0 1px 3px 0 rgba(0, 0, 0, 0.1),
        0 1px 2px 0 rgba(0, 0, 0, 0.06);
    `;
    
              <Label invalid={emailNotValid}>Email</Label>
              <Input
                type="email"
                invalid={emailNotValid}
                // style={{ backgroundColor: emailNotValid ? "#fed2d2" : "#d1d5db" }}
                // className={emailNotValid ? "invalid" : undefined}
                onChange={(event) => handleInputChange("email", event.target.value)}
              />
    ```
    
    ![캡처13](https://github.com/serethia/streamlit_docs/assets/137035446/59543ebf-eb2f-461c-90b8-82ed860f6a78)
    
    - 비밀번호도 styled components를 적용해준다.
    
    ```jsx
              <Label invalid={passwordNotValid}>Password</Label>
              <Input
                type="password"
                invalid={passwordNotValid}
                // className={passwordNotValid ? "invalid" : undefined}
                onChange={(event) =>
                  handleInputChange("password", event.target.value)
                }
              />
    ```
    
    - 기본 내장된 속성과 충돌되지 않도록 앞에 $ 기호를 작성해야 한다. (예) invalid ⇒ $invalid)
    
    ```jsx
    const Label = styled.label`
      display: block;
      margin-bottom: 0.5rem;
      font-size: 0.75rem;
      font-weight: 700;
      letter-spacing: 0.1em;
      text-transform: uppercase;
      color: ${({ $invalid }) => ($invalid ? "#f87171" : "#6b7280")};
    `;
    
    const Input = styled.input`
      width: 100%;
      padding: 0.75rem 1rem;
      line-height: 1.5;
      background-color: ${({ $invalid }) => ($invalid ? "#fed2d2" : "#d1d5db")};
      color: ${({ $invalid }) => ($invalid ? "#ef4444" : "#374151")};
      border: 1px solid ${({ $invalid }) => ($invalid ? "#f73f3f" : "transparent")};
      border-radius: 0.25rem;
      box-shadow:
        0 1px 3px 0 rgba(0, 0, 0, 0.1),
        0 1px 2px 0 rgba(0, 0, 0, 0.06);
    `;
    
              <Label $invalid={emailNotValid}>Email</Label>
              <Input
                type="email"
                $invalid={emailNotValid}
                onChange={(event) => handleInputChange("email", event.target.value)}
              />
    
              <Label $invalid={passwordNotValid}>Password</Label>
              <Input
                type="password"
                $invalid={passwordNotValid}
                onChange={(event) =>
                  handleInputChange("password", event.target.value)
                }
              />
    ```
    

### Styled Components: 가상 선택자, 중첩 규칙 & 미디어 쿼리

- 중첩 선택자
    - 모든 자식 속성들(img, h1, p)에 적힌 부모 요소 header를 모두 &로 바꾼다.
    - 이 때, 공백을 꼭 넣고 작성해야 자식 요소로 넘어갈 수 있다.
    - media 쿼리의 기능: 화면 비율에 따라 다른 스타일(여기서는 margin-bottom)을 적용
    
    ```jsx
    # Header.jsx
    
    import logo from "../assets/logo.png";
    // import classes from "./Header.module.css";
    import styled from "styled-components";
    
    const StyledHeader = styled.header`
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      margin-top: 2rem;
      margin-bottom: 2rem;
    
      & img {
        object-fit: contain;
        margin-bottom: 2rem;
        width: 11rem;
        height: 11rem;
      }
    
      & h1 {
        font-size: 1.5rem;
        font-weight: 600;
        letter-spacing: 0.4em;
        text-align: center;
        text-transform: uppercase;
        color: #9a3412;
        font-family: "Pacifico", cursive;
        margin: 0;
      }
    
      & p {
        text-align: center;
        color: #a39191;
        margin: 0;
      }
    
      @media (min-width: 768px) {
        & {
          margin-bottom: 4rem;
        }
    
        // margin-bottom: 4rem; 으로만 적어도 무방
    
        & h1 {
          font-size: 2.25rem;
        }
      }
    `;
    
    export default function Header() {
      return (
        <StyledHeader>
          <img src={logo} alt="A canvas" />
          <h1>ReactArt</h1>
          <p>A community of artists and art-lovers.</p> {/* className 삭제 */}
        </StyledHeader>
      );
    }
    ```
    
- pseudo 선택자
    - 공백 없이 &:를 작성해야 해당 속성(여기서는 button) 자체에 스타일을 적용해줄 수 있다.
    
    ```jsx
    # AuthInputs.jsx
    
    const Button = styled.button`
      padding: 1rem 2rem;
      font-weight: 600;
      text-transform: uppercase;
      border-radius: 0.25rem;
      color: #1f2937;
      background-color: #f0b322;
      border-radius: 6px;
      border: none;
    
      &: hover {
        background-color: #f0920e;
      }
    `;
    
            <Button onClick={handleLogin}>Sign In</Button>
            {/* className 삭제 */}
    ```
    

### 재사용 가능 컴포넌트 생성 및 컴포넌트 조합

- 해당 컴포넌트 내에서만 사용되는 styled components 스타일:
    - 그 컴포넌트에 직접 작성하는 편이 좋다.
- 다른 컴포넌트에서도 사용될 수 있는 스타일들은 따로 파일을 생성해서 작성하자.
    - Button.jsx, Input.jsx, Label.jsx(얘는 추후 삭제) 파일을 생성
    - Button에 관련된 styled components 스타일들을 Button.jsx로 옮긴다.
    - Button.jsx에서 styled를 import하고, Button을 export한다.
    - Button 스타일을 사용하는 파일에서 Button을 import한다.
    
    ```jsx
    # Button.jsx
    
    import styled from "styled-components";
    
    const Button = styled.button`
      padding: 1rem 2rem;
      font-weight: 600;
      text-transform: uppercase;
      border-radius: 0.25rem;
      color: #1f2937;
      background-color: #f0b322;
      border-radius: 6px;
      border: none;
    
      &: hover {
        background-color: #f0920e;
      }
    `;
    
    export default Button;
    
    ```
    
    ```jsx
    # AuthInputs.jsx
    
    import Button from "./Button.jsx";
    ```
    
    - Input과 Label에 관련된 스타일은 하나로 묶는 게 나아 보여 Input.jsx로 모두 옮긴다.
    - 이번에는 함수를 써서 export한다.
    - 이 함수를 사용하는 파일에서는 필요없는 코드(Label, p 등)를 지우고 함수 형식에 맞게 Input 속성을 수정한다.
        - 속성 $invalid를 invalid로 수정
        - Label 태그 제거 후 label 속성 추가
        - p 태그 제거
    
    ```jsx
    # Input.jsx
    
    import styled from "styled-components";
    
    const Label = styled.label`
      display: block;
      margin-bottom: 0.5rem;
      font-size: 0.75rem;
      font-weight: 700;
      letter-spacing: 0.1em;
      text-transform: uppercase;
      color: ${({ $invalid }) => ($invalid ? "#f87171" : "#6b7280")};
    `;
    
    const Input = styled.input`
      width: 100%;
      padding: 0.75rem 1rem;
      line-height: 1.5;
      background-color: ${({ $invalid }) => ($invalid ? "#fed2d2" : "#d1d5db")};
      color: ${({ $invalid }) => ($invalid ? "#ef4444" : "#374151")};
      border: 1px solid ${({ $invalid }) => ($invalid ? "#f73f3f" : "transparent")};
      border-radius: 0.25rem;
      box-shadow:
        0 1px 3px 0 rgba(0, 0, 0, 0.1),
        0 1px 2px 0 rgba(0, 0, 0, 0.06);
    `;
    
    export default function CustomInput({ label, invalid, ...props }) {
      return (
        <p>
          <Label $invalid={invalid}>{label}</Label>
          <Input $invalid={invalid} {...props} />
        </p>
      );
    }
    
    ```
    
    ```jsx
    # AuthInputs.jsx
    
    import Input from "./Input.jsx"; // default는 CustomInput이지만, 이름 변경 가능
    
          <ControlContainer>
            <Input
              type="email"
              invalid={emailNotValid}
              label="Email"
              onChange={(event) => handleInputChange("email", event.target.value)}
            />
            <Input
              type="password"
              invalid={passwordNotValid}
              label="Password"
              onChange={(event) =>
                handleInputChange("password", event.target.value)
              }
            />
          </ControlContainer>
    ```
    
    - 재사용이 가능한 코드는 항상 외부로 분리하는 작업이 필요하다.
- Styled Components의 장단점
    - 장점
        - 상대적으로 빠르고 쉽게 추가 가능
        - 리액트에서 **생각**하면서 스타일 함수를 구성 작업할 수 있음
        - 스타일이 자동으로 컴포넌트에 스코프됨 ⇒ CSS 스타일이나 규칙의 충돌을 피함
    - 단점
        - CSS를 알아야 함
        - React 코드와 CSS 코드의 분리가 모호해짐
        - 상대적으로 작고 많은 래퍼(wrapper) 컴포넌트가 생김
            - 물론 리액트의 목표는 컴포넌트의 생성 및 재사용이기 때문에 단점으로 보기 어려움
            - 다만, 스타일링 목적으로만 여러 컴포넌트를 생성해야 하는 점이 불편할 수 있음

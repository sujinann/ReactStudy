# 모듈 소개 & 프로젝트 시작하기 ~ 동적 및 조건적 Inline(인라인) 스타일

### 모듈 소개 & 프로젝트 시작하기

- 학습 예정:
    - 바닐라 CSS로 리액트 앱을 스타일링하는 법
    - 추가 패키지(예를 들어, CSS 모듈 등)을 사용해 특정 컴포넌트를 스타일링하는 법
    - Styled Components라고 불리는 CSS-in-JS (자바스크립트 스타일링의 CSS 패키지)에 대해
    - Tailwind CSS로 스타일링하는 법
    - 정적 스타일 & 동적(조건부적) 스타일 적용

### CSS 코드 여러 파일에 분단하기

- index.css 파일 생성 ⇒ 해당 css 파일을 import
    - head 섹션 안에 vite를 통해 동적으로 css 스타일 주입 가능
    - 여러 css 파일을 만들어도 무방
    - 예) 헤더에 관련된 css 코드만 따로 떼어 Header.css 파일에 분리해서 적용할 수도 있음 (Header.css도 import해줄 것!)
        
        ```jsx
        # main.jsx
        
        import "./index.css";
        import "./components/Header.css"; // 생성한 css 파일을 import해준다.
        ```
        
        - head 섹션 안에 style 태그가 2개로 증가하나, 렌더링되는 결과는 동일

### 바닐라 CSS로 리액트 앱 스타일링하기 - 장.단점

- 장점:
    - vanilla css로 작업하고 싶거나, 다른 개발자나 디자이너가 css 파일에 작업하고 있다면 css 파일을 전달하기에 용이
    - css 코드에서 작업하는 사람과, 컴포넌트 및 jsx 코드에서 작업하는 사람을 분리 가능
    - vanilla css 코드 작성은 특별한 관례가 없어 간단
- 단점:
    - css를 알아야 함
    - 컴포넌트에 스코핑되어 있지 않음 ⇒ 다른 컴포넌트 간의 스타일 충돌이 있을 수 있음

### 바닐라 CSS 스타일이 컴포넌트에 스코핑되지 않는 이유

- vanilla css 스타일은 import한 컴포넌트에만 스코핑되지 않음
    
    ```jsx
    # AuthInputs.jsx
    
    <p>Some text</p> // 테스트용으로 p 태그 추가
    ```
    
    ```jsx
    # Header.css 수정 전
    
    header p {
      text-align: center;
      color: #a39191;
      margin: 0;
    }
    
    # Header.css 수정 후
    
    p {
      text-align: center;
      color: red;
      margin: 0;
    }
    ```
    
    - 다른 컴포넌트(AuthInputs.jsx)에도 스타일이 적용되어 빨간 글자로 변해버림
        
        ![캡처1](https://github.com/serethia/Computer-Science/assets/137035446/1a476a29-4bf2-4da2-95e6-c0158ab5f793)
        
    - 전체 페이지에 스타일이 적용되었기 때문

### Inline(인라인) 스타일로 리액트 앱 스타일링하기

- 컴포넌트에 스코핑되지 않는 문제를 해결하려면? ⇒ 인라인 스타일로 직접 적용
    - jsx 요소에 style 속성을 설정
        - String value로는 매핑할 수 없음 (예) ```style = "color: red"```는 적용 불가)
        - 스타일 값의 객체를 키-값 쌍으로 작성(값은 string으로)한 후 중괄호로 감싸 동적 값 ```{}```으로 전달
            
            ```jsx
            # Header.jsx
            
            <p style={{ color: "red" }}>A community of artists and art-lovers.</p>
            ```
            
        - 스타일 작성의 특수한 이중 중괄호 문법이 아니다! ⇒ 표준 동적 값 문법으로 객체를 값으로 전달
            - 원하는 곳에만 빨간색이 적용되었음을 볼 수 있음
                
                ![캡처2](https://github.com/serethia/Computer-Science/assets/137035446/3820d041-d998-4f19-8814-46c883a61b6d)
                
        - text-align처럼 자바스크립트에서 유효하지 않은 속성명은
            - 작은 따옴표로 묶거나
            - dash(```-```) 기호를 빼고 Camel Case로 표기
                
                ```jsx
                # Header.jsx 수정 전
                
                <p style={{ color: "red", text-align }}>A community of artists and art-lovers.</p>
                
                # Header.jsx 수정 후 (작은 따옴표)
                
                <p style={{ color: "red", "text-align": "left" }}>A community of artists and art-lovers.</p>
                
                # Header.jsx 수정 후 (카멜 표기법)
                
                <p style={{ color: "red", textAlign: "left" }}>A community of artists and art-lovers.</p>
                ```
                
        - 그러나 text-align 스타일이 적용되지는 않았음
            
            ![캡처3](https://github.com/serethia/Computer-Science/assets/137035446/13cf8321-d1b1-4f04-8b1d-31ad7b366c8e)
            
            - 헤더가 flexbox를 사용한 후 요소들을 중간 정렬(```align-items```)하기 때문에 효력이 없기 때문
            - Header.css의 중간 정렬 코드를 비활성화하면 원하던 대로 좌측 정렬로 바뀜
                
                ```jsx
                # Header.css 수정
                
                header {
                  display: flex;
                  flex-direction: column;
                  /* align-items: center; */
                  justify-content: center;
                  margin-top: 2rem;
                  margin-bottom: 2rem;
                }
                ```
                
            
            ![캡처4](https://github.com/serethia/Computer-Science/assets/137035446/d187e769-288a-424d-855c-aee165b124cb)
            
- 인라인 스타일의 장단점
    - 장점:
        - 쉽게 추가할 수 있음
        - 사용자가 추가하는 요소에만 스타일이 영향을 미침
        - 동적(조건부적)으로 정확한 스타일링 설정이 용이
    - 단점:
        - css를 알아야 함
        - 모든 요소를 개별적으로 스타일링해야 함
        - css와 jsx 코드에 구분이 없음 ⇒ jsx 코드 안에 css 코드가 들어감
        

### 동적 및 조건적 Inline(인라인) 스타일

- 동적(조건부적) 스타일링이 용이
    - 유효하지 않은 이메일 값을 작성했을 경우의 스타일을 설정해보자.
    - 삼항 연산자를 이용해 해당 조건에 부합할 경우에만 스타일 적용 가능
        
        ```jsx
        <input type="email"
        style={{ backgroundColor: emailNotValid ? "#fed2d2" : "#d1d5db" }}
        // className={emailNotValid ? "invalid" : undefined}
        onChange={(event) => handleInputChange("email", event.target.value)}
        />
        ```
        
        ![캡처5](https://github.com/serethia/Computer-Science/assets/137035446/cff7ce95-ddd2-4471-9013-bf9bb1f8eb92)

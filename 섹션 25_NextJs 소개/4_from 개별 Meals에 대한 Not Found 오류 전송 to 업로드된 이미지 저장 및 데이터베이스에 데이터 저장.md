# 개별 Meals에 대한 Not Found 오류 전송 ~ 업로드된 이미지 저장 및 데이터베이스에 데이터 저장

### 개별 Meals에 대한 Not Found 오류 전송

- 존재하지 않는 음식을 찾았을 경우 오류 페이지가 뜨는 문제
    - undefined에서 instructions에 접근했기 때문 (meal.instrunctions)
- 실습
    - meal-slug 폴더의 page.js 파일에 if 조건문 추가
        - Next.js에서 제공되는 notFound 함수 활용
            - notFound 함수: 해당 컴포넌트의 실행 중지 및 제일 가까운 not-found 혹은 오류 화면을 보여줌
            - meals 폴더에 not-found.js 파일 추가
    
    ```jsx
    // page.js
    
    import { notFound } from 'next/navigation';
    
    export default function MealDetailsPage({ params }) {
        const meal = getMeal(params.mealSlug);
    
        if (!meal) {
            notFound();
        }
    ```
    
    ```jsx
    // not-found.js
    
    export default function NotFound() {
        return (
            <main className="not-found">
                <h1>Meal Not found</h1>
                <p>Unfortunately, we could not find the requested page or meal data.</p>
            </main>
        );
    }
    
    ```
    
    - 실행: 존재하지 않는 음식을 url에 입력하면 Meal Not Found 페이지가 정상적으로 뜸

### ‘Share Meal’ 양식 사용에 대한 기초

- 자신의 메뉴를 목록에 추가할 수 있도록 구현해보자
- 실습
    - 첨부파일을 share 폴더의 page.js로 사용
    
    ```jsx
    // page.js
    
    import classes from './page.module.css';
    
    export default function ShareMealPage() {
        return (
            <>
                <header className={classes.header}>
                    <h1>
                        Share your <span className={classes.highlight}>favorite meal</span>
                    </h1>
                    <p>Or any other meal you feel needs sharing!</p>
                </header>
                <main className={classes.main}>
                    <form className={classes.form}>
                        <div className={classes.row}>
                            <p>
                                <label htmlFor="name">Your name</label>
                                <input type="text" id="name" name="name" required />
                            </p>
                            <p>
                                <label htmlFor="email">Your email</label>
                                <input type="email" id="email" name="email" required />
                            </p>
                        </div>
                        <p>
                            <label htmlFor="title">Title</label>
                            <input type="text" id="title" name="title" required />
                        </p>
                        <p>
                            <label htmlFor="summary">Short Summary</label>
                            <input type="text" id="summary" name="summary" required />
                        </p>
                        <p>
                            <label htmlFor="instructions">Instructions</label>
                            <textarea id="instructions" name="instructions" rows="10" required></textarea>
                        </p>
                        IMAGE PICKER
                        <p className={classes.actions}>
                            <button type="submit">Share Meal</button>
                        </p>
                    </form>
                </main>
            </>
        );
    }
    
    ```
    
    - share 폴더에 첨부된 page.module.css 파일 추가

### 커스텀 이미지 피커(선택 도구) 입력 컴포넌트에 대한 기초

- IMAGE PICKER 자리에 이미지 피커 컴포넌트를 추가해보자
- 실습
    - components 쪽 meals 폴더에 image-picker.js 파일과 첨부된 image-picker.module.css 파일 추가
    - image-picker.js에 필요한 코드 작성
        - htmlFor : 해당 label을 input에 연결 후 image ID에 연결시킴
        - accept : 받고 싶은 파일 타입 지정
        - name : 추후 업로드된 이미지를 추출하기 위해 사용
            - prop으로 받아와서 더 자유롭게 설정도 가능 (단, label의 htmlFor 값 == input의 id 값 == input의 name 값으로 통일할 것!)
    
    ```jsx
    // image-picker.js
    
    import classes from './image-picker.module.css';
    
    export default function ImagePicker({ label, name }) {
        return (
            <div className={classes.picker}>
                <label htmlFor={name}>{label}</label>
                <div className={classes.controls}>
                    <input type="file" id={name} accept="image/png, image/jpeg" name={name} />
                </div>
            </div>
        );
    }
    
    ```
    
    - share 폴더의 page.js에서 IMAGE PICKER 자리에 ImagePicker 컴포넌트 호출
    
    ```jsx
    // page.js
    
    import ImagePicker from '@/components/meals/image-picker';
    
                        <ImagePicker />
    ```
    
    - 못생긴 버튼을 보이지 않게 (= hidden 처리) 스타일 클래스를 적용해보자
    
    ```jsx
    // image-picker.js
    
                    <input className={classes.input} type="file" id={name} accept="image/png, image/jpeg" name={name} />
    ```
    
    - 이미지를 첨부할 수 있도록 input에 연결된 버튼을 추가 구현해보자 (맨 아래 코드 참고)
        - type=”button”으로 해야 주변 form 제출을 방지할 수 있음
        - onClick에 추가 작성한 handlePickClick 함수를 연결
        - onClick을 작성하면 서버 컴포넌트여서 작동하지 않는 오류 발생
            - ‘use client’ 지시어 추가
        - useRef를 추가 작성해 input의 ref 값에 넣어 HTML 요소로 연결
            - current 작성 필요 : 실제로 연결된 요소와 객체에 접근 가능하게 함
    - image-picker.js 파일 원본:
    
    ```jsx
    // image-picker.js
    
    'use client';
    
    import classes from './image-picker.module.css';
    import { useRef } from 'react';
    
    export default function ImagePicker({ label, name }) {
        const imageInput = useRef();
    
        function handlePickClick() {
            imageInput.current.click();
        }
    
        return (
            <div className={classes.picker}>
                <label htmlFor={name}>{label}</label>
                <div className={classes.controls}>
                    <input
                        className={classes.input}
                        type="file"
                        id={name}
                        accept="image/png, image/jpeg"
                        name={name}
                        ref={imageInput}
                    />
                    <button className={classes.button} type="button" onClick={handlePickClick}>
                        Pick an Image
                    </button>
                </div>
            </div>
        );
    }
    
    ```
    

### 피커에 이미지 미리보기 추가

- 선택된 이미지의 미리보기를 구현해보자
- 실습
    - image-picker.js에 useState 상수를 추가 작성
    
    ```jsx
    // image-picker.js
    
    import { useRef, useState } from 'react';
    
        const [pickedImage, setPickedImage] = useState();
    ```
    
    - handleImageChange 함수 추가 작성
    
    ```jsx
    // image-picker.js
    
        function handlePickClick() {
            imageInput.current.click();
        }
    ```
    
    - input의 onChange 값에 handleImageChange를 넣음
        - handleImageChange는 자동으로 event 객체를 받아오게 됨
    
    ```jsx
    // image-picker.js
    
                    <input
                        className={classes.input}
                        type="file"
                        id={name}
                        accept="image/png, image/jpeg"
                        name={name}
                        ref={imageInput}
                        onChange={handleImageChange}
                    />
    ```
    
    - event.target.files[0] 상수를 작성해 event 객체에 들어 있는 files 성질에서 1번째 사진을 저장
        - 다중 파일 선택을 구현하고 싶다면: input에 multiple 속성을 추가
        - file이 없으면 return하도록 조건문 추가
        - setPickedImage(null)을 추가해 초기화시켜줘야 함
    
    ```jsx
    // image-picker.js
    
        function handleImageChange(event) {
            const file = event.target.files[0];
    
            if (!file) {
                return;
            }
        }
    ```
    
    - 미리보기를 위한 data URL로 변환시켜야 함
        - 자바스크립트에 내장된 FileReader 클래스 활용
        - readAsDataURL을 이용해 파일의 URL을 읽는다
            - readAsDataURL은 void를 반환하기 때문에 callback도 없음
            - onload로 파일을 먼저 불러오고 (readAsDataURL보다 위에 작성)
            - onload 함수 안에서 setPickedImage 값에 fileReader.result 값으로 갱신해준다
                - 첨부한 파일을 저장하게 됨
    
    ```jsx
    // image-picker.js
    
        function handleImageChange(event) {
            const file = event.target.files[0];
    
            if (!file) {
                return;
            }
    
            const fileReader = new FileReader();
            fileReader.onload = () => {
                setPickedImage(fileReader.result);
            };
            fileReader.readAsDataURL(file);
        }
    ```
    
    - 미리보기를 구현할 div를 작성
        - pickedImage가 없을 경우 화면에 띄울 p 태그 작성
        - pickedImage가 있을 경우 Image 컴포넌트로 화면에 띄움
            - src 값으로 pickedImage를 넣어줌 (pickedImage는 첨부된 이미지의 URL)
            - 파일의 크기를 모르기 때문에 fill 속성을 추가
    
    ```jsx
    // image-picker.js
    
                    <div className={classes.preview}>
                        {!pickedImage && <p>No image picked yet</p>}
                        {pickedImage && <Image src={pickedImage} alt="The image selected by the user." fill />}
                    </div>
    ```
    

### 이미지 피커 컴포넌트 개선하기

- 개선을 위해 추가 작성
    - handleImageChange 함수 안의 if문에 setPickedImage(null) 추가 작성
        - 미리보기된 이미지를 초기화하기 위함
    
    ```jsx
    // image-picker.js
    
            if (!file) {
                setPickedImage(null);
                return;
            }
    ```
    
    - input 태그에 required 속성 추가 작성
        - 필수로 첨부하지 않으면 form 제출 불가능
    
    ```jsx
    // image-picker.js
    
                    <input
                        className={classes.input}
                        type="file"
                        id={name}
                        accept="image/png, image/jpeg"
                        name={name}
                        ref={imageInput}
                        onChange={handleImageChange}
                        required
                    />
    ```
    

### 양식 제출 처리를 위한 서버 액션 소개 및 사용 방법

- form에 입력한 정보를 제출하고 저장할 수 있도록 구현해보자
- 실습
    - share 폴더의 page.js에서 form 태그에 직접 onSubmit를 작성하기보다는, form이 있는 컴포넌트에 함수를 만들어주자
    - 함수 안에 ‘use server’ 지시어 작성
        - use server : 서버에서만 실행되도록 보장해주는 Server Action을 생성
        - 함수 앞에 async도 붙여줘야 함
    
    ```jsx
    // page.js
    
        async function shareMeal() {
            'use server';
        }
    ```
    
    - form 태그에 action 속성을 추가 작성
        - 본래 action 값으로는 요청이 보내질 경로(path)가 들어갔었음
            - 예) action=”/some-path”
        - 여기서는 action 값으로 shareMeal 함수를 넣어줌
    
    ```jsx
    // page.js
    
                    <form className={classes.form} action={shareMeal}>
    ```
    
    - form 안의 input들에서 수집된 formData 객체가 shareMeal에 들어오게 됨
        - 객체 안에 각 값들을 가져오기
            - 예) title에 formData.get(’title’)을 해주면 name=”title”인 input의 값을 가져오게 됨
        - ImagePicker 컴포넌트에 name으로 image를 넣어주면 formData 객체가 image도 추출할 수 있게 됨
    
    ```jsx
    // page.js
    
        async function shareMeal(formData) {
            'use server';
    
            const meal = {
                title: formData.get('title'),
                summary: formData.get('summary'),
                instructions: formData.get('instructions'),
                image: formData.get('image'),
                creator: formData.get('name'),
                creator_email: formData.get('email'),
            };
        }
        
        // 아래는 name 참고용
        
                    <form className={classes.form} action={shareMeal}>
                        <div className={classes.row}>
                            <p>
                                <label htmlFor="name">Your name</label>
                                <input type="text" id="name" name="name" required />
                            </p>
                            <p>
                                <label htmlFor="email">Your email</label>
                                <input type="email" id="email" name="email" required />
                            </p>
                        </div>
                        <p>
                            <label htmlFor="title">Title</label>
                            <input type="text" id="title" name="title" required />
                        </p>
                        <p>
                            <label htmlFor="summary">Short Summary</label>
                            <input type="text" id="summary" name="summary" required />
                        </p>
                        <p>
                            <label htmlFor="instructions">Instructions</label>
                            <textarea id="instructions" name="instructions" rows="10" required></textarea>
                        </p>
                        <ImagePicker label="Your image" name="image" />
                        <p className={classes.actions}>
                            <button type="submit">Share Meal</button>
                        </p>
                    </form>
    ```
    
    - 실행 (+ 콘솔에 meal을 출력해보자)
        - 개발자 도구 콘솔에 아무것도 뜨지 않고 페이지 새로고침도 일어나지 않음
        - 서버 터미널에는 meal 내용이 뜸

### 개별 파일에 서버 액션 저장

- server actions를 다른 파일에 저장해보자 (분리)
- 실습
    - lib 폴더 안에 actions.js 파일을 생성
    - actions.js 맨 위에 ‘use server’ 지시어 작성
    
    ```jsx
    // actions.js
    
    'use server';
    ```
    
    - share 폴더의 page.js에서 shareMeal 함수를 잘라내 lib 폴더의 actions.js에 붙여넣기
        - 함수 안의 use server 삭제
        - 함수를 export
    
    ```jsx
    // actions.js
    
    'use server';
    
    export async function shareMeal(formData) {
        const meal = {
            title: formData.get('title'),
            summary: formData.get('summary'),
            instructions: formData.get('instructions'),
            image: formData.get('image'),
            creator: formData.get('name'),
            creator_email: formData.get('email'),
        };
    }
    
    ```
    
    - share 폴더의 page.js로 돌아가서 shareMeal을 호출한 뒤 사용
        - 여기서는 아니지만, 필요 시 ‘use client’를 추가할 수도 있음 (server actions를 분리했기 때문)
    
    ```jsx
    // page.js
    
    import { shareMeal } from '@/lib/actions';
    
                    <form className={classes.form} action={shareMeal}>
    ```
    

### XSS 보호를 위한 슬러그 생성 및 유저 입력 무결 처리하기

- 데이터베이스에 저장하게끔 해보자
- 실습
    - lib 폴더의 meals.js에서 meal을 저장할 saveMeal 함수를 추가 작성
    - saveMeal 안에 slug도 만들어줘야
        - SQL 쿼리문을 보면 slug가 존재하나, form에서 입력으로 받아오지 않기 때문
        - 실행 중지 후 추가 패키지 설치
            
            ```jsx
            npm install slugify xss
            ```
            
            - 크로스 사이트 스크립팅 (xss) 공격을 방어해줌
            - 필요한 이유: [mealslug] 폴더의 page.js에서 instructions를 HTML 형태로 출력하려 하기 때문 → xss에 취약함
            
            ```jsx
            // page.js (참고용)
            
                            <p
                                className={classes.instructions}
                                dangerouslySetInnerHTML={{
                                    __html: meal.instructions,
                                }}
                            ></p>
            ```
            
        - meals.js로 돌아와 두 패키지를 import해줌
        
        ```jsx
        // meals.js
        
        import slugify from 'slugify';
        import xss from 'xss';
        ```
        
        - saveMeal 함수 안에 title을 기반으로 slug을 생성하고 모든 문자를 소문자로 설정하는 lower: true를 작성
            - 이 때, const slug로 쓰기보다 meal.slug로 작성해 SQL문의 속성에 저장되도록 함
        
        ```jsx
        // meals.js
        
        export function saveMeal(meal) {
            meal.slug = slugify(meal.title, { lower: true });
        }
        ```
        
        - saveMeal 함수 안에 instructions를 인수로 받는 xss 함수를 작성해 검열하도록 함
            - 이 때, const instructions로 쓰기보다 meal.instructions로 작성해 SQL문의 속성에 저장되도록 함
        
        ```jsx
        // meals.js
        
        export function saveMeal(meal) {
            meal.slug = slugify(meal.title, { lower: true });
            meal.instructions = xss(meal.instructions);
        }
        ```
        

### 업로드된 이미지 저장 및 데이터베이스에 데이터 저장

- 데이터베이스가 아니라 파일 시스템에 이미지를 저장하도록 해보자
    - (물론 결점이 있긴 하지만) public 폴더의 images 폴더 안에 이미지를 저장해보자
- 실습
    - extension 상수 (= 확장자) 에 meal.image를 저장한 후 spilit한 뒤 pop으로 마지막 요소를 받아옴
        - SQL문에서 보면 image라는 키에 저장하고 있기 때문
    
    ```jsx
    // meals.js
    
    export function saveMeal(meal) {
        meal.slug = slugify(meal.title, { lower: true });
        meal.instructions = xss(meal.instructions);
    
        const extension = meal.image.name.split('.').pop();
    }
    ```
    
    - 템플릿 리터럴로 meal.slug의 extension 값을 fileName 상수에 저장
        - meal.slug와 extension은 따로 템플릿 리터럴을 해준다는 점에 유의!
    
    ```jsx
    // meals.js
    
    export function saveMeal(meal) {
        meal.slug = slugify(meal.title, { lower: true });
        meal.instructions = xss(meal.instructions);
    
        const extension = meal.image.name.split('.').pop();
        const fileName = `${meal.slug}.${extension}`;
    }
    ```
    
    - node.js에서 제공해주는 파일 시스템 API를 import
    
    ```jsx
    // meals.js
    
    import fs from 'node:fs';
    ```
    
    - fs의 createWriteStream 함수를 호출해 stream 상수에 저장
        - createWriteStream 함수 : 파일을 사용하고 싶은 경로 (path) 필요. stream 객체를 반환
        - 경로를 동적으로 작성 : 맨 뒤에 템플릿 리터럴로 fileName을 넣음
    
    ```jsx
    // meals.js
    
    export function saveMeal(meal) {
        meal.slug = slugify(meal.title, { lower: true });
        meal.instructions = xss(meal.instructions);
    
        const extension = meal.image.name.split('.').pop();
        const fileName = `${meal.slug}.${extension}`;
    
        const stream = fs.createWriteStream(`public/images/${fileName}`);
        stream.write();
    }
    ```
    
    - stream.write()를 호출해 파일을 사용
    
    ```jsx
    // meals.js
    
    export function saveMeal(meal) {
        meal.slug = slugify(meal.title, { lower: true });
        meal.instructions = xss(meal.instructions);
    
        const extension = meal.image.name.split('.').pop();
        const fileName = `${meal.slug}.${extension}`;
    
        const stream = fs.createWriteStream(`public/images/${fileName}`);
        stream.write();
    }
    ```
    
    - bufferedImage 상수에 meal.image.arrayBuffer()를 저장해 buffer를 만들어 저장
        - bufferedImage 상수가 stream.write() 동작에 필요
        - buffer로 반환하기 때문에 async와 await를 작성해줘야
        - write 함수에서는 일반 buffer가 필요하므로 타입을 바꿔준다
            - 첫 번째 인수: 저장할 파일
            - 두 번째 인수: 작성 후 실행될 함수 (error 인수를 받아 정상이면 null을, 아니라면 에러의 정보를 받아옴
    
    ```jsx
    // meals.js
    
    export async function saveMeal(meal) {
        meal.slug = slugify(meal.title, { lower: true });
        meal.instructions = xss(meal.instructions);
    
        const extension = meal.image.name.split('.').pop();
        const fileName = `${meal.slug}.${extension}`;
    
        const stream = fs.createWriteStream(`public/images/${fileName}`);
        const bufferedImage = await meal.image.arrayBuffer();
        stream.write(Buffer.from(bufferedImage), (error) => {
            if (error) {
                throw new Error('Saving image failed!');
            }
        });
    }
    ```
    
    - meal.image 객체에 createWriteStream에 작성했던 URL 경로로 덮어씌움 (public은 지우고)
        - public 폴더 내용은 root 단계와 동일하게 동작하기 때문
    
    ```jsx
    // meals.js
    
    export async function saveMeal(meal) {
        meal.slug = slugify(meal.title, { lower: true });
        meal.instructions = xss(meal.instructions);
    
        const extension = meal.image.name.split('.').pop();
        const fileName = `${meal.slug}.${extension}`;
    
        const stream = fs.createWriteStream(`public/images/${fileName}`);
        const bufferedImage = await meal.image.arrayBuffer();
        stream.write(Buffer.from(bufferedImage), (error) => {
            if (error) {
                throw new Error('Saving image failed!');
            }
        });
    
        meal.image = `/images/${fileName}`;
    }
    ```
    
    - 데이터베이스에 저장
        - db 객체를 활용
        - prepare 함수로 또 다른 문(statement)를 준비 (템플릿 리터럴로 작성)
            - VALUES 안에 템플릿 리터럴로 직접 지정하는 건 SQL 인젝션 문제에 취약해서 비추
            - 차라리 VALUES에 물음표들을 표기할 수 있음
            - 혹은 better-sqlite가 제공하는 initdb.js에서의 문법 활용
                - initdb.js에 있는 initData 함수의 어노테이션들 잘라내서 meals.js로 가져오기
                - 필드명을 이용해 특정 필드 연결 가능
                - 단, INSERT INTO meals 옆 괄호 안 순서와 VALUES 옆 괄호 안 순서가 동일해야!
                    - 같은 필드에 알맞은 데이터가 자동 추출됨
                - db.prepare 끝에 .run(meal)을 추가 작성
                    - meal 객체를 인수로 넘겨 run 함수 실행
            
            ```jsx
            // meals.js
            
                db.prepare(
                    `
                INSERT INTO meals (title, summary, instructions, creator, creator_email, image, slug) VALUES (
                    @title,
                    @summary,
                    @instructions,
                    @creator,
                    @creator_email,
                    @image,
                    @slug
                    )`
                ).run(meal);
            
            ```
            
        - lib 폴더의 actions.js에서 saveMeal 함수를 호출
            - 이 함수는 Promise를 반환하므로 async, await를 사용함
    - 실행
        - 화면에는 아무 변화가 없는 것처럼 보임
        - public 폴더 안의 images 폴더에 방금 제출한 이미지가 생겨있는 것을 볼 수 있음
            
            ![캡처 0429](https://github.com/serethia/serethia/assets/137035446/d4a10378-7383-42e3-9fe0-38052db2f722)
            
        - 음식 목록 페이지로 돌아가 보면 방금 제출한 이미지가 정상적으로 추가되어 있음
    - 개선
        - 제출 후 리다이렉트되게 하자
        - lib 폴더의 actions.js에서 redirect를 import하고 shareMeal 맨 마지막에 redirect 함수를 호출
        
        ```jsx
        // actions.js
        
        import { redirect } from 'next/dist/server/api-utils';
        
        export async function shareMeal(formData) {
        
        // 중략
        
            redirect('/meals'); 
        }
        ```
        
    - 실행 2
        - 로딩이 걸리기는 하지만 제출 후 자동으로 음식 목록 페이지로 이동함
        - 제출한 이미지도 정상적으로 추가되어 있음

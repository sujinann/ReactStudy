## 모듈 소개

- 리액트는 프레임워크이고, 넥스트는 이런 리액트 위에 빌드되는 프레임워크임
- 프론트엔드는 여전히 리액트로 제어할 수 있으며 백엔드도 섞여 들어간다는 점이 특징임
- 넥스트가 무엇인지, 왜 사용하는지
- 넥스트에서는 라우팅이나 페이지, 컴포넌트, 백 통신 등을 어떻게 다루는지
- 일반적인 리액트와 클라이언트 사이드 리액트가 어떤 차이를 가지는지 등등 학습

<br/>

## 파일 기반 라우팅과 리액트 서버 컴포넌트의 이해

- .next 파일, app 파일
    - app 파일 안에 layout.js , page.js 존재. 예약어임.
- 서버 컴포넌트는 리액트 만으로는 만들기 힘들지만 넥스트의 도움으로 리액트와 동일한 컴포넌트 형태로 지원
    - 유저 콘솔에서 보이지 않고 서버 로그에 보여짐

<br/>

## 파일 시스템을 통한 추가 경로 추가

넥스트의 파일 기반 라우팅

- 리액트처럼 경로를 설정하지 않고 파일로 추가함
- app 안에 about 폴더를 만들고 그 안에 page.js 생성
    - app 바로 안에 있는 page.js 는 주소의 메인 페이지 ~.com
    - ~.com/about 처럼 경로로 폴더 만들기

<br/>

## 페이지 간 이동(옳고 그른 해결책)

- 리액트는 Client Side의 Single Page Application SPA 를 활용함
- 넥스트는 둘 다 사용 가능해서 일반적으로 페이지에 접속하기 이전에는 완성된 페이지를 Server Side에서 받아오고, 페이지에 한번 접속한 뒤부터는 Client Side에서 출력한다는 장점이 있음
- 리액트와 같이 a href 도 사용 가능하지만 이것을 사용하면 SPA에서 벗어나게 되고 위의 넥스트 장점이 사라짐
    - 따라서 아래와 같이 Link 사용

```jsx
import Link from 'next/link';

export default function Home() {
  return (
    <main>
      <img src="/logo.png" alt="A server surrounded by magic sparkles." />
      <h1>Welcome to this NextJS Course!</h1>
      <p>🔥 Let&apos;s get started! 🔥</p>
      <p><Link href="/about">About Us</Link></p>
    </main>
  );
}
```

<br/>

## 페이지 및 레이아웃 작업하기

- 최소 하나의 root layout.js 파일이 필요함!!
    - app 파일 바로 밑의 layout.js
    - 여기에 레이아웃에 해당하는 페이지 전체에 적용할 global.css 를 import 하여 스타일을 전역적으로 사용할 수 있게 함
- metadata 도 예약어임
    - 레이아웃에 속한 요소를 넣어서 여러 페이지를 보여주며 공통적으로 사용할 수 있음

```jsx
import './globals.css'

export const metadata = {
  title: 'NextJS Course App',
  description: 'Your first NextJS app!',
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

<br/>

## 보호된 파일명, 커스텀 컴포넌트 및 NEXT.JS 프로젝트 정리 방법

- app 폴더 바로 안의 icon.png는 자동으로 favicon으로 이용됨 예약어
- 컴포넌트는 리액트 때와 같이 자유롭게 아무데나 만들어도 됨
    - 헷갈릴 수도 있으니 app 안의 폴더는 라우팅 방식에만 이용하고 컴포넌트 폴더는 app 밖에 생성하는 것을 강의자는 추천하나 어디에 만들어도 상관없고 선호의 차이임
    - 컴포넌트 명은 위의 icon이나 page 등 특별한 예약어 명이 아니기 때문에 기본적으로 무시되고 있음
    - 이를 이용하기 위해 import 해서 사용함. 리액트와 동일.
- 컴포넌트 폴더 안에 만들어도 되는데 이러면 컴포넌트 폴더 경로가 추가된다고 생각 할 수도 있지만 폴더 안에 page.js 가 없으므로 작동하지 않음
    - 이게 Next App Router의 작동 원리임

```jsx
// @를 이용하여 루트 프로젝트 폴더에 접근 가능
// jsconfig 파일에서 설정함
// next.js에서 이용가능한 유용한 기능
import Header from '@/components/header';
```

<br/>

## 보호된 파일명

배우신 것과 같이 NextJS에는 일부 보호된 파일명이 있습니다.

중요: 이 파일명들은 `app/`폴더(부 폴더 포함) 내부에서 생성될 때만 보호됩니다. `app/`폴더 외부에서 생성될 경우 이 파일명들을 특별한 방식으로 처리하지 않습니다.

다음 목록은 NextJS에서 보호된 파일명이며 이 섹션에서 중요한 파일명을 배울 것입니다:

- `page.js` => 신규 페이지 생성 (예: `app/about/page.js`은 `<your-domain>/about page`을 생성)
- `layout.js` => 형제 및 중첩 페이지를 감싸는 신규 레이아웃 생성
- `not-found.js` => ‘Not Found’ 오류에 대한 폴백 페이지(형제 또는 중첩 페이지 또는 레이아웃에서 전달된)
- `error.js` => 기타 오류에 대한 폴백 페이지(형제 또는 중첩 페이지 또는 레이아웃에서 전달된)
- `loading.js` => 형제 또는 중첩 페이지(또는 레이아웃)가 데이터를 가져오는 동안 표시되는 폴백 페이지
- `route.js` => API 경로 생성(즉, JSX 코드가 아닌 데이터를 반환하는 페이지, 예: JSON 형식)

공식 문서에 지원되는 모든 파일 이름과 자세한 설명이 포함된 목록을 찾을 수 있습니다: https://nextjs.org/docs/app/api-reference/file-conventions

<br/>

## 동적 경로 환경설정 및 경로 매개 변수 사용 방법

![next-dynamic-route.PNG](https://prod-files-secure.s3.us-west-2.amazonaws.com/ca254afe-fecd-4396-92df-1ab2cb1215ad/b2f35cac-1b43-42d4-acf2-dfbd35f97d63/next-dynamic-route.png)

<br/>

## 레이아웃 개념 다시 보기

- 라우팅 구조 설정할 때 경로 폴더 안에 page.js 생성하듯 레이아웃도 layout.js 파일을 폴더 안에 생성하여 중첩 레이아웃을 커스텀 할 수 있음

```jsx
import MainHeader from '@/components/main-header';
import './globals.css';

export const metadata = {
  title: 'NextLevel Food',
  description: 'Delicious meals, shared by a food-loving community.',
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <div className="header-background">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1440 320">
            <defs>
              <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="0%">
                <stop
                  offset="0%"
                  style={{ stopColor: '#59453c', stopOpacity: '1' }}
                />
                <stop
                  offset="100%"
                  style={{ stopColor: '#8f3a09', stopOpacity: '1' }}
                />
              </linearGradient>
            </defs>
            <path
              fill="url(#gradient)"
              d="M0,256L48,240C96,224,192,192,288,181.3C384,171,480,181,576,186.7C672,192,768,192,864,181.3C960,171,1056,149,1152,133.3C1248,117,1344,107,1392,101.3L1440,96L1440,0L1392,0C1344,0,1248,0,1152,0C1056,0,960,0,864,0C768,0,672,0,576,0C480,0,384,0,288,0C192,0,96,0,48,0L0,0Z"
            ></path>
          </svg>
        </div>

        <MainHeader />
        {children}
      </body>
    </html>
  );
}
```

# 레이아웃에 커스텀 컴포넌트 추가 ~ 클라이언트 컴포넌트의 효율적 사용

### 레이아웃에 커스텀 컴포넌트 추가

- 모든 파일에서 공유할 main-header.js를 작성해보자
- 실습
    - app 폴더 외부에 components 폴더 생성 및 main-header.js 추가 작성
        
        ![캡처1](https://github.com/serethia/serethia/assets/137035446/5e315918-31b0-4b64-8e2f-35695146736b)
        
    - 클릭 시 메인 페이지로 돌아가는 로고 코드를 추가 작성
        - Next 프로젝트에서는 이미지의 src를 작성할 때 src 속성에 접근해야 함 (.src)
    
    ```jsx
    // main-header.js
    
    import Link from 'next/link';
    import logoImg from '@/assets/logo.png';
    
    export default function MainHeader() {
        return (
            <header>
                <Link href="/">
                    <img src={logoImg.src} alt="A plate with food on it" />
                    NextLevel Food
                </Link>
            </header>
        );
    }
    
    ```
    
    - nav 추가 작성
        - 커뮤 멤버들이 공유한 식사를 검색하기 위한 meals 링크
        - 커뮤니티로 이동하기 위한 community 링크
    
    ```jsx
    // main-header.js
    
    import Link from 'next/link';
    import logoImg from '@/assets/logo.png';
    
    export default function MainHeader() {
        return (
            <header>
                <Link href="/">
                    <img src={logoImg.src} alt="A plate with food on it" />
                    NextLevel Food
                </Link>
    
                <nav>
                    <ul>
                        <li>
                            <Link href="/meals">Browse Meals</Link>
                        </li>
                        <li>
                            <Link href="/community">Foodies Community</Link>
                        </li>
                    </ul>
                </nav>
            </header>
        );
    }
    
    ```
    
    - MainHeader 컴포넌트를 사용할 곳에 작성해 호출
    
    ```jsx
    // layout.js
    
    import MainHeader from '@/components/main-header';
    // 중략
    export default function RootLayout({ children }) {
        return (
            <html lang="en">
                <body>
    // 중략
                    <MainHeader />
                    {children}
                </body>
            </html>
        );
    }
    ```
    

### NextJS 디자인: 옵션 및 CSS 모듈 사용

- 헤더를 스타일링해보자
- 실습
    - 두 가지 방법
    1. globals.css 파일에 모든 스타일링 코드를 작성
    2. 테일윈드 활용 (특히 Next.js와 함께 시작하는 테일윈드 docs 참고)
    - 첫 번째 방법 + Next.js가 지원하는 CSS 모듈로 진행!
        - css 파일에 특정 이름 (.module.css) 을 지정한 후 특정 컴포넌트로 스코핑
    - main-header.module.css 파일을 main-header.js 옆에 추가
        - classes 객체명으로 import해 사용
        
        ```jsx
        // main-header.js
        
        import classes from './main-header.module.css';
        ```
        
        - 스타일을 적용할 컴포넌트의 className에 이 객체를 동적으로 불러옴 (단, 클래스에 접근해야 함)
        
        ```jsx
        // main-header.js
        
                <header className={classes.header}>
                    <Link className={classes.logo} href="/">
        // 중략
                    </Link>
        
                    <nav className={classes.nav}>
        // 중략
                    </nav>
                </header>
        ```
        

### NextJS 이미지 컴포넌트를 통한 이미지 최적화

- 기본 (정규) img 요소보다 Next.js에 내장된 더 좋은 요소(Image 컴포넌트)로 이미지를 출력해보자 ([참고](https://nextjs.org/docs/app/api-reference/components/image))
- 실습
    - img를 Image로 변경하고, src 속성값인 logoImg.src에서 .src를 제거
    
    ```jsx
    // main-header.js
    
    import Image from 'next/image';
    // 중략
                    <Image src={logoImg} alt="A plate with food on it" />
    ```
    
    - 개발자 도구를 보면 작성하지 않은 여러 속성들이 자동으로 추가되어 있음
        
        ```jsx
        <img alt="A plate with food on it" loading="lazy" width="1024" height="1024" decoding="async" data-nimg="1" style="color:transparent" srcset="/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flogo.5daadb4d.png&amp;w=1080&amp;q=75 1x, /_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flogo.5daadb4d.png&amp;w=2048&amp;q=75 2x" src="/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flogo.5daadb4d.png&amp;w=2048&amp;q=75">
        ```
        
        - loading=”lazy”
            - 눈에 실제로 보일 때만 이미지를 로딩(지연로딩)시킴
        - srcset=”/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flogo.5daadb4d.png&amp;w=1080&amp;q=75 1x, /_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flogo.5daadb4d.png&amp;w=2048&amp;q=75 2x”
            - 뷰포트와 웹사이트를 방문하는 기기에 따라 조정된 크기의 이미지가 로딩되도록 보장함
            - 사용되는 브라우저에 가장 알맞은 파일 포맷으로 자동 서브함
        - 이미지가 항상 페이지와 함께 보이는 것을 막을 때는 priority 속성(우선적으로 로딩) 추가 필요
            - 콘솔에 노란색 경고가 뜨지 않게 됨
            - 필요하지 않은 컨텐츠 변경 방지 가능
            - 깜빡임 방지 가능
        
        ```jsx
        // main-header.js
        
                        <Image src={logoImg} alt="A plate with food on it" priority />
        ```
        

### 기타 커스텀 컴포넌트 사용

- header background를 외부 컴포넌트로 아웃소싱해보자
- 실습
    - components 폴더에 main-header-background.js 파일을 추가
    - layout.js의 div 태그 코드를 잘라내 main-header-background.js에서 반환
    
    ```jsx
    // main-header-background.js
    
    export default function MainHeaderBackground() {
        return (
            <div className="header-background">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1440 320">
                    <defs>
                        <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="0%">
                            <stop offset="0%" style={{ stopColor: '#59453c', stopOpacity: '1' }} />
                            <stop offset="100%" style={{ stopColor: '#8f3a09', stopOpacity: '1' }} />
                        </linearGradient>
                    </defs>
                    <path
                        fill="url(#gradient)"
                        d="M0,256L48,240C96,224,192,192,288,181.3C384,171,480,181,576,186.7C672,192,768,192,864,181.3C960,171,1056,149,1152,133.3C1248,117,1344,107,1392,101.3L1440,96L1440,0L1392,0C1344,0,1248,0,1152,0C1056,0,960,0,864,0C768,0,672,0,576,0C480,0,384,0,288,0C192,0,96,0,48,0L0,0Z"
                    ></path>
                </svg>
            </div>
        );
    }
    
    ```
    
    - components 폴더에 main-header-background.module.css 파일을 추가
    - globals.css 파일의 헤더 관련 스타일 코드를 main-header-background.module.css 파일로 옮기기
    
    ```jsx
    // main-header-background.module.css
    
    .header-background {
      position: absolute;
      width: 100%;
      height: 320px;
      top: 0;
      left: 0;
      z-index: -1;
    }
    
    svg {
      display: block;
      width: 100%;
      height: auto;
    }
    ```
    
    - svg 선택자를 header-background 클래스를 가진 요소의 선택자로 수정
    
    ```jsx
    // main-header-background.module.css
    
    .header-background svg {
        display: block;
        width: 100%;
        height: auto;
    }
    ```
    
    - main-header-background.js에서 classes로 import한 후 className 값을 동적인 classes로 수정
    
    ```jsx
    // main-header-background.js
    
    import classes from './main-header-background.module.css';
    // 중략
            <div className={classes}>
    ```
    
    - .으로 접근(classes.header-background)하지 말고 대괄호 안에 속성값을 작성해 접근
    
    ```jsx
    // main-header-background.js
    
            <div className={classes['header-background']}>
    ```
    
    - layout.js에서 MainHeaderBackground 컴포넌트 호출
        - 이 방법 말고 main-header.js에서 호출할 수도 있음 (이 방법으로 진행!)
    
    ```jsx
    // layout.js
    
    import MainHeaderBackground from '@/components/main-header-background';
    // 중략
        return (
            <>
                <MainHeaderBackground />
                <html lang="en">
                    <body>
                        <MainHeader />
                        {children}
                    </body>
                </html>
            </>
        );
    ```
    
    ```jsx
    // main-header.js
    
    import Link from 'next/link';
    import logoImg from '@/assets/logo.png';
    import classes from './main-header.module.css';
    import Image from 'next/image';
    import MainHeaderBackground from './main-header-background';
    
    export default function MainHeader() {
        return (
            <>
                <MainHeaderBackground />
                <header className={classes.header}>
                    <Link className={classes.logo} href="/">
                        <Image src={logoImg} alt="A plate with food on it" priority />
                        NextLevel Food
                    </Link>
    
                    <nav className={classes.nav}>
                        <ul>
                            <li>
                                <Link href="/meals">Browse Meals</Link>
                            </li>
                            <li>
                                <Link href="/community">Foodies Community</Link>
                            </li>
                        </ul>
                    </nav>
                </header>
            </>
        );
    }
    
    ```
    
    - components 폴더 안에 main-header 폴더를 생성해 관련 파일을 모두 집어넣음
        
        ![캡처2](https://github.com/serethia/serethia/assets/137035446/0865eef2-c495-447d-9cc0-f91b5a385671)
        

### 시작 페이지 내용 채우기

- 메인 페이지 컨텐츠를 app 폴더 안의 page.js에서 작업해보자
- 실습
    - page.js의 기존 반환 코드를 지우고 아래와 같이 수정해두자
    
    ```jsx
    // page.js
    
    export default function Home() {
        return (
            <>
                <header></header>
                <main></main>
            </>
        );
    }
    ```
    
    - app 폴더에 page.module.css 파일을 추가
    - page.js에서 classes로 import한 후 className 값에 알맞게 작성 (+ 기타 코드도 추가 작성)
    
    ```jsx
    // page.js
    
    import Link from 'next/link';
    import classes from './page.module.css';
    
    export default function Home() {
        return (
            <>
                <header className={classes.header}>
                    <div className={classes.slideshow}></div>
                    <div>
                        <div className={classes.hero}>
                            <h1>NextLevel Food for NextLevel Foodies</h1>
                            <p>Taste & share food from all over the world.</p>
                        </div>
                        <div className={classes.cta}>
                            <Link href="/community">Join the Community</Link>
                            <Link href="/meals">Explore Meals</Link>
                        </div>
                    </div>
                </header>
                <main>
                    
                </main>
            </>
        );
    }
    ```
    
    - main 태그 사이에 필요한 코드를 추가 작성
    
    ```jsx
    // page.js
    
    import Link from 'next/link';
    import classes from './page.module.css';
    
    export default function Home() {
      return (
        <>
    // 중략
          <main>
            <section className={classes.section}>
              <h2>How it works</h2>
              <p>
                NextLevel Food is a platform for foodies to share their favorite
                recipes with the world. It&apos;s a place to discover new dishes, and to
                connect with other food lovers.
              </p>
              <p>
                NextLevel Food is a place to discover new dishes, and to connect
                with other food lovers.
              </p>
            </section>
    
            <section className={classes.section}>
              <h2>Why NextLevel Food?</h2>
              <p>
                NextLevel Food is a platform for foodies to share their favorite
                recipes with the world. It&apos;s a place to discover new dishes, and to
                connect with other food lovers.
              </p>
              <p>
                NextLevel Food is a place to discover new dishes, and to connect
                with other food lovers.
              </p>
            </section>
          </main>
        </>
      );
    }
    ```
    

### 이미지 슬라이드쇼 준비

- 아직 비어있는 슬라이드 쇼 코드를 작성해보자
- 실습
    - components 폴더에 images 폴더를 추가하고 image-slideshow.js 파일과 image-slideshow.module.css 파일 추가
        - 매 5초마다 인덱스 번호 state를 변경해 슬라이드 쇼를 구현함
        - unmount 시 cleanup 함수에 의해 초기화
    
    ```jsx
    // image-slideshow.js
    
    import { useEffect, useState } from 'react';
    import Image from 'next/image';
    
    import burgerImg from '@/assets/burger.jpg';
    import curryImg from '@/assets/curry.jpg';
    import dumplingsImg from '@/assets/dumplings.jpg';
    import macncheeseImg from '@/assets/macncheese.jpg';
    import pizzaImg from '@/assets/pizza.jpg';
    import schnitzelImg from '@/assets/schnitzel.jpg';
    import tomatoSaladImg from '@/assets/tomato-salad.jpg';
    import classes from './image-slideshow.module.css';
    
    const images = [
      { image: burgerImg, alt: 'A delicious, juicy burger' },
      { image: curryImg, alt: 'A delicious, spicy curry' },
      { image: dumplingsImg, alt: 'Steamed dumplings' },
      { image: macncheeseImg, alt: 'Mac and cheese' },
      { image: pizzaImg, alt: 'A delicious pizza' },
      { image: schnitzelImg, alt: 'A delicious schnitzel' },
      { image: tomatoSaladImg, alt: 'A delicious tomato salad' },
    ];
    
    export default function ImageSlideshow() {
      const [currentImageIndex, setCurrentImageIndex] = useState(0);
    
      useEffect(() => {
        const interval = setInterval(() => {
          setCurrentImageIndex((prevIndex) =>
            prevIndex < images.length - 1 ? prevIndex + 1 : 0
          );
        }, 5000);
    
        return () => clearInterval(interval);
      }, []);
    
      return (
        <div className={classes.slideshow}>
          {images.map((image, index) => (
            <Image
              key={index}
              src={image.image}
              className={index === currentImageIndex ? classes.active : ''}
              alt={image.alt}
            />
          ))}
        </div>
      );
    }
    
    ```
    
    - page.js에서 image-slideshow.js를 호출
    
    ```jsx
    // page.js
    
    import ImageSlideshow from '@/components/images/image-slideshow';
    // 중략
                    <div className={classes.slideshow}>
                        <ImageSlideshow />
                    </div>
    ```
    
    - BUT 실행 시 오류 → 클라이언트 컴포넌트에서만 useState가 작동하기 때문
    
    ```jsx
    You're importing a component that needs useState. It only works in a Client Component but none of its parents are marked with "use client", 
    so they're Server Components by default.
    Learn more: https://nextjs.org/docs/getting-started/react-essentials  
    
       ╭─[C:\Users\SSAFY\Downloads\foodies-starting-project\05-onwards-foodies-starting-project\components\images\image-slideshow.js:1:1]
     1 │ import { useEffect, useState } from 'react';
       ·                     ────────
     2 │ import Image from 'next/image';
     3 │
     4 │ import burgerImg from '@/assets/burger.jpg';
       ╰────
    
    Maybe one of these should be marked as a client entry with "use client":
      ./components\images\image-slideshow.js
      ./app\page.js
    ```
    

### 리액트 서버 컴포넌트 vs 클라이언트 컴포넌트 - 적절한 사용 선택 방법

- Next.js는 풀스택 프레임워크
    - 프론트엔드 코드와 백엔드 코드가 함께 실행됨 → HTML 코드를 재실행
    - RSC (React Server Components) : 서버에서만 렌더링됨 → 그래서 ‘리액트 서버 컴포넌트’로 불림
- 리액트 서버 컴포넌트 vs 클라이언트 컴포넌트
    - 리액트 서버 컴포넌트 (RSC)
        - 서버에서만 렌더링
        - 모든 리액트 컴포넌트는 Next.js 앱에서 RSC에 해당함
        - 클라리언트 측에서 다운로드하는 JavaScript 코드가 줄어듦
            - 성능 향상 및 검색엔진 최적화 (SEO) 에도 좋음
    - 클라이언트 컴포넌트
        - 서버에서 사전 렌더링되고 클라이언트에서도 렌더링
        - use로 시작하는 훅들이나 onClick 같은 이벤트 핸들러들을 실행할 수 있음 → 클라이언트의 실행이 기준이 되어야 하기 때문
- 만약 다른 코드에서 콘솔에 메세지를 출력하고 page.js에서 일부러 주석 처리해 오류를 내보면 콘솔에 아무런 메세지가 나오지 않게 됨
    - 브라우저에서 실행되지 않았기 때문
    - 서버에서는 실행됨 → 터미널에는 출력됨
- 만약 meals 폴더 안의 page.js에 콘솔 메세지를 출력하도록 작성한 후 실행해봐도 콘솔에 아무것도 뜨지 않음 (물론 터미널에는 출력됨)
- Next.js에서 클라이언트 컴포넌트를 사용하고 싶다면?
    - use client 라는 지시어를 사용
    - 클라이언트와의 상호작용이 가능해짐
- image-slideshow.js 맨 위에 ‘use client’를 추가 작성
    - 실행해보면 정상적으로 5초마다 슬라이드 쇼가 바뀌며 사이트도 오류 없이 작동함

```jsx
// image-slideshow.js

'use client';
```

### 클라이언트 컴포넌트의 효율적 사용

- 커뮤니티 페이지를 작업해보자
- 실습
    - community 폴더 안의 page.js를 수정하고 page.module.css 파일을 추가
    
    ```jsx
    // page.js
    
    import Image from 'next/image';
    
    import mealIcon from '@/assets/icons/meal.png';
    import communityIcon from '@/assets/icons/community.png';
    import eventsIcon from '@/assets/icons/events.png';
    import classes from './page.module.css';
    
    export default function CommunityPage() {
        return (
            <>
                <header className={classes.header}>
                    <h1>
                        One shared passion: <span className={classes.highlight}>Food</span>
                    </h1>
                    <p>Join our community and share your favorite recipes!</p>
                </header>
                <main className={classes.main}>
                    <h2>Community Perks</h2>
    
                    <ul className={classes.perks}>
                        <li>
                            <Image src={mealIcon} alt="A delicious meal" />
                            <p>Share & discover recipes</p>
                        </li>
                        <li>
                            <Image src={communityIcon} alt="A crowd of people, cooking" />
                            <p>Find new friends & like-minded people</p>
                        </li>
                        <li>
                            <Image src={eventsIcon} alt="A crowd of people at a cooking event" />
                            <p>Participate in exclusive events</p>
                        </li>
                    </ul>
                </main>
            </>
        );
    }
    
    ```
    
    ```jsx
    // page.module.css
    
    .header {
        gap: 3rem;
        margin: 3rem auto 5rem auto;
        width: 90%;
        max-width: 75rem;
        color: #ddd6cb;
        font-size: 1.5rem;
        text-align: center;
    }
    
    .header h1 {
        font-family: 'Montserrat', sans-serif;
    }
    
    .highlight {
        background: linear-gradient(90deg, #f9572a, #ff8a05);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }
    
    .main {
        width: 90%;
        max-width: 40rem;
        margin: 0 auto;
        text-align: center;
    }
    
    .main h2 {
        font-family: 'Montserrat', sans-serif;
        font-size: 2rem;
        margin-bottom: 3rem;
        color: #ddd6cb;
    }
    
    .perks {
        list-style: none;
        margin: 3rem 0;
        padding: 0;
    }
    
    .perks li {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 2rem;
    }
    
    .perks img {
        width: 8rem;
        height: 8rem;
        object-fit: contain;
    }
    
    .perks p {
        font-family: 'Montserrat', sans-serif;
        font-size: 1.5rem;
        font-weight: bold;
        margin: 0;
        color: #ddd6cb;
    }
    
    ```
    
    - 선택한 메뉴에 하이라이트 효과를 추가하기 위해 스타일을 적용해보자
        - main-header.js에서 usePathname 훅을 호출해 path 상수에 저장
            - usePathname : 현재 활동 경로를 도메인 다음 부분에 줌
        - nav의 Link에 className을 동적으로 path.startsWith(’/어쩌고’)로 작성
            - 중첩된 링크도 포함되기 때문
        - 삼항연산자로 true일 경우 active 스타일을 적용
            - CSS 모듈을 사용하므로 ‘active’로 적으면 안됨 → classes.active로 작성해야
        - false일 경우 undefined를 작성해 아무 클래스도 적용되지 않게 함
    
    ```jsx
    // main-header.js
    
    import { usePathname } from 'next/navigation';
    
    export default function MainHeader() {
        const path = usePathname();
    // 중략
                            <li>
                                <Link href="/meals" className={path.startsWith('/meals') ? classes.active : undefined}>
                                    Browse Meals
                                </Link>
                            </li>
                            <li>
                                <Link href="/community" className={path === '/community' ? classes.active : undefined}>
                                    Foodies Community
                                </Link>
                            </li>
    ```
    
    - 실행 시 use로 시작하는 훅이어서 오류가 뜸 → 클라이언트 컴포넌트에서만 작동
    - use client 추가 작성
        - 최대한 트리 아래쪽으로 내려가 use client를 사용해야
        - 필요한 컴포넌트만 클라이언트 컴포넌트로 만들고 나머지는 서버 컴포넌트로 유지하기 위함
    
    ```jsx
    // main-header.js
    
    'use client';
    ```
    
    - main-header 폴더에 nav-link.js 파일과 nav-link.module.css 추가
    - nav-link.js에 Link 코드들을 공유할 수 있도록 작성
        - use client를 써서 클라이언트 컴포넌트로 만듦
        - nav-link.module.css 파일에서 classes라는 이름으로 import
    
    ```jsx
    // nav-link.js
    
    'use client';
    
    import Link from 'next/link';
    import { usePathname } from 'next/navigation';
    import classes from './main-header.module.css';
    
    export default function NavLink({ href, children }) {
        const path = usePathname();
    
        return (
            <Link href={href} className={path.startsWith(href) ? classes.active : undefined}>
                {children}
            </Link>
        );
    }
    
    ```
    
    - main-header.js에서 두 nav Link들을 삭제하고 NavLink 호출
    
    ```jsx
    // main-header.js
    
    import NavLink from './nav-link';
    // 중략
                            <li>
                                <NavLink href="/meals">Browse Meals</NavLink>
                            </li>
                            <li>
                                <NavLink href="/community">Foodies Community</NavLink>
                            </li>
    ```
    
    - main-header.module.css에서 active 관련 및 nav a 스타일 코드만 nav-link.module.css로 옮겨옴
    
    ```jsx
    // nav-link.module.css
    
    .nav a {
        text-decoration: none;
        color: #ddd6cb;
        font-weight: bold;
        padding: 0.5rem 1rem;
        border-radius: 0.5rem;
    }
    
    .nav a:hover,
    .nav a:active {
        background: linear-gradient(90deg, #ff8a05, #f9b331);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        text-shadow: 0 0 18px rgba(248, 190, 42, 0.8);
    }
    
    .active {
        background: linear-gradient(90deg, #ff8a05, #f9b331);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }
    
    ```
    
    - nav a를 전부 link로 수정
    
    ```jsx
    // nav-link.module.css
    
    .link {
        text-decoration: none;
        color: #ddd6cb;
        font-weight: bold;
        padding: 0.5rem 1rem;
        border-radius: 0.5rem;
    }
    
    .link:hover,
    .link:active {
        background: linear-gradient(90deg, #ff8a05, #f9b331);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        text-shadow: 0 0 18px rgba(248, 190, 42, 0.8);
    }
    
    .active {
        background: linear-gradient(90deg, #ff8a05, #f9b331);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }
    
    ```
    
    - nav-link.js에서 classes.active를 템플릿 리터럴로 classes.link와 classes.active로 수정하고, undefined를 classes.link로 수정
    
    ```jsx
    // nav-link.js
    
            <Link href={href} className={path.startsWith(href) ? `${classes.link} ${classes.active}` : classes.link}>
    ```

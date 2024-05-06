## 정적 메타데이터 추가

페이지 안에 메타데이터로 넣으면 됨

- 기본적으로 루트 메타데이터 적용
- 세부 페이지 안에 메타데이터가 존재하면 덮어씌움

```jsx
import { Suspense } from 'react';
import Link from 'next/link';

import classes from './page.module.css';
import MealsGrid from '@/components/meals/meals-grid';
import { getMeals } from '@/lib/meals';

// 예시
export const metadata = {
  title: 'All Meals',
  description: 'Browse the delicious meals shared by our vibrant community.',
};

async function Meals() {
  console.log('Fetching meals');
  const meals = await getMeals();

  return <MealsGrid meals={meals} />;
}
... 생략
```

</ br>

## 동적 메타데이터 추가

- 정적 메타데이터 형식과 다른 방법으로 추가

```jsx
export async function generateMetadata({ params }) {
  const meal = getMeal(params.mealSlug);

  if (!meal) {
    notFound();
  }

  return {
    title: meal.title,
    description: meal.summary,
  };
}

export default function MealDetailsPage({ params }) {
  const meal = getMeal(params.mealSlug);
... 생략
```

</ br>

## NEXT.JS 페이지 라우터에 대하여

- NEXT.JS 에는 크게 두 가지 라우터 방식이 있음
    - App Router & Pages Router
    - 현재까지 강의에서 다뤄온 것은 App Router 방식
    - App Router 방식이 더 현대적이고 추천하는 기술
    - 그러나 Pages Router가 이전부터 사용해오던 방식이기에 더 안정적인 면이 있음

</ br>

## NEXT.JS 프로젝트 생성하기

- npx create-next-app@latest 명령어로 넥스트 프로젝트 생성
- 생성시 주의할 점은 옵션 선택 부분에서 App Router를 추천하며 이것을 사용할 지 묻는데 여기서 No 를 골라야 Pages Router 방식을 사용할 수 있음

</ br>

## 첫 페이지 추가하기

처음 넥스트 프로젝트를 생성하면 pages 폴더가 존재함

- 그 안에 페이지들을 추가
- index.js index는 특수해서 [our-domain.com/](http://our-domain.com/) 로 연결
- news.js 나머지는 [our-domain.com/news](http://our-domain.com/news) 파일 이름에 따른 경로로 연결
- 서버 사이드 사전 렌더링이 기본적으로 동작

</ br>

## 중첩 경로 및 페이지 추가하기

Pages Router 방식에서 index 는 항상 특별한 기능이 지정되어 있음

index.js 파일이 속한 경로의 페이지를 나타냄

- [our-domain.com/news](http://our-domain.com/news) 경로의 페이지를 만들고 싶은 경우
    - pages 폴더의 news.js 파일 혹은 pages/news 폴더의 index.js 파일로 나타낼 수 있음
- [our-domain.com/mews/detail](http://our-domain.com/mews/detail) 경로의 페이지를 만들고 싶은 경우에는
    - pages/news 폴더의 detail.js 혹은 pages/news/detail 폴더의 index.js 이런 식

</ br>

## 동적 페이지 값 만들기

- 페이지가 많을 경우 하나하나 하드코딩으로 만들 수 없음
- [newsId].js 와 같은 [식별자].확장자 형태로 대괄호를 이용하여 동적 페이지를 생성

</ br>

## 동적 매개변수 값 추출하기

- useRouter 훅을 이용함
- 동적 매개변수 값을 이용하여 효율적으로 api요청 혹은 페이지 렌더링이 가능

```jsx
import { useRouter } from 'next/router';

// our-domain.com/news/something-important

function DetailPage() {
  const router = useRouter();

  const newsId = router.query.newsId;

  // send a request to the backend API
  // to fetch the news item with newsId

  return <h1>The Detail Page</h1>
}

export default DetailPage;
```

</ br>

## 페이지 간 연결하기

- 특수 컴포넌트 Link가 앵커 태그를 렌더링하고 이 앵커태그에 하는 클릭을 감지함
- 클릭하면 브라우저의 기본 동작으로 새 HTML 페이지를 받는 요청을 보낼 수 없게 함
- 대신에 불러올 컴포넌트를 읽어들이고 URL을 변경하여 페이지가 바뀐 것처럼 보이게 함
- 이를 통해 Single Page Application 형태를 유지

```jsx
// our-domain.com/news
import Link from 'next/link';
import { Fragment } from 'react';

function NewsPage() {
  return (
    <Fragment>
      <h1>The News Page</h1>
      <ul>
        <li>
          <Link href='/news/nextjs-is-a-great-framework'>
            NextJS Is A Great Framework
          </Link>
        </li>
        <li>Something Else</li>
      </ul>
    </Fragment>
  );
}

export default NewsPage;
```

### Meal 데이터와 규격이 불분명한 이미지 출력

- Meal 페이지에서 등록한 이미지들을 grid 형식으로 보여줄 MealGrid를 사용할 예정

```jsx
// page.js
import Link from "next/link";
import classes from "./page.module.css";
import MealsGrid from "./meals-grid";

export default function MealPage() {
	return (
		<>
			<header className={classes.header}>
				<h1>
					Delicious meals, created
					<span className={classes.highlight}> by you</span>
				</h1>
				<p>니 알아서 해 믁으라</p>
				<p className={classes.cta}>
					<Link href="/meals/share">Share Your Favorite Recipe</Link>
				</p>
			</header>
			<main className={classes.main}>
				<MealsGrid meals={[]} />
			</main>
		</>
	);
}

// meals-grid.js
import MealItem from "./meal-item";
import classes from "./meals-grid.module.css";

export default function MealsGrid({ meals }) {
	return (
		<ul className={classes.meals}>
			{meals.map((meal) => {
				<li key={meals.id}>
					<MealItem {...meal} />
				</li>;
			})}
		</ul>
	);
}

```

- DB에 업로드한 사진을 동적으로 로딩할 예정
- Next에서는 동적으로 가져온 모든 이미지의 크기 규격을 맞출 수 없음
    - Image에 fill 속성으로 width, height를 수동 지정할 필요 없이 해결

```jsx
<Image src={image} alt={title} fill />
```

### SQLite DB 설정

- 공유한 사진을 위한 DB 설치
    - SQLIte3

```jsx
npm i better-sqlite3
```

```jsx
const sql = require("better-sqlite3");
const db = sql("meals.db");

const dummyMeals = [
	{
		title: "Juicy Cheese Burger",
		slug: "juicy-cheese-burger",
		image: "/images/burger.jpg",
		instruction: "",
		creator_email: "sophiagreen@example.com",
	},
	...
];

db.prepare(
	`
   CREATE TABLE IF NOT EXISTS meals (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       slug TEXT NOT NULL UNIQUE,
       title TEXT NOT NULL,
       image TEXT NOT NULL,
       summary TEXT NOT NULL,
       instructions TEXT NOT NULL,
       creator TEXT NOT NULL,
       creator_email TEXT NOT NULL
    )
`
).run();

async function initData() {
	const stmt = db.prepare(`
      INSERT INTO meals VALUES (
         null,
         @slug,
         @title,
         @image,
         @summary,
         @instructions,
         @creator,
         @creator_email
      )
   `);

	for (const meal of dummyMeals) {
		stmt.run(meal);
	}
}

initData();

```

```jsx
node initdb.js
```

### NextJS 및 풀스택 기능을 활용한 데이터 불러오기

- 이제 mealpage에서 생성한 로컬 db의 데이터를 불러옴
- 리액트앱은 별도의 백엔드 서버가 필요하지만 Next.js는 프론트와 서버가 통합되어 있어서 그럴 필요가 없음
    - 모든 컴포넌트는 기본적으로 서버에서만 실행되는 서버 컴포넌트
    - useEffect 훅으로 데이터를 가져올 필요가 없음

```jsx
// meals.js
import sql from 'better-sqlite3';

const db = sql('meals.db');

export async function getMeals() {
  db.prepare('SELECT * FROM meals')
  .all(); // run() -> db 변경시
  // get('') -> 특정 컬럼
}
// 실제 서버와 유사하게 임의의 로딩시간 설정
export async function getMeals() {
  await new Promise((resolve) => setTimeout(resolve, 2000));
  return db.prepare('SELECT * FROM meals').all();
}
```

- 서버 컴포넌트는 async 함수로 지정 가능
    - promise method에 직접 await 실행 가능

```jsx
import { getMeals } from "@/lib/meals";

export default async function MealPage() {
  const meals = await getMeals();
  ..
```

### 로딩 페이지 추가

- 로딩 과정에서 2초 정도 딜레이가 있음
    - 처음 로딩 시에만 적용, 다른 페이지 이동 후 다시 불러오면 로딩이 없음
    - NextJS의 방문한 모든 페이지 캐싱 기능
    - 배포 빌드는 심화된 캐싱 제공
- Next에서 loading.js를 추가하면 디폴트 로딩 페이지 제공

```jsx
import classes from "./loading.module.css";

export default function MealsLoadingPage() {
	return <div className={classes.loading}>Fetching data..</div>;
}
```

### Suspense & Streamed Response를 이용한 세분화 로딩 상태 관리

- 현재는 로딩페이지가 전체를 차지
    - 이미지가 로드되는 영역에만 로딩 효과를 주고싶음
- 기본 loading.js를 사용하지 않음

```jsx
import Link from "next/link";
import classes from "./page.module.css";
import MealsGrid from "./meals-grid";
import { getMeals } from "@/lib/meals";
import { Suspense } from "react";
import MealsLoadingPage from "./loading-out";

async function Meals() {
	const meals = await getMeals();
	return <MealsGrid meals={meals} />;
}

export default function MealPage() {
	return (
		<>
			<header className={classes.header}>
				<h1>
					Delicious meals, created
					<span className={classes.highlight}> by you</span>
				</h1>
				<p>니 알아서 해 믁으라</p>
				<p className={classes.cta}>
					<Link href="/meals/share">Share Your Favorite Recipe</Link>
				</p>
			</header>
			<main className={classes.main}>
				<Suspense fallback={<p className={classes.loading}>Fetching data..</p>}>
					<Meals />
				</Suspense>
			</main>
		</>
	);
}

```

### 오류 처리 방법

- Next에서 error.js로 에러 대비 페이지 생성 가능
    - 로딩과 같이 라우팅 별로 별도 생성 가능
- 에러페이지가 없으면 콘솔 에러가 직접적으로 보여서 UX에 악영향
- 에러페이지는 기본적으로 클라이언트 컴포넌트
- Next에서 중요한 정보를 숨겨서 표시해주는 별도의 error prop을 받을 수 있음

```jsx
//error.js
"use client";

export default function Error({ error }) {
	return (
		<main className="error">
			<h1>Error!</h1>
			<p>{error}</p>
		</main>
	);
}

```

### Not Found 상태 처리 방법

- 일반 Error와 Not Found 페이지가 별도로 존재
    - 잘못된 url 입력 시 처리

```jsx
// not-found.js
export default function NotFound() {
	return (
		<main className="not-found">
			<h1>Not found</h1>
			<p>없는 페이지임</p>
		</main>
	);
}

```

### 동적 경로와 경로 매개변수를 활용한 Meals 세부내용 로딩 및 렌더링

- meal의 하위 페이지([mealSlug] 폴더 페이지)로 이동
- dangerouselySetInnerHtml Prop 사용
    - 출력되는 내용을 검증하지 않으면 XSS 공격에 노출

```jsx
// meals.js
export async function getMeal(slug) {
  return db.prepare('SELECT * FROM meals WHERE slug= ?').get(slug);
	// WHERE slug=slug 식의 직접 대입은 SQL Injection 위험
}

// page.js([mealSlug])
import classes from "./page.module.css";
import Image from "next/image";
import { getMeal } from "@/lib/meals";

export default function MealDetailPage({ params }) {
	const meal = getMeal(params.mealSlug);

	meal.instructions = meal.instructions.replace(/\n/g, "<br/>");
	return (
		<>
			<header className={classes.header}>
				<div className={classes.image}>
					<Image src={meal.image} fill alt="image" />
				</div>
				<div className={classes.headerText}>
					<h1>{meal.title}</h1>
					<p className={classes.creator}>
						by <a href={`${meal.creator_email}`}>{meal.creator}</a>
					</p>
					<p className={classes.summary}>{meal.summary}</p>
				</div>
			</header>
			<main>
				<p
					className={classes.instructions}
					dangerouslySetInnerHTML={{
						__html: meal.instructions,
					}}
				></p>
			</main>
		</>
	);
}

```

### useFormStatus를 이용한 양식 제출 관리

- 폼 제출 후 리다이렉트까지 경과시간 UX 개선
- React의 useFormStatus 훅으로 로딩 UI 구현
- pending이면 true를 반환하는 status 객체 사용 가능
- UI 변경 사항이라 클라이언트 컴포넌트로 사용해야함
    - 버튼 하나의 텍스트 변경에 전체 컴포넌트 렌더링은 불필요함

```jsx
//button
"use client";

import { useFormStatus } from "react-dom";

export default function MealsFormSubmit() {
	const { pending } = useFormStatus();

	return (
		<button disabled={pending} type="submit">
			Share Meal
		</button>
	);
}

```

### 서버 사이드 입력 유효성 확인 추가 방법

- 폼에서 받은 입력값의 유효성을 검증해야 함
    - 기본적으로 required로 필수값 처리
        - devtools에서 html을 삭제하면서 넘어갈 수 있음

```jsx
// action.js
"use server";

import { redirect } from "next/navigation";
import { saveMeal } from "./meals";

function isInvalid(text) {
	return !text || text.trim() === "";
}

export default async function shareMeal(formData) {
	const meal = {
		title: formData.get("title"),
		image: formData.get("image"),
		summary: formData.get("summary"),
		instructions: formData.get("instructions"),
		creator: formData.get("name"),
		creator_mail: formData.get("email"),
	};

	if (
		isInvalid(meal.title) ||
		isInvalid(meal.summary) ||
		isInvalid(meal.instructions) ||
		isInvalid(meal.name) ||
		isInvalid(meal.creator) ||
		!meal.image ||
		meal.image.size === 0
	) {
		throw new Error("Invalid Input");
	}

	await saveMeal(meal);
	redirect("/meals");
}

```

- 유효성 실패 시 메인 에러 페이지로 넘어감
- 하나의 값이 잘못되면 모든 입력값이 날아가는 불편함이 있다

### 서버 행동 응답 및 useFormState 작업

- 에러 말고 다른 값을 반환할 수 있음

```jsx
// action.js
...
return {
			message: "Invalid input.",
};
// page.js
export default function ShareMealPage() {
  const [state, formAction] = useFormState(shareMeal, {message: null});
	return (
	...
					<form className={classes.form} action={formAction}>

	...
					{state.message && <p>{state.message}</p> }
// action.js
export default async function shareMeal(prevState, formData) {

```

- formState의 이전 상태를 정의하는 prevState 변수가 formState의 state에 있기 때문에 함수에 변수로 추가

### NextJS 캐싱 구축 및 이해

- 개발 환경에는 문제가 없으나 배포 에러가 발생할 수 있음
- 배포 환경 확인 방법 → npm run build, npm start
    - 주소는 동일하지만 배포 환경에 맞는 최적화 환경으로 실행
- 폼에서 제출하면 리다이렉트가 매우 빠르지만 추가가 되지 않음
- 새로고침 후 meal 페이지로 가면 로딩 시간이 걸리지 않음
    - meal.js에서 임의로 만든 5초의 딜레이가 적용되지 않음
- Next의 공격적인 캐싱 때문
    - 미리 만들 수 있는 모든 페이지를 (static pages) 렌더링해서 저장
    - == 서버 컴포넌트의 모든 페이지를 미리 불러와서 저장
    - 저장된 페이지를 캐싱해서 방문자에게 저장
    
    → 업데이트된 데이터를 다시 가져오는 작업이 생략
    

### 캐시 유효성 재확인 트리거

- 새로운 음식이 등록되면 캐싱의 일부를 버리고 다시 렌더링 하도록 지시
- revalidatePath
    - 해당 경로의 페이지의 캐시 유효성 검사를 다시 하도록 지시

```jsx
// action.js
	await saveMeal(meal);
  revalidatePath('/meals');
	redirect("/meals");
```

- 해당 경로만 revalidate, 하위 경로는 해당되지 않음
- 두 번째 변수를 넣을 수 있음

```jsx
revalidatePath('/meals', page // 해당 페이지만
layout // layout을 공유하는 모든 페이지
)
```

### 로컬 FileSystem에 파일 저장 금지

- 등록한 사진이 보이지 않음
    - public 폴더의 image에 저장하고 있기 때문
- 개발 환경에서는 접근이 가능하지만 배포된 폴더(.next)에서는 public이 존재하지 않아 무시됨
    - 새로운 파일을 public에 넣을 수 없음
    - 공식 문서에서 S3와 같은 외부 서버를 사용하도록 권장됨

### 업로드한 이미지를 클라우드에 업로드(AWS S3)

1. 이미지를 업로드할 AWS S3 버킷을 생성
2. public 폴더에 포함된 이미지들의 src를 해당 s3 버킷의 주소로 재할당

```jsx
src={`https://qqq.s3.amawonaws.com./${image}`}
```

→ Next의 Image src는 외부 경로를 받지 않으므로 에러가 생길 수 있음

1. next.config.js 수정

```jsx
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'qqq.s3.amazonaws.com',
        port: '',
        pathname: '/**',
      },
    ],
  },
};
```

1. 업로드하는 사진을 S3에 포워딩
    - aws-sdk/client-s3 library 설치

```jsx
// meals.js
import { S3 } from '@aws-sdk/client-s3';
const s3 = new S3({
  region: 'ap-northeast-2' // 서울
});
const db = sql('meals.db');

.. 
export async function saveMeal(meal) {
  meal.slug = slugify(meal.title, { lower: true });
  meal.instructions = xss(meal.instructions);
 
  const extension = meal.image.name.split('.').pop();
  const fileName = `${meal.slug}.${extension}`;
 
  const bufferedImage = await meal.image.arrayBuffer();
 
  s3.putObject({
    Bucket: 'qqq',
    Key: fileName,
    Body: Buffer.from(bufferedImage),
    ContentType: meal.image.type,
  });
 
 
  meal.image = fileName;
 
  db.prepare(
    `
    INSERT INTO meals
      (title, summary, instructions, creator, creator_email, image, slug)
    VALUES (
      @title,
      @summary,
      @instructions,
      @creator,
      @creator_email,
      @image,
      @slug
    )
  `
  ).run(meal);
}
```

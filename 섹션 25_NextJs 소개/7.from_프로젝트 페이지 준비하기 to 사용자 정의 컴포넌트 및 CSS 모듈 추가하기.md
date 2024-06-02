### 프로젝트 페이지 준비하기

- 기본 프로젝트의 페이지가 모두 비어있음
- 기본 페이지는 모임 목록 페이지, 모임 추가 페이지, 모임 상세 페이지가 필요
- App 라우터가 아닌 페이지 라우터의 경우 index.js가 기본 페이지 역할
- 다른 페이지는 파일 명대로 하위 도메인을 따라감
- 상세 페이지(id에 따른 동적 페이지)는 대괄호([])

-pages

ㄴ[meetupId]

ㄴindex.js

ㄴnew-meetup

ㄴindex.js

ㄴindex.js

### Meetup 목록 출력하기

- 메인 페이지에서 MeetupList 컴포넌트를 불러올 예정
- 컴포넌트 폴더명은 pages 폴더와 달리 이름명에 제약이 없음

```jsx
import MeetupList from "../components/meetups/MeetupList";

const DUMMY_MEETUPS = [
	{
		id: "1",
		title: "Eiffel Tower",
		image:
			"https://upload.wikimedia.org/wikipedia/commons/8/85/Tour_Eiffel_Wikimedia_Commons_%28cropped%29.jpg",
		address: "Champ de Mars, 5 Avenue Anatole France, 75007 Paris, France",
		description:
			"The Eiffel Tower is a wrought-iron lattice tower on the Champ de Mars in Paris, France. It is named after the engineer Gustave Eiffel, whose company designed and built the tower.",
	},
	{
		id: "2",
		title: "Great Wall of China",
		image:
			"https://upload.wikimedia.org/wikipedia/commons/2/23/The_Great_Wall_of_China_at_Jinshanling-edit.jpg",
		address: "Huairou District, China",
		description:
			"The Great Wall of China is a series of fortifications that were built across the historical northern borders of ancient Chinese states and Imperial China as protection against various nomadic groups.",
	},
	{
		id: "3",
		title: "Statue of Liberty",
		image:
			"https://upload.wikimedia.org/wikipedia/commons/d/dd/Lady_Liberty_under_a_blue_sky_%28cropped%29.jpg",
		address: "Liberty Island, New York, NY 10004, United States",
		description:
			"The Statue of Liberty is a colossal neoclassical sculpture on Liberty Island in New York Harbor in New York City, in the United States. The copper statue, a gift from the people of France, was designed by French sculptor Frédéric Auguste Bartholdi and its metal framework was built by Gustave Eiffel.",
	},
];

export default function HomePage() {
	return (
		<>
			<MeetupList meetups={DUMMY_MEETUPS} />
		</>
	);
}
// 사진 형식이 제대로 안 나옴
```

### 새 Meetup 양식 출력하기

- Meetup 추가페이지

```jsx
import NewMeetupForm from "../../components/meetups/NewMeetupForm";

export default function NewMeetupPage() {
  const addMeetupHandler = (enteredMeetupData) => {
    
  }

	return (
		<>
			<NewMeetupForm addMeetup={addMeetupHandler}/>
		</>
	);
}

```

- Form 입력 후 제출하면 콘솔로 제대로 출력되는 걸 확인
- 현재는 레이아웃과 라우팅이 없음

### app.js 파일 및 레이아웃 감싸기

- Layout 폴더의 layout을 사용할 예정
- layout 함수로 여러 페이지에서 동일한 레이아웃을 사용할 수 있음

```jsx
// index.js
import Layout from "../components/layout/Layout"
...
		<Layout>
			<MeetupList meetups={DUMMY_MEETUPS} />
		</Layout>
```

- Layout 함수에서 Next의 Link 함수를 사용하므로 에러 발생 → import
- 하위 페이지인 new-meetup에는 레이아웃이 적용이 안 되어 있음
    - 페이지 수가 많아질수록 번거로움
    - app.js 파일에 레이아웃을 적용해서 모든 페이지에 적용할 수 있음(Root Component)

`

### 프로그래밍 방식 탐색 사용하기

- 동적인 세부 페이지를 만들예정
- Link 컴포넌트 외에 프로그래밍 방식으로 링크 이동 방법

```jsx
const showDetailsHandler = () => {
    router.push('/' + props.id);
}
...
				<div className={classes.actions} onClick={showDetailsHandler}>
					<button>Show Details</button>
				</div>
```

### 사용자 정의 컴포넌트 및 CSS 모듈 추가하기

- 백엔드 연결 전 임의로 하드코딩한 상세 페이지 추가

```jsx
//MeetupDetail.js
import classes from "./MeetupDetail.module.css";

export default function MeetupDetail(props) {
	return (
		<section className={classes.detail}>
			<img src={props.image} alt={props.title} />
			<h1>{props.title}</h1>
			<address>{props.address}</address>
			<p>{props.description}</p>
		</section>
	);
}

//index.js
import MeetupDetail from "../../components/meetups/MeetupDetail";

export default function MeetupDetailPage() {
	return (
		<>
			<MeetupDetail
				image="https://upload.wikimedia.org/wikipedia/commons/d/dd/Lady_Liberty_under_a_blue_sky_%28cropped%29.jpg"
				title="A First Meeting"
				address="Some Street, Racoon City"
				description="Description"
			/>
		</>
	);
}
//MeetupDetail.module.css
.detail {
	text-align: center;
}

.detail img {
	width: 100%;
	height: 100%;
}

```

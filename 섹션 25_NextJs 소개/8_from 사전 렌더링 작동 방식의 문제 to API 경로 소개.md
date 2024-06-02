## 사전 렌더링 작동 방식의 문제

- 원래 리액트에서는 데이터를 받아오고 화면을 구성할 때 useEffect를 이용했음
    - 이 방식에서는 데이터를 받아오기 전 빈 배열과 받아온 후의 상태에 따라 두 번 렌더링 함
- 그러나 넥스트에서는 사전 렌더링을 진행하기 때문에 검색 엔진 최적화 문제가 발생함

</ br>

## 정적 페이지에 대한 데이터 가져오기

![next](https://github.com/sujinann/ReactStudy/assets/139312979/56d3bb9d-59f9-4a07-9f01-6822082e9ad5)

- 정적 생성에서 페이지가 사전 렌더링 되는 시점은 프로덕션 용으로 빌드하는 시점
    - 요청이 서버에 도달했을 때 사전 렌더링을 갱신하는 것이 아님
    - 빌드 시 갱신되는 것
    - 그러므로 기본적으로 사전 렌더링을 갱신하고 싶으면 재배포를 수행하여야 함
    - getStaticProps 페이지 컴포넌트 안에서만 작동하는 예약어

```jsx
function HomePage(props) {
  return <MeetupList meetups={props.meetups} />;
}

export async function getStaticProps() {
  // fetch data from an API
  return {
    props: {
      meetups: DUMMY_MEETUPS
    }
  }; 
}

export default HomePage;
```

</ br>

## 정적 사이트 생성(SSG)에 대한 추가 정보

- 위의 해결책 만으론 여전히 최신 데이터를 반영하지 못한다는 문제점이 남음
- revalidate 를 통해 데이터 갱신을 반영한 사전 렌더링을 적용 할 수 있음(초 단위)

```jsx
function HomePage(props) {
  return <MeetupList meetups={props.meetups} />;
}

export async function getStaticProps() {
  // fetch data from an API
  return {
    props: {
      meetups: DUMMY_MEETUPS
    },
    revalidate: 1
  }; 
}

export default HomePage;
```

</ br>

## getSeverSideProps를 사용하여 서버 사이드 렌더링(SSR) 탐색하기

- 위의 방법도 좋은 방법이지만 일정 주기가 아니라 아예 데이터가 갱신 될 때마다 페이지를 다시 사전 생성 하고 싶을 경우에도 방법이 있음
- getStaticProps가 캐시를 이용할 수 있기 때문에 기본적으로 더 빠름
- 데이터가 자주 갱신되는 경우나 요청 객체에 접근할 필요가 있는 경우 getSeverSideProps 사용

```jsx
function HomePage(props) {
  return <MeetupList meetups={props.meetups} />;
}

export async function getServerSideProps(context) {
  const req = context.req;
  const res = context.res;

  // fetch data from an API

  return {
    props: {
      meetups: DUMMY_MEETUPS
    }
  };
}

// export async function getStaticProps() {
//   // fetch data from an API
//   return {
//     props: {
//       meetups: DUMMY_MEETUPS
//     },
//     revalidate: 1
//   }; 
// }

export default HomePage;
```

</ br>

## SSG 데이터를 가져오기 위한 Params 작업하기

- useRouter를 이용해 Params를 가져오고 싶지만 getStaticProps 에서는 리액트 훅을 이용할 수 없음
    - 따라서 context.params 이용

```jsx
import MeetupDetail from '../../components/meetups/MeetupDetail';

function MeetupDetails() {
  return (
    <MeetupDetail
      image='https://upload.wikimedia.org/wikipedia/commons/thumb/d/d3/Stadtbild_M%C3%BCnchen.jpg/1280px-Stadtbild_M%C3%BCnchen.jpg'
      title='First Meetup'
      address='Some Street 5, Some City'
      description='This is a first meetup'
    />
  );
}

export async function getStaticPaths() {
  return {
    fallback: false,
    paths: [
      {
        params: {
          meetupId: 'm1',
        },
      },
      {
        params: {
          meetupId: 'm2',
        },
      },
    ],
  };
}

export async function getStaticProps(context) {
  // fetch data for a single meetup

  const meetupId = context.params.meetupId;

  console.log(meetupId);

  return {
    props: {
      meetupData: {
        image:
          'https://upload.wikimedia.org/wikipedia/commons/thumb/d/d3/Stadtbild_M%C3%BCnchen.jpg/1280px-Stadtbild_M%C3%BCnchen.jpg',
        id: meetupId,
        title: 'First Meetup',
        address: 'Some Street 5, Some City',
        description: 'This is a first meetup',
      },
    },
  };
}

export default MeetupDetails;
```

</ br>

## getStaticPaths 로 경로 준비 및 대체 페이지 작업하기

- 다른 페이지에도 동적으로 반영되어야 하기 때문에 그것을 알려주는 과정 필요
- 위의 코드 참조
- 여기서는 예시라 pahts 값을 하드코딩 했지만 실제로는 동적 값으로 적용 필요
- fallback 설정 필수
    - 모든 paths 값에 대해 사전 렌더링 false
    - 일부 값에만 사전 렌더링 적용 후 해당 페이지 요청 마다 사전 렌더링 true

</ br>

## API 경로 소개

- pages/api/meetups.js 이런 경로로 이용함
    - pages/meetups/index.js 에서 이용하는 api 경로

```jsx
import { MongoClient } from 'mongodb';

// /api/new-meetup
// POST /api/new-meetup

async function handler(req, res) {
  if (req.method === 'POST') {
    const data = req.body;

    const client = await MongoClient.connect(
      'mongodb+srv://maximilian:arlAapzPqFyo4xUk@cluster0.ntrwp.mongodb.net/meetups?retryWrites=true&w=majority'
    );
    const db = client.db();

    const meetupsCollection = db.collection('meetups');

    const result = await meetupsCollection.insertOne(data);

    console.log(result);

    client.close();

    res.status(201).json({ message: 'Meetup inserted!' });
  }
}

export default handler;
```

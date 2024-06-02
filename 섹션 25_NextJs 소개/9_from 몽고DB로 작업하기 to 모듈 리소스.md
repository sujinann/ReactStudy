# 몽고DB로 작업하기 ~ 모듈 리소스

### 몽고DB로 작업하기

- 무료 다운로드
    - MongoDB Atlas
- Clusters
    - 새 Cluster 생성 가능
        - 무료: free-tier로 설정
    - M0 Sandbox 사용
- Network Access
    - IP 주소 추가
        - local IP 추가 입력
        - local 컴퓨터가 몽고DB에 요청을 보낼 수 있게 함
- Database Access
    - 데이터베이스 읽기 쓰기 권한을 가지는 사용자 최소 1명 생성
        - 사용자 이름 지정
- Clusters
    - 애플리케이션을 연결
    - Node.js MongoDB 드라이버를 사용해 MongoDB Atlas 클러스터에 조회를 보내는 API 경로에 코드 작성 필요
- 터미널(VS Code)
    - MongoDB 패키지에 npm 추가 설치하는 패키지를 추가
        - 몽고DB에 쉽게 쿼리를 보내도록 해줌
        - 몽고DB 드라이버가 해당 클러스터에 접속해 데이터를 삽입하거나 가져올 수 있게 됨
    
    ```java
    npm install mongodb
    ```
    
    - MongoClient 객체를 mongodb로부터 import
    
    ```java
    // new-meetup.js
    
    import { MongoClient } from 'mongodb';
    ```
    
    - MongoClient를 사용해 connect 메소드 호출
    
    ```java
    // new-meetup.js
    
    function handler(req, res) {
    	if (req.method === 'POST') {
    		// 중략
    		MongoClient.connect();
    	}
    }
    ```
    
    - 클러스터 연결 모달창에 있는 connection string을 복사해 connect 메소드의 매개 변수로 붙여넣어 사용 (작은 따옴표 필요)
        - 클라이언트 측에서 절대 실행될 수 없도록 해야 하는데, 이 API 코드는 안전해서 보안은 상관 없다!
    
    ```java
    // new-meetup.js
    
    function handler(req, res) {
    	if (req.method === 'POST') {
    		// 중략
    		MongoClient.connect('복사해 온 connection string 붙여넣기');
    	}
    }
    ```
    
    - <username>:<password>에 실제 데이터로 수정
        - username : 읽기 쓰기 권한을 가지도록 설정했던 사용자의 이름
            - Database Access의 해당 사용자의 user name 복사 및 붙여넣기
        - password : 읽기 쓰기 권한을 가지도록 설정했던 사용자의 비밀번호
            - Database Access의 해당 사용자의 Actions에서 Edit 누르고 edit password를 누른 뒤 autogenerate secure password를 눌러 생성한 후 copy 눌러 복사 및 붙여넣기
    - myFirstDatabase에 원하는 데이터베이스명으로 변경 (강의에서는 meetups로 진행)
    - function handler에 await, async 추가
    
    ```java
    // new-meetup.js
    
    async function handler(req, res) {
    	if (req.method === 'POST') {
    		// 중략
    		await MongoClient.connect('수정한 connection string');
    	}
    }
    ```
    
    - await 코드를 client로 받아 db 메소드를 호출해 meetups에 연결 중인 데이터베이스를 확보
        - 데이터베이스가 meetupsCollection에 접근할 수 있게 됨
            - 이제 몽고DB는 문서(tables의 항목)들로 가득 찬 컬렉션(SQL 데이터베이스의 tables)을 작동시키는 NoSQL 데이터베이스가 됨
        - 단일 meetup은 하나의 문서가 되고, 전체 컬렉션은 여러 meetup 문서들을 보관
    
    ```java
    // new-meetup.js
    
    async function handler(req, res) {
    	if (req.method === 'POST') {
    		// 중략
    		const client = await MongoClient.connect('수정한 connection string');
    		const db = client.db();
    		
    		const meetupsCollection = db.collection();
    	}
    }
    ```
    
    - collection 함수 안에 작은 따옴표 후 컬렉션 이름을 원하는 대로 설정할 수 있음 (강의에서는 meetups로 진행)
        - 물론 데이터베이스와 다른 이름이어도 무방!
    
    ```java
    // new-meetup.js
    
    const meetupsCollection = db.collection('meetups');
    ```
    
    - meetupsCollection에서 insertOne 호출
        - 컬렉션에 새 문서 삽입하는 역할
        - 지난 번 req.body를 받아왔던 data를 insertOne에 집어넣어 데이터베이스에 저장 가능
            - await 추가 후 result로 받음
        
        ```java
        // new-meetup.js
        
        const meetupsCollection = db.collection('meetups');
        
        const result = await meetupsCollections.insertOne(data);
        
        console.log(result); // 콘솔 출력으로 확인 가능
        ```
        
        - 지난 번 작성했던 객체 형태의 destructuring은 필요 없어짐 → 지운다
        
        ```java
        // const { title, image, address, description } = data;   -> 코드 삭제
        ```
        
    - client.close를 호출해 데이터베이스 연결 차단
    
    ```java
    // new-meetup.js
    
    client.close();
    ```
    
    - 응답 객체 res를 활용해 다시 응답을 보냄
        - status를 호출해 HTTP status 코드를 설정
            - 201 : 성공적인 삽입
        - json 호출로 응답에 추가될 json 데이터를 준비
    
    ```java
    // new-meetup.js
    
    res.status(201).json({ message: 'Meetup inserted!' });
    ```
    

### API 경로로 Http 요청 보내기

- 터미널(VS Code)
    - new-meetup 폴더의 index.js에 작성된 addMeetupHandler 함수에 API를 완성하자
        - request를 이 쪽으로 보내준다
            - fetch 함수를 활용하거나 (async, await도 작성해준다)
            - axios라는 타사 패키지를 사용할 수도 있음
            - 여기서는 fetch 함수를 활용하기로 한다!
        
        ```java
        // index.js
        
        function NewMeetupPage() {
        	async function addMeetupHandler(enteredMeetupData) {
        		const response = await fetch();
        	}
        
        	return <NewMeetupForm onAddMeetup={addMeetupHandler} />
        }
        ```
        
        - fetch 함수에 request로 받아올 URL을 집어넣는다
            - 여기서는 외부 API가 아닌 내장 API를 한 서버에서 사용하기 때문에 절대경로로 작성해도 무방 (’/api/new-meetup’)
        
        ```java
        // index.js
        
        function NewMeetupPage() {
        	async function addMeetupHandler(enteredMeetupData) {
        		const response = await fetch('https://some-domain.com/abc');
        	}
        
        	return <NewMeetupForm onAddMeetup={addMeetupHandler} />
        }
        ```
        
        - request의 path를 받게 되면 api 폴더의 new-meetup.js에 작성된 handler 함수가 trigger되는 원리이다
        - method에 POST, body에 request의 데이터를 저장할 자바스크립트 객체(중괄호 안에), 필요 시 headers에 content-type를 작성한다
            - (addMeetupHandler 함수에 들어갈 enteredMeetupData에 해당하는) NewMeetupForm에 작성된 meetupData를 보면 title, image, address, description이 이미 작성되어 있어 body에 JSON으로 전환한 enteredMeetupData를 넣어준다
        
        ```java
        // index.js
        
        function NewMeetupPage() {
        	async function addMeetupHandler(enteredMeetupData) {
        		const response = await fetch('https://some-domain.com/abc', {
        			method: 'POST',
        			body: JSON.stringify(enteredMeetupData)
        			headers: {
        				'Content-Type': 'application/json'
        			}
        		});
        	}
        
        	return <NewMeetupForm onAddMeetup={addMeetupHandler} />
        }
        ```
        
        - 받아온 응답을 data에 저장할 수도 있고, 그걸 활용할 수도 있음
        
        ```java
        // index.js
        
        function NewMeetupPage() {
        	async function addMeetupHandler(enteredMeetupData) {
        		const response = await fetch('https://some-domain.com/abc', {
        			method: 'POST',
        			body: JSON.stringify(enteredMeetupData)
        			headers: {
        				'Content-Type': 'application/json'
        			}
        		});
        		
        		const data = await response.json();
        		
        		console.log(data); // 여기서는 단순 기록용
        	}
        
        	return <NewMeetupForm onAddMeetup={addMeetupHandler} />
        }
        ```
        
        - 개발자도구를 연 상태에서 title, image, address, description 데이터를 입력하고 Add Meetup 버튼을 누르면 콘솔에 message: “Meetup inserted!” 라는 응답을 확인할 수 있음
- Clusters
    - Collections 클릭
    - meetups를 눌러 데이터베이스 확인 가능
    - 아까 추가한 데이터가 보일 것 (id는 자동 생성됨)
- 터미널(VS Code)
    - new-meetup 폴더의 index.js로 돌아와 페이지 라우팅을 작성하자
        - useRouter 및 router.push 혹은 router.replace 이용
        
        ```java
        // index.js
        
        import { useRouter } from 'next/router';
        
        function NewMeetupPage(){
        	const router = useRouter();
        
        	// 중략
        	
        	router.push('/');
        	// router.replace(); => replace : 이전 페이지로 돌아가지 못함
        }
        ```
        
        - 데이터 입력 후 Add Meetup 버튼을 누르면 이제는 해당 페이지로 정상적으로 이동함

### 데이터베이스에서 데이터 가져오기

- 아직 화면에는 더미 데이터가 보이는 상태: 데이터베이스에서 데이터를 가져와 화면에 보이도록 해보자
- 터미널(VS Code)
    - api 폴더에 meetups.js 파일을 새로 생성해 handler 함수를 추가해 몽고DB에서 meetups를 모두 가져오는 코드를 작성하도록 하자
    - pages 폴더의 index.js(시작 페이지)의 getStaticProps 함수 안에 fetch 함수를 추가하면 nextJS가 서버 코드에서도 fetch를 사용 가능하도록 알아서 기능을 추가함 (원래는 로컬에서만 가능) ⇒ fetch 함수 안에 request를 보낼 API의 URL을 입력하고 props을 입력해볼 수 있음
    - meetups.js 파일과 fetch 함수를 추가할 때의 문제점
        - 이 모든 과정은 번거롭고, 중복될 수 있음
        - 클라이언트에서 실행되지 않고 빌드 시에만 실행되기 때문에 코드가 클라이언트에 노출되지 않음
    - 대안
        - getStaticProps 내부나 helper 함수에 직접 코드를 작성해야 함 (= 이 곳에서 바로 코드를 실행하고 API 경로에 request를 따로 보내지 않는다)
        - meetups.js 파일은 필요 없으니 다시 삭제
    - 시작 페이지(pages 폴더의 index.js 파일)에서 MongoClient를 import
        - getServerSide props나 getStaticProps에서만 사용될 경우 클라이언트 측 번들에 import 패키지가 포함되지 않음 (= 서버에서만 실행되고, nextJS가 감지해 클라이언트 측 번들에서 제외하기 때문에 보안에도 유용)
    
    ```java
    // index.js
    
    import { MongoClient } from 'mongodb';
    ```
    
    - getStaticProps 함수 안에 MongoClient.connect를 작성
    
    ```java
    // index.js
    
    export async function getStaticProps() {
    	MongoClient.connect();
    }
    ```
    
    - api 폴더의 new-meetup.js에 작성했던 데이터베이스 연결 코드를 그대로 복사해온다
    
    ```java
    // index.js
    
    export async function getStaticProps() {
    	const client = await MongoClient.connect(
    		'(지난 번에 작성했던 몽고DB 정보)'
    	};
    	const db = client.db();
    	
    	const meetupsCollection = db.collection('meetups');
    	
    	// 후략
    }
    ```
    
    - meetupsCollection에서 find 메소드를 호출해 컬렉션의 모든 문서를 찾아 promise를 반환
        - 배열로 받기 위해 toArray를 써서 meetups에 저장
    
    ```java
    // index.js
    
    export async function getStaticProps() {
    	const client = await MongoClient.connect(
    		'(지난 번에 작성했던 몽고DB 정보)'
    	};
    	const db = client.db();
    	
    	const meetupsCollection = db.collection('meetups');
    	
    	const meetups = await meetupsCollection.find().toArray();
    	
    	// 후략
    }
    ```
    
    - 반환값의 props에 작성했던 meetups 값을 meetups로 수정하고, client.close로 연결 차단
    
    ```java
    // index.js
    
    export async function getStaticProps() {
    	const client = await MongoClient.connect(
    		'(지난 번에 작성했던 몽고DB 정보)'
    	};
    	const db = client.db();
    	
    	const meetupsCollection = db.collection('meetups');
    	
    	const meetups = await meetupsCollection.find().toArray();
    	
    	client.close();
    	
    	return {
    		props: {
    			meetups: meetups // 수정
    		},
    		revalidate: 1
    	};
    }
    ```
    
    - 그러나 오류 발생 ⇒ 자동 생성된 id 때문
        - id가 데이터로 반환될 수 없는 복잡한 객체이기 때문
    - meetups를 map으로 변형해 객체를 반환하도록 변경해준다
        - 이 때, id도 추가해 사용 가능한 문자열로 변형해야 한다
    
    ```java
    // index.js
    
    	return {
    		props: {
    			meetups: meetups.map(meetup => ({
    				title: meetup.title,
    				address: meetup.address,
    				image: meetup.image,
    				description: meetup.description
    				id: meetup._id.toString()
    			}))
    		},
    		revalidate: 1
    	};
    ```
    
    - 실행하면 meetup 데이터를 기반으로 한 목록이 화면에 뜨게 됨
        - 더미 데이터를 삭제해도 문제 없음

### Meetup 세부 정보 데이터 가져오기 및 페이지 준비하기

- 서버 측 코드를 getStaticProps에서 실행되도록 해보자 = 표준 방식
    - 페이지가 pre-generate될 때까지만 코드가 실행되어 개인 정보도 안전함
- 터미널(VS Code)
    - [meetupId] 폴더의 index.js(= MeetupDetail 페이지)에서 몽고DB에 연결해 id를 가져오도록 해야 함
    - 아까 전의 몽고DB 연결 코드를 재차 복사해 이 곳에도 붙여넣기
        - find의 첫 번째 argument로는 모든 객체를 가져오기 위해 빈 객체를 전달하고
        - find의 두 번째 argument로는 추출되어야 하는 필드를 정의할 객체 안에 _id: 1을 전달해 id만 반환해온다
            - 만약 모든 필드를 반환하고 싶다면 두 번째 argument도 빈 객체를 전달한다
        - toArray를 붙여 자바스크립트 객체 배열로 변환
    
    ```java
    // index.js
    
    import { MongoClient } from 'mongodb';
    
    export async function getStaticPaths() {
    	const client = await MongoClient.connect(
    		'(아까 그 몽고DB 정보)'
    	);
    	const db = client.db();
    	
    	const meetupsCollection = db.collection('meetups');
    	
    	const meetups = await meetupsCollection.find({}, { _id: 1 }).toArray();
    	
    	// 후략
    }
    ```
    
    - 아까 했던 것처럼 반환값의 paths에 meetups의 map을 활용한다
        - 기존에 더미에서 작성했던 틀대로 params라는 key를 작성해주고 위에서 찾아온 _id를 string으로 바꿔 동적으로 경로 배열을 생성하게끔 한다
        - 반환 직전에 client.close로 연결을 끊어준다
    
    ```java
    // index.js
    
    import { MongoClient } from 'mongodb';
    
    export async function getStaticPaths() {
    	const client = await MongoClient.connect(
    		'(아까 그 몽고DB 정보)'
    	);
    	const db = client.db();
    	
    	const meetupsCollection = db.collection('meetups');
    	
    	const meetups = await meetupsCollection.find({}, { _id: 1 }).toArray();
    	
    	client.close();
    	
    	return {
    		fallback: false,
    		paths: meetups.map(meetup => ({
    			params: { meetupId: meetup._id.toString() },	
    		})),		
    	};
    }
    ```
    
    - Show Details를 눌러보면 해당 meetup에 대한 상세 페이지로 이동하게 됨
    - 그러나 아직 동적으로 데이터를 가져오지는 못하고 있음
    - 아까 그 몽고DB 연결 코드를 다시 복사해서 하단의 getStaticProps 함수 안에 붙여 넣고 콘솔로 출력하던 코드를 지운다
        - meetups와 find 메소드로 되어있던 코드를 selectedMeetup과 findOne 메소드로 수정해 단 하나의 meetup만 가져오게끔 변경
        - findOne 안에 검색하고자 하는 id로 객체 값을 작성해준다
    
    ```java
    // index.js
    
    export async function getStaticProps(context) {
    	const meetupId = context.params.meetupId;
    	
    	const client = await MongoClient.connect(
    		'(아까아까의 그 몽고DB 정보)'
    	);
    	const db = client.db();
    	
    	const meetupsCollection = db.collection('meetups');
    	
    	const selectedMeetup = await meetupsCollection.findOne({ _id: meetupId });
    	// const meetups = await meetupsCollection.find({}, { _id: 1 }).toArray();	
    	
    	client.close();	
    
    	// 후략
    }
    ```
    
    - 반환값에서 meetupData를 selectedMeetup으로 수정
        - ObjectId를 import
        - findOne에서의 meetupId를 ObjectId로 감싸 id를 정확하게 찾을 수 있도록 처리해줌
        - 반환값인 selectedMeetup의 _id도 문자열로 변환하도록 toString 처리를 해주고 다른 key들도 작성해준다
    
    ```java
    // index.js
    
    import { MongoClient, ObjectId } from 'mongodb';
    
    export async function getStaticProps(context) {
    	const meetupId = context.params.meetupId;
    	
    	const client = await MongoClient.connect(
    		'(아까아까의 그 몽고DB 정보)'
    	);
    	const db = client.db();
    	
    	const meetupsCollection = db.collection('meetups');
    	
    	const selectedMeetup = await meetupsCollection.findOne({ _id: ObjectId(meetupId) });
    	// const meetups = await meetupsCollection.find({}, { _id: 1 }).toArray();	
    	
    	client.close();	
    
    	return {
    		props: {
    			meetupData: {
    				id: selectedMeetup._id.toString(),
    				title: selectedMeetup.title,
    				address: selectedMeetup.address,
    				image: selectedMeetup.image,
    				description: selectedMeetup.description,
    			}
    		}.
    	};
    }
    ```
    
    - 다시 실행해보면 모든 데이터를 성공적으로 보여주게 되긴 하지만, 여전히 JSX 코드로 하드코딩된 상태임
    - props를 활용해보자
        - 상단의 MeetupDetails 함수에 props를 받아오게끔 하고 반환값에 props의 (meetupData의) 속성들을 각각 전달하도록 변경
        
        ```java
        // index.js
        
        function MeetupDetails(props) {
        	return {
        		<MeetupDetail
        			image={props.meetupData.image}
        			title={props.meetupData.title}
        			address={props.meetupData.address}
        			description={props.meetupData.description}
        		/>
        	};
        }
        ```
        
    - 다시 실행해보면 모든 meetups가 정상적으로 작동함

### “head” 메타데이터 추가하기

- 원격 기기에 배포(deploy)하는 작업을 해보자
    - 페이지에 메타데이터를 추가해보자
    - 개발자 도구의 elements에서 확인해보면 head section이 상대적으로 비어져 있음
    - 검색했을 때 본인이 지정한 웹사이트 이름과 설명이 나오도록 설정하는 작업
- 터미널(VS Code)
    - pages 폴더의 index.js 파일(시작 페이지)에서 Head를 import
        - head section에 head 태그를 추가하는 역할
    
    ```java
    // index.js
    
    import Head from 'next/head';
    ```
    
    - HomePage 함수의 반환값에 Head를 추가하고, Fragment로 묶어준다 (Fragment도 import에 추가)
        - head 태그로 필요한 것들을 추가 작성 ⇒ 개발자 도구 elements에도 추가되었음을 확인 가능
    
    ```java
    // index.js
    
    import Head from 'next/head';
    import { Fragment } from 'react';
    
    function HomePage(props) {
    	return
    		<Fragment>
    			<Head>
    				<title>React Meetups</title>
    				<meta name="description" content="Browse a huge list of highly active React meetups!" />
    			</Head>
    			<MeetupList meetups={props.meetups} />
    		</Fragment>
    }
    ```
    
    - new-meetup 폴더의 index.js 파일에서도 반환값에 Head를 추가하고, Fragment로 묶음 (필요한 import도 추가)
    
    ```java
    // index.js
    
    import { Fragment } from 'react';
    import Head from 'next/head';
    
    function NewMeetupPage() {
    	async function addMeetupHandler(enteredMeetupData) {
    		// 중략
    	}
    	
    	return (
    		<Fragment>
    			<Head>
    				<title>Add a New Meetup</title>
    				<meta name="description" content="Add your own meetups and create amazing networking opportunities." />
    			</Head>
    			<NewMeetupForm onAddMeetup={addMeetupHandler} />
    		</Fragment>
    	);
    }
    ```
    
    - [meetupId] 폴더의 index.js에서도 같은 작업 반복
        - 각 meetup에 따라 동적으로 title과 description 태그가 반영되도록 중괄호 안에 props.meetupData에서 뽑아오도록 함
    
    ```java
    // index.js
    
    import { Fragment } from 'react';
    import Head from 'next/head';
    
    function MeetupDetails(props) {
    	return (
    		<Fragment>
    			<Head>
    				<title>{props.meetupData.title}</title>
    				<meta name="description" content={props.meetupData.description} />
    			</Head>
    			<MeetupDetail
    				image={props.meetupData.image}
    				title={props.meetupData.title}
    				address={props.meetupData.address}
    				description={props.meetupData.description}
    			/>
    		</Fragment>
    	);
    }
    ```
    
    - 개발자 도구를 확인해보면 모두 반영되어 있음 ⇒ deploy 준비가 끝남

### Next.js 프로젝트 배포하기

- Next.js 프로젝트를 deploy해보자
    - 이 강의에서는 NextJS 호스팅 제공자인 Vercel을 이용했음
        - Git 레포지토리에 연결해 해당 소스 코드를 바로 배포 가능 (여기서는 Github을 활용)
    - Github에서 새 레포지토리 생성 (명칭, 설명, 옵션 등은 본인 재량껏)
    - 로컬 컴퓨터에 Git 설치 (이후 과정은 Git 관련이어서 생략됨)
        - commit이라는 코드 스냅샷을 생성해 Github에 push
    - git remote add origin 해당레포지토리URL
        - 첫 push일 경우 자격 인증을 위해 URL의 github.com 앞에 사용자 이름과 골뱅이(@) 기호를 붙인다
        - git push —all 명령어로 모든 코드를 깃헙에 push한다
    - settings에서 developer settings에 들어가 personal access tokens 메뉴에서 generate를 눌러 새 토큰을 생성
        - 배포 등의 목적으로 생성
        - 하단의 select scopes에서는 repo, delete_repo, admin:repo_hook에만 체크하고 생성해준다
        - 생성된 토큰을 복사해 암호 입력 메세지에서 활용
    - Vercel에서 방금 생성된 깃헙 레포지토리를 연결해 배포한다
        - 원래는 터미널에서 npm run build로 .next 폴더를 생성할 수 있는데, Vercel에서 자동 실행해줌
        - import 버튼을 눌러 해당 레포지토리와 개인 계정을 선택하고 deploy를 누른다
    - MongoDB Atlas로 돌아가서 Network Access에 있는 Add IP Address 버튼을 누르고 Allow Access From Anywhere 버튼을 눌러 어디에서나 액세스 가능하도록 설정해줘야 함 ⇒ 그래야 Vercel이 MongoDB에 연결할 수 있게 됨
        - Visit 버튼을 눌러 배포된 사이트를 확인해볼 수 있다
        - 그런데 예상대로 404 오류가 뜨기도 함

### 대체 페이지 사용 및 재배포하기

- 404 오류의 원인
    - 미리 생성되어야 하는 meetup 페이지가 build time에서만 설정되도록 해 그 이후로는 설정되지 않았기 때문
    - fallback이 false였기 때문에 미리 생성된 페이지가 없다는 request가 실패하게 됨
- 해결법
    - fallback을 true로 설정 ⇒ 빈 페이지가 즉시 반환됨 (데이터가 없는 경우를 처리해줘야 함)
    - 혹은 fallback을 blocking으로 설정 ⇒ 페이지가 미리 생성될 때까지 사용자는 아무것도 볼 수 없게 됨
- NextJS가 곧바로 페이지를 찾을 수 없을 때 404로 응답하지 않게 됨
    - 요청 시 미리 페이지를 생성해 캐시에 저장
    - 새 commit을 생성하면 자동으로 redeploy가 작동함

### 요약

- NextJS에서 제공하는 추가 옵션도 아직 많이 있긴 함
    - getStaticProps에서 반환할 수 있는 옵션들이 더 있음
    - 내장된 Image 컴포넌트로 이미지 최적화 가능
    - 간편한 인증 추가 가능 (세션 관리도 용이해짐)

### 모듈 리소스

- 데모 프로젝트 코드 스냅샷과 섹션 슬라이드는 [이 곳](https://github.com/academind/react-complete-guide-code/tree/23-nextjs-introduction)에서 확인 가능

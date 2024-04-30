### 데이터베이스 연결/해지하는 법

- 어플리케이션 데이터 관리를 위해 별도의 중앙 단일 DB가 필요함
- 리액트 앱과 DB를 직접 연결한다면?
    - 리액트 앱은 모든 방문자의 브라우저에서 실행되므로 둘이 연결된 경우 모든 코드가 노출되므로 사용자가 백엔드 DB를 직접 다룰 수 있음
    - 파일 시스템(중앙 서버, 컴퓨터 속 공유되지 않은 시스템 파일)에 접근 가능
- 대신 API로 원격 백엔드 서버와 통신함
    - 백엔드가 통신을 허용한 기능만 사용할 수 있음 → 서버에서 사용자를 통제 가능
    - 특정 기능에 대해 통신을 백엔드 endpoint 지정

### 앱의 데이터 Fetching을 위한 준비

- 백엔드 서버를 별도의 터미널에서 실행
    - Node.js 설치
    - terminal → node app.js
- 갈 수 있는 장소들을 백엔드에서 fetching하기
    - 백엔드에 저장된 데이터를 endpoint에서 API로 받아올 예정
    - AvailablePlaces.jsx에서 fetch API 작성
    - 백엔드 통신 과정은 즉각적이지 않음 → 로컬 코드는 바로 실행
        - 컴포넌트를 먼저 렌더링 한 후 데이터를 받아오면 ui를 리렌더링 해줘야함

### HTTP 요청을 잘못 보내는 방법

- fetch()는 HTTP 요청을 다른 서버로 보낼 때 씀
    - 서버의 url을 매개변수로 받음
    - Http Response를 감싸는 Promise를 반환
        - Response의 State에 따라 다른 값을 산출
        - fetch 후 then 안에 메서드를 전달하면 Promise 해결 후 함수를 실행
    
    ```jsx
    fetch('http://localhost:3000/places').then(() => {})
    ```
    
    - 모던 JS에서는 Http 비동기성도 감안해서 await 같이씀
    
    ```jsx
    const response = await fetch('http://localhost:3000/places');
    ```
    
    - 하지만 리액트의 컴포넌트 함수에서 비동기성 함수를 실행할 수 없음(export default async function..)
    - 일반적으로 Http 통신의 데이터 표준(de facto standard)는 JSON임
        - JS 배열과 같지만 모든값을 큰따옴표로 감쌈
    
    ```jsx
    fetch('http://localhost:3000/places').then((response) => {
        response.json();
      }).then((res) => {
        setAvailablePlaces(res.places);
      })
    ```
    
    → 무한 루프 생성
    
    - 두 번째 then에서 state를 업데이트하면서 컴포넌트 함수 재실행을 반복
    
    ### useEffect로 Http 요청 전송하기
    
    - Http 요청을 딱 한번만 실행하게 useEffect 활용
    
    ```jsx
    useEffect(() => {
      fetch("http://localhost:3000/places")
        .then((response) => {
          response.json();
        })
        .then((res) => {
          setAvailablePlaces(res.places);
        });
    }, []);
    ```
    
    - 받아온 데이터의 이미지가 제대로 표시되지 않음
        - 데이터의 img.src는 백엔드 이미지 소스에 저장된 이미지의 이름만 담고있음
        - 프론트엔드에서 template literal을 사용해 백엔드 이미지 주소로 대체 가능
        
        ```jsx
        <img src={`http://localhost:3000/${place.image.src}`} alt={place.image.alt} />
        ```
        
        - 백엔드로 알맞은 이미지를 요청(백엔드에서 이미지 주소를 expose 설정 해야함)

### Async/Await 사용하기

- 비동기 요청에 async/await 사용이 일반적
- useEffect 내에서 async/await 함수를 직접 사용은 불가능
    - 새로운 async 함수를 만들기

```jsx
useEffect(() => {
    async function fetchPlaces() {
      const response = await fetch("http://localhost:3000/places")
      const res = await response.json();
      setAvailablePlaces(res.places);
    };
    fetchPlaces();
  }, []);
```

### 로딩 State 다루기

- Throttling으로 인터넷 속도를 매우 느리게 해서 로딩 속도를 늘리면 사진 데이터를 받아오기 전까지 아무 화면도 보여주지 않음 (로딩 UI가 구림)

→ Fallback 지정해주기

```jsx
//AvailablePlaces
<Places
  title="Available Places"
  places={[]}
  fallbackText="No places available."
  onSelectPlace={onSelectPlace}
  isLoading={true}
  loadingText="Fetching place data.."
/>

// places
{isLoading && <p className="fallback-text">{loadingText}</p>}

```

+fetching 과정에서 loading을 state로 관리해서 로딩 전후를 렌더링 해주기

### Http 에러 다루기

- Http 요청은 항상 에러가 발생할 여지가 있음
    - 네트워크 이슈, 백엔드 이슈, 인터넷 이슈, 서버 오프라인, 버그 등
    - 항상 대비를 해줘야함

```jsx
useEffect(() => {
    async function fetchPlaces() {
    setIsFetching(true);
    try {
      const response = await fetch("http://localhost:3000/places");
      const res = await response.json();
      // 백엔드에서 정상적인 response가 오지 않았음
      if (!response.ok) {
        setError("Error fetching places");
      }
      setAvailablePlaces(res.places);
    } catch (error) {
      setError({message: error.message || "Error, try again later"});
    }
    setIsFetching(false);
  }
  fetchPlaces();
}, []);

if (error) {
  return <Error title="Error" message={error.message}/>
}
```

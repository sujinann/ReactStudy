# 모듈 소개 & 초기 프로젝트 ~ 유동성 있는 커스텀 Hooks(훅) 생성하기

### 모듈 소개 & 초기 프로젝트

- 학습 목표:
    - 훅의 규칙 복습
    - 커스텀 훅의 개념과 필요한 이유
    - 커스텀 훅 직접 만들고 써보기

### “Hooks(훅)의 규칙” 복습 & 훅을 사용하는 이유

- 훅의 규칙
    - 컴포넌트 함수 안에서만 훅을 호출한다
    - 상위에 훅을 호출한다
        - if 조건 함수 안에 중첩되지 않아야
- 커스텀 훅 구축 이유
    - 컴포넌트 함수 내의 코드를 항상 감싸고 재사용하기 위함
        - 만약 여러 곳에서 사용하는 로직이 있는 경우:
        
        ```jsx
        // App.jsx
        
        function fetchData() {
            useEffect(() => {
                async function fetchPlaces() {
                    setIsFetching(true);
                    try {
                        const places = await fetchUserPlaces();
                        setUserPlaces(places);
                    } catch (error) {
                        setError({ message: error.message || 'Failed to fetch user places.' });
                    }
        
                    setIsFetching(false);
                }
        
                fetchPlaces();
            }, []);
        }
        
        function App() {
        
        // 상수/변수들 선언 (중략)
        
            fetchData();
         
        // 함수들 호출 (중략)
        
        }
        ```
        
        - App 컴포넌트 함수 바깥에 새 함수 작성
        - 그 함수 안에 useEffect 훅 코드를 옮기기
        - 해당 함수 필요 시 여러 적당한 위치에서 호출
        - BUT 훅은 컴포넌트 함수 안 혹은 상태 업데이트에서만 사용 가능 ⇒ 제대로 작동하지 않음! 원래 코드로 돌려놓자..
        - THEREFORE, 커스텀 훅이 필요!

### 커스텀 Hooks(훅) 생성하기

- 실습
    
    ```jsx
    // useFetch.js
    
    import { useEffect } from 'react';
    
    function useFetch() {
        useEffect(() => {
            async function fetchPlaces() {
                setIsFetching(true);
                try {
                    const places = await fetchUserPlaces();
                    setUserPlaces(places);
                } catch (error) {
                    setError({ message: error.message || 'Failed to fetch user places.' });
                }
    
                setIsFetching(false);
            }
    
            fetchPlaces();
        }, []);
    }
    
    ```
    
    ```jsx
    // App.jsx
    
        useFetch();
    ```
    
    - hooks 폴더 생성
    - 그 안에 useFetch.js 파일 생성
        - use로 시작 시 리액트가 훅으로 인식하기 때문
        - 만약 fetch로 명명하면 추후 리액트가 커스텀 훅으로 인식하지 않아서 그 안에 들어간 useEffect 훅에 빨간 줄이 뜨게 됨
    - useFetch 안에 아까의 useEffect 훅 옮기기
    - 훅을 사용할 곳에서 useFetch 훅을 호출

### 커스텀 Hooks(훅): State(상태) 관리 & 상태 값 반환

- 실습
    - useFetch.js에 useState 상수들 작성
    
    ```jsx
    // useFetch.js
    
        const [isFetching, setIsFetching] = useState();
        const [error, setError] = useState();
        const [fetchedData, setFetchedData] = useState();
    ```
    
    - setUserPlaces 함수를 setFetchedData 함수로 수정
    
    ```jsx
                    setFetchedData(places);
    ```
    
    - 파라미터(매개변수) fetchFn을 넣어주고
    
    ```jsx
    function useFetch(fetchFn) {
    ```
    
    - 이 커스텀 훅이 모든 fetchFn에서 사용되도록 fetchUserPlaces 함수를 fetchFn 함수로 수정
    
    ```jsx
                    const places = await fetchFn();
    ```
    
    - places를 data로 수정
    
    ```jsx
                try {
                    const data = await fetchFn();
                    setFetchedData(data);
                } catch (error) {
                    setError({ message: error.message || 'Failed to fetch data.' });
                }
    ```
    
    - fetchPlaces를 fetchData로 수정
    
    ```jsx
            async function fetchData() {
            
    // 중략
    
            }
    
            fetchData();
    ```
    
    - fetchFn을 의존성 배열에 추가 (빈 배열일 때 노란 줄 떴었음)
    
    ```jsx
        useEffect(() => {
        
    // 중략
    
        }, [fetchFn]);
    ```
    
    - effect 함수 밖에서 useState 상수들을 그룹화해 반환 (여기서는 객체로 그룹화)
    - useFetch 함수를 export
    
    ```jsx
    export function useFetch(fetchFn) {
        const [isFetching, setIsFetching] = useState();
        const [error, setError] = useState();
        const [fetchedData, setFetchedData] = useState();
    
    // 중략
    
        return {
            isFetching,
            error,
            fetchedData,
        };
    }
    ```
    
    - App.jsx에서 useFetch를 import하고 스냅샷 상태를 얻기 위해 useFetch 안에 fetchUserPlaces를 넣어 useFetch 훅에서 반환하는 객체 형태 그대로 받아준다
    
    ```jsx
    // App.jsx
    
    import { useFetch } from './hooks/useFetch.js';
    
    // 중략
    
        const { isFetching, error, fetchedData } = useFetch(fetchUserPlaces);
    ```
    
    - App.jsx에서 상태가 업데이트되는 함수들을 주석 처리한 후 그 함수들을 호출한 prop들도 주석 처리
    
    ```jsx
        // async function handleSelectPlace(selectedPlace) {
        //     // await updateUserPlaces([selectedPlace, ...userPlaces]);
    
        //     setUserPlaces((prevPickedPlaces) => {
        //         if (!prevPickedPlaces) {
        //             prevPickedPlaces = [];
        //         }
        //         if (prevPickedPlaces.some((place) => place.id === selectedPlace.id)) {
        //             return prevPickedPlaces;
        //         }
        //         return [selectedPlace, ...prevPickedPlaces];
        //     });
    
        //     try {
        //         await updateUserPlaces([selectedPlace, ...userPlaces]);
        //     } catch (error) {
        //         setUserPlaces(userPlaces);
        //         setErrorUpdatingPlaces({
        //             message: error.message || 'Failed to update places.',
        //         });
        //     }
        // }
    
        // const handleRemovePlace = useCallback(
        //     async function handleRemovePlace() {
        //         setUserPlaces((prevPickedPlaces) =>
        //             prevPickedPlaces.filter((place) => place.id !== selectedPlace.current.id)
        //         );
    
        //         try {
        //             await updateUserPlaces(userPlaces.filter((place) => place.id !== selectedPlace.current.id));
        //         } catch (error) {
        //             setUserPlaces(userPlaces);
        //             setErrorUpdatingPlaces({
        //                 message: error.message || 'Failed to delete place.',
        //             });
        //         }
    
        //         setModalIsOpen(false);
        //     },
        //     [userPlaces]
        // );
        
    // 중략
        
                    <Modal open={modalIsOpen} onClose={handleStopRemovePlace}>
                    <DeleteConfirmation
                        onCancel={handleStopRemovePlace}
                        // onConfirm={handleRemovePlace}
                    />
                    </Modal>
    		            
    // 중략
    
                    <AvailablePlaces
                    // onSelectPlace={handleSelectPlace}
                    />
    ```
    
    - Places 컴포넌트의 places라는 prop의 값을 userPlaces에서 fetchedData로 수정
        
        ```jsx
                        {!error && (
                            <Places
                                title="I'd like to visit ..."
                                fallbackText="Select the places you would like to visit below."
                                isLoading={isFetching}
                                loadingText="Fetching your places..."
                                places={fetchedData}
                                onSelectPlace={handleStartRemovePlace}
                            />
                        )}
        ```
        
        - userPlaces라는 별칭으로 쓰고 싶다면 선언했던 객체 상수의 속성 이름 옆에 콜론을 붙이고 별칭 작성
        
        ```jsx
            const { isFetching, error, fetchedData: userPlaces } = useFetch(fetchUserPlaces);
            
        // 중략
            
                        {!error && (
                            <Places
                                title="I'd like to visit ..."
                                fallbackText="Select the places you would like to visit below."
                                isLoading={isFetching}
                                loadingText="Fetching your places..."
                                places={userPlaces}
                                onSelectPlace={handleStartRemovePlace}
                            />
                        )}
        ```
        
    - 실행 시 오류가 뜸: undefined의 length에 접근했기 때문
        - useFetch 훅에서 빈 배열이 아닌 undefined로 초기화했기 때문
        - initialValue라는 prop을 받아와 fetchedData의 초기 값으로 설정
    
    ```jsx
    // useFetch.js
    
    export function useFetch(fetchFn, initialValue) {
        const [fetchedData, setFetchedData] = useState(initialValue);
    ```
    
    - App.jsx에서 initialValue에 들어갈 값을 빈 배열로 넣어줌
    
    ```jsx
    // App.jsx
    
        const { isFetching, error, fetchedData: userPlaces } = useFetch(fetchUserPlaces, []);
    ```
    

### 커스텀 Hooks(훅)에서 중첩 함수 노출시키기

- 커스텀 훅:
    - 커스텀 함수들도 노출시킬 수 있음
    - 사용할 때마다 새 복사본이 독립적으로 생성됨 (사용된 컴포넌트와만 상태 값이 연결됨 = 독립적인 스냅샷)
- 실습
    - useFetch.js에서 setFetchedData를 포인터로 반환하고 App.jsx에서도 작성 (별칭은 setUserPlaces)
    
    ```jsx
    // useFetch.js
    
        return {
            isFetching,
            error,
            setFetchedData,
            fetchedData,
        };
    ```
    
    ```jsx
    // App.jsx
    
        const { isFetching, error, setFetchedData: setUserPlaces, fetchedData: userPlaces } = useFetch(fetchUserPlaces, []);
    ```
    
    - 지난 번에 주석 처리했던 코드들을 전부 돌려놓음 (handleSelectPlace, handleRemovePlace)
    - handleRemovePlace의 의존성에 커스텀 훅과 연관된 setUserPlaces 추가 필요 (변화는 없음, 경고만 제거)
    
    ```jsx
        const handleRemovePlace = useCallback(
            async function handleRemovePlace() {
    
    // 중략
    
            },
            [userPlaces, setUserPlaces]
        );
    ```
    

### 다중 컴포넌트에서 커스텀 Hook(훅) 사용하기

- 커스텀 훅 사용 이유
    - 더 효율적인 컴포넌트 생성을 위함
    - 여러 컴포넌트가 비슷한 원리를 가질 경우 동일 훅을 공유하기 위함
- 실습
    - AvailablePlaces.jsx도 useFetch를 가져와 객체 형태로 호출 가능
        - 첫 번째 인자인 fetch함수는 fetchAvailablePlaces를 넣어줌
        - 두 번째 인자인 initialValue는 빈 배열을 넣어줌
        - 위에서 선언했던 상태 관리 useState 상수들은 모두 제거
    
    ```jsx
    // AvailablePlaces.jsx
    
    import { useFetch } from '../hooks/useFetch.js';
    
    export default function AvailablePlaces({ onSelectPlace }) {
        const { isFetching, error, setFetchedData: setAvailablePlaces, fetchedData: availablePlaces } = useFetch(fetchAvailablePlaces, []);
    ```
    
    - 거리 계산 로직을 따로 주석 처리해서 빼두고 useEffect 코드는 제거
    
    ```jsx
    import Places from './Places.jsx';
    import Error from './Error.jsx';
    // import { sortPlacesByDistance } from '../loc.js';
    import { fetchAvailablePlaces } from '../http.js';
    
    import { useFetch } from '../hooks/useFetch.js';
    
    // navigator.geolocation.getCurrentPosition((position) => {
    //   const sortedPlaces = sortPlacesByDistance(
    //       places,
    //       position.coords.latitude,
    //       position.coords.longitude
    //   );
    //   setAvailablePlaces(sortedPlaces);
    //   setIsFetching(false);
    // });
    
    export default function AvailablePlaces({ onSelectPlace }) {
        const {
            isFetching,
            error,
            setFetchedData: setAvailablePlaces,
            fetchedData: availablePlaces,
        } = useFetch(fetchAvailablePlaces, []);
    
        if (error) {
            return <Error title="An error occurred!" message={error.message} />;
        }
    
        return (
            <Places
                title="Available Places"
                places={availablePlaces}
                isLoading={isFetching}
                loadingText="Fetching place data..."
                fallbackText="No places available."
                onSelectPlace={onSelectPlace}
            />
        );
    }
    
    ```
    
    - 실행 시 장소 선택 배열에 사진을 추가해도 전체 선택 가능 목록에는 변화 없음 (서로 영향이 없는 독립 값들이기 때문)

### 유동성 있는 커스텀 Hooks(훅) 생성하기

- 따로 빼둔 거리 계산 로직을 활용하기 위해 새 함수를 추가하자
- 실습
    - AvailablePlaces.jsx에서 fetchSortedPlaces 함수 작성
    - fetchAvailablePlaces 함수를 비동기로 호출해 places 상수로 선언
    - 거리 계산 로직 주석 풀고 그 함수 안으로 옮기기
    
    ```jsx
    // AvailablePlaces.jsx
    
    async function fetchSortedPlaces() {
        const places = await fetchAvailablePlaces();
    
        navigator.geolocation.getCurrentPosition((position) => {
            const sortedPlaces = sortPlacesByDistance(places, position.coords.latitude, position.coords.longitude);
            setAvailablePlaces(sortedPlaces);
            setIsFetching(false);
        });
    }
    ```
    
    - 상태 업데이트 함수 제거 후 새 promise 안에 거리 계산 로직을 옮기고 반환
        - Promise 함수: 브라우저에 이미 있음. 함수를 인수로 인식. 매개변수(resolve와 reject)로 브라우저를 통해 전달
    
    ```jsx
    async function fetchSortedPlaces() {
        const places = await fetchAvailablePlaces();
    
        return new Promise((resolve, reject) => {
            navigator.geolocation.getCurrentPosition((position) => {
                const sortedPlaces = sortPlacesByDistance(places, position.coords.latitude, position.coords.longitude);
            });
        });
    }
    ```
    
    - resolve 안에 sortedPlaces를 넣어 실행
        - sortedPlaces 값을 반환하게 됨
        - reject는 오류를 작성할 때만 필요해서 제거
    
    ```jsx
        return new Promise((resolve) => {
            navigator.geolocation.getCurrentPosition((position) => {
                const sortedPlaces = sortPlacesByDistance(places, position.coords.latitude, position.coords.longitude);
                resolve(sortedPlaces);
            });
        });
    ```
    
    - useFetch에서 불러올 함수를 fetchAvailablePlaces에서 fetchSortedPlaces로 수정
    
    ```jsx
        const { isFetching, error, fetchedData: availablePlaces } = useFetch(fetchSortedPlaces, []);
    ```
    
    - 실행 시 거리에 따라 정렬된 장소들을 정상적으로 불러옴

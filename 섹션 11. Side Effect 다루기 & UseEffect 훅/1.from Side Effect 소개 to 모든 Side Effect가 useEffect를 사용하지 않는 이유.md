### Side Effect 소개

- 앱이 작동하는데 필요하지만 렌더링에 직접 영향을 주지 않음
- 실행은 필수지만 직접적이고 instant 영향 x
- ex : 장소를 거리 기준으로 정렬하기
    - 사용자의 위치를 구해야 함
    - 브라우저에서 내장 메소드가 있음

```jsx
// 현재 위도, 경도 구하기
// navigator .. 는 callback 함수이므로 처리해줌
navigator.geolocation.getCurrentPosition((pos) => {
  const sortedPlaces = sortPlacesByDistance(AVAILABLE_PLACES, 
    pos.coords.latitude, 
    pos.coords.longitude);
});
```

→ 이 함수 전체가 side effect임

- 위치가 필요는 하지만 컴포넌트의 주된 목적(jsx 요소 반환)과 관계가 없기 때문

### Side Effect의 무한 루프

- sortedPlace를 화면에 보여주고 싶음
    - Places 컴포넌트 props에 직접 전달해주고 싶음
    - getLocation 함수 실행에 시간이 걸림
    - availablePlaces를 state로 관리한 뒤 getLocation 실행 뒤 set해줄 예정

```jsx
const [availablePlaces, setAvailablePlaces] = useState();
  
navigator.geolocation.getCurrentPosition((pos) => {
  const sortedPlaces = sortPlacesByDistance(AVAILABLE_PLACES,
    pos.coords.latitude,
    pos.coords.longitude);
  setAvailablePlaces(sortedPlaces);
});
```

→ 무한루프터짐

- 상태 업데이트 함수를 호출하면 리액트의 컴포넌트 함수가 재실행됨 → getLocation 함수 다시 호출

### useEffect를 사용하는 Side Effect

- useEffect 훅으로 이슈를 해결할 수 있음
- sideEffect 함수와 결과를 param으로 받음

```
useEffect(() => {
navigator.geolocation.getCurrentPosition((pos) => {
	const sortedPlaces = sortPlacesByDistance(AVAILABLE_PLACES,
  pos.coords.latitude,
  pos.coords.longitude);
	setAvailablePlaces(sortedPlaces);
}), []});
```

- 모든 컴포넌트 함수가 실행된 후에 useEffect 훅이 실행됨 (JSX 코드 반환 후)
- dependency 배열을 정의 → 의존성의 값이 변화되면 Effect 함수를 재실행함
    - 현재는 비어있으므로 재실행 X
- useEffect가 실행되는 동안 fallback 함수를 설정할 수 있음

```jsx
<Places
  title="Available Places"
  places={availablePlaces}
  fallbackText="Sorting by distance"
  onSelectPlace={handleSelectPlace}
/>
```

### 모든 Side Effect가 useEffect를 사용하지 않는 이유

- 굳이 항상 useEffect 훅처리 안해도 됨
- 전체 selectedPlace를 localStorage에 저장하고 싶음

```jsx
function handleSelectPlace(id) {
  setPickedPlaces((prevPickedPlaces) => {
    if (prevPickedPlaces.some((place) => place.id === id)) {
      return prevPickedPlaces;
    }
    const place = AVAILABLE_PLACES.find((place) => place.id === id);
    return [place, ...prevPickedPlaces];
  });
	// fallback 처리
  const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
  if (storedIds.indexOf(id) === -1) { // 배열에 없으면
    localStorage.setItem('selectedPlaces', JSON.stringify(id, ...storedIds));
  }
}
```

- 함수 내에서 useEffect 훅을 사용할 수 없음
    - 가장 상위 root에서 직접 사용만 가능함
    - 함수는 사진 클릭 시에만 호출되기 때문에 useEffect 훅을 쓸 필요도 없음

- Control
    - if & else if & for문
    - 쉬워서 생략

```jsx
const password = prompt("Your Password") // 브라우저 팝업에서 변수 입력받음
```

- DOM
    - Pure JS로 DOM 조작
        - addEventListner
        - document.querySelector 등
    - React나 Vue는 Virtual DOM을 사용해 실제 DOM과 동기화 하는 방식을 사용
        
        → 리액트에서 직접 JS로 DOM에 접근하면 동기화가 깨질 위험이 있음
        
        +성능 저하 우려
        
- 함수를 value로 사용하기
    - 함수를 다른 함수에서 value로 사용 가능

```jsx
function handleTimeout() {
	console.log("timeout")
}

function handle2() => {console.log("timeout");}

setTimeout(handleTimeout, 2000); // handleTimeout function을 setTimeout의 value로 전달
// handleTimeout 함수를 2000ms 대기한 후 실행할 것

// 같은 동작 코드
setTimeout(() => {
	console.log("timeout")
}, 2000);
```

- 함수 내부에서 함수 생성
    - 함수 내부에서 정의한 함수는 해당 함수의 내부에서 다시 실행할 수 있음

```jsx
function init() {
	function greet() {
		console.log("hi");
	}
	greet(); // 정상 작동
}

greet(); // 에러
```

- 참조형과 기본
    - 기본형 객체(숫자, 문자열, boolean)는 변경할 수 없음
    - 참조형 객체(배열, 함수 등 객체)은 값을 저장하지 않고 값이 저장된 메모리 주소를 참조
        - 실제 배열은 컴퓨터 메모리에 저장됨
        - 배열에 대한 조작은 JS가 직접 처리

```jsx
// 기본형
let userMessage = "hi"; // let -> 수정 가능한 변수 선언
userMessage = "ㅎㅇ"; // 변수가 overwrite -> 새로운 메모리에 저장된 "ㅎㅇ"가 새로운 userMessasge가 됨
userMessage = userMessage..concat("111"); // 마찬가지로 overwrite

// 참조형(배열)
const member = ["수진", "세원]; // const -> 수정할 수 없는 변수 선언
member.push("용운"); // 참조형 값의 경우 조작 시 
console.log(member); // 정상 수정
member = []; // 에러
```

- let & const
    - let
        - 재할당 가능한 변수 선언
        - 변수 호이스팅 가능
    - const
        - 변하지 않는 상수형 변수 선언
        - 변수 호이스팅 불가
    - 둘 다 block scope 변수이므로 선언한 블록 밖에서 사용할 수 없음
    
    ```jsx
    let message = "Hi";
    let message = "Hello"; // 가능
    
    const msg = "Hi";
    const msg = "Hello"; // 에러
    
    let function1() {
    	let a = 5; // 함수 스코프 안에서 사용 가능
    	return (a++);
    }
    
    console.log(a) // 지역 밖에서 변수 호출 에러
    ```
    
- Array Functions
    - push() : 배열에 푸쉬 푸
    - Array.length()
        - 배열의 원래 길이보다 길이를 더 길게 설정할 수 있음
        - 이 경우 초과하는 길이 값만큼 “empty” 값이 설정됨
            - empty ≠ undefined, null
    
    ```jsx
    let arr1 = [1,2,3,4,5];
    arr1.length = 10;
    console.log(arr1) // 1,2,3,4,5, empty*5
    ```
    
    - Arrays.forEach()
        - 자바의d for item 문법과 같음
        - 배열의 모든 원소를 조회하며 작동
    
    ```jsx
    const colors = ["red", "yellow", "blue"];
    colors[5] = "purple";
    colors.forEach((item, index) => {
      console.log(`${index}: ${item}`);
    });
    // 0: red
    // 1: yellow
    // 2: blue
    // 5: purple
    ```
    
    - map()
        - Arrays.forEach와 유사하지만 함수적으로 다른 배열로 매핑
    
    ```jsx
    const array1 = [1, 4, 9, 16];
    
    // Pass a function to map
    const map1 = array1.map((x) => x * 2);
    
    console.log(map1);
    // Expected output: Array [2, 8, 18, 32]
    ```
    
    - find(), findIndex()
        - 배열 내에서 조건에 맞는 가장 첫 번째 항목을 반환함
        - 못찾으면 undefined
        - find → 값, findIndex → 해당 인덱스
    
    ```jsx
    const array1 = [5, 12, 8, 130, 44];
    
    const found = array1.find((element) => element > 10);
    
    console.log(found);
    //12
    
    const found2 = array1.find((e) => e < 5);
    
    console.log(found2); // undefined
    ```
    
    - filter()
        - 배열 내에서 조건에 맞는 원소의 부분 배열을 반환
    
    ```jsx
    const words = ['spray', 'elite', 'exuberant', 'destruction', 'present'];
    
    const result = words.filter((word) => word.length > 6);
    
    console.log(result);
    // Expected output: Array ["exuberant", "destruction", "present"]
    ```
    
    - reduce()
        - 배열에서 원소를 값으로 지정해서 제거
    - concat()
        - 배열들 합친 새로운 배열 생성
        - 기존 배열은 제거되지 않음
        - 여러 개의 배열 병합 가능
    
    ```jsx
    const arr1 = [1,2,3];
    arr1.reduce(2); // [1,3]
    
    const arr2 = [4,5];
    const arr3 = [6,7];
    const arr3 = arr1.concat(arr2, arr3);
    console.log(arr3); // [1,3,4,5,6,7]
    
    ```
    
    - slice()
        - 시작 인덱스, 끝 인덱스, 혹은 둘 모두를 지정한 배열의 일부를 반환
            - 입력값이 하나일 때 음수인 경우 파이썬과 유사하게 length - value 인덱스로 계산
            - 입력값이 범위를 벗어나면 배열 마지막 인덱스로 인식
        - 본 배열을 shallow copy 후 복사한 값 사용
    
    ```jsx
    const animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];
    
    console.log(animals.slice(2));
    // Expected output: Array ["camel", "duck", "elephant"]
    
    console.log(animals.slice(2, 4));
    // Expected output: Array ["camel", "duck"]
    
    console.log(animals.slice(1, 5));
    // Expected output: Array ["bison", "camel", "duck", "elephant"]
    
    console.log(animals.slice(-2));
    // Expected output: Array ["duck", "elephant"]
    
    console.log(animals.slice(2, -1));
    // Expected output: Array ["camel", "duck"]
    
    console.log(animals.slice());
    // Expected output: Array ["ant", "bison", "camel", "duck", "elephant"]
    ```
    
    - splice()
        - 시작 인덱스, 삭제할 배열 길이, 추가할 값을 지정 가능
            - 삭제할 배열 길이를 지정하지 않으면 start 인덱스 이후 모든 원소 삭제
            - 삭제할 배열이 0이면 원소 추가 작업만 가능
    
    ```jsx
    splice(start) // 
    splice(start, deleteCount)
    splice(start, deleteCount, item1)
    splice(start, deleteCount, item1, item2)
    splice(start, deleteCount, item1, item2, /* …, */ itemN)
    
    const months = ['Jan', 'March', 'April', 'June'];
    months.splice(1, 0, 'Feb');
    // Inserts at index 1
    console.log(months);
    // Expected output: Array ["Jan", "Feb", "March", "April", "June"]
    
    months.splice(4, 1, 'May');
    // Replaces 1 element at index 4
    console.log(months);
    // Expected output: Array ["Jan", "Feb", "March", "April", "May"]
    
    months.splice(
    ```

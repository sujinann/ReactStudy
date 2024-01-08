# 화살표 함수 ~ 스프레드 연산자

<br/>

## 화살표 함수

```jsx
function 함수명(변수) {
	console.log(변수);
}
```

function을 붙여야 하는 위 함수와 비교하여
익명함수를 사용할 때 유용하게 사용함

```jsx
(변수) => {
	console.log(변수);
}
```

<br/>

## 화살표 함수 구문에 대한 상세내용

1) 매개변수 목록 괄호 생략하기

    - 화살표 함수가 정확히 하나의 매개변수만 사용하는 경우, 묶는 괄호를 생략할 수 있습니다.

        - (userName) => { ... }
        가 아니라

        - userName => { ... }
        라고 쓸 수 있습니다.
      

    - 함수에 매개변수가 없는 경우에는, 괄호를 생략해서는 안 됩니다.

        - () => { ... } 라고 써야 옳습니다.
      

    - 함수가 둘 이상의 매개변수를 받는 경우에도 괄호를 생략해서는 안 됩니다.

        - userName, userAge => { ... } 라고 쓰면 안 됩니다.

        - (userName, userAge) => { ... } 라고 써야 합니다.
     

2) 함수 본문 중괄호 생략하기

	- 화살표 함수에 반환문 외에 다른 로직이 없는 경우, return키워드와 중괄호를 생략할 수 있습니다.

		- number => {   return number * 3;}
		라고 쓰는 게 아니라

		- number => number * 3;
		라고 쓸 수 있습니다.

	- 아래와 같이 오류가 생깁니다.

		- number => return number * 3; // 이 경우 retrun 키워드는 생략되어야 하므로, 오류가 생깁니다.
		- number => if (number === 2) { return 5 }; // 이 경우 if 문은 반환될 수 없으므로 오류가 생깁니다. 


3) 특수한 경우: 객체만 반환하는 경우

	- 2)에서 설명한 짧은 대안으로 자바스크립트 객체를 반환하려고 하면, 다음과 같이 유효하지 않은 코드가 나올 수 있습니다.

		- number => ({ age: number }); // 객체를 반환하려고 합니다.
		- 자바스크립트는 중괄호를 JS 객체를 생성하는 코드가 아닌 함수 본문 래퍼로 취급하기 때문에 이 코드는 유효하지 않습니다.

	- 객체를 생성하고 반환해야 한다고 자바스크립트에 “말하려면” 코드를 다음과 같이 수정해야 합니다:

		- number => ({ age: number }); // 추가 괄호를 써서 객체를 감싸줍니다.
		- 객체와 중괄호를 추가 괄호로 감싸면, 자바스크립트는 중괄호가 함수 본문을 정의하는 것이 아니라 객체를 생성하기 위한 것임을 이해합니다. 따라서 객체가 반환됩니다.

<br/>

## 객체와 클래스

- object를 key - value 형태로 그룹화하여 나타낼 수 있음

```jsx
const user = {
	name: "Max",
	age: 34,
	greet() {
		console.log("Hello");
		console.log("this.age");
	}
}

console.log(user.name);
user.greet();
```

- 객체안에 메소드도 정의할 수 있는데 function은 필요 없음
- 오브젝트 안의 필드에 접근할 때에는 this 키워드를 사용함

```jsx
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log("Hi!");
  }
}

const user1 = new User("Manuel", 35)
consle.log(user1);
user1.greet();
```

- 클래스 키워드로 블루프린트를 만들고 이 블루프린트를 이용하여 객체를 생성할 수도 있음

- 클래스명은 첫글자가 대문자여야 함

<br/>

## 배열

- 배열의 특성
	- 대괄호 사용
	- 객체의 키 - 값 그룹화와 다르게 value만 쉼표로 구분하여 나열함
	- 배열안에 또 배열들을 넣을 수 있음
	- 인덱스 번호로 배열안의 순서에 맞는 값을 꺼내올 수 있음
	- 인덱스 번호는 0부터 시작

```jsx
const hobbies = ["Sports", "Cooking", "Reading"];
console.log(hobbies[0]);
```

hobbies.~~~
배열에 . 을 찍고 메소드를 사용할 수 있음
아래는 map 메소드 활용 예시

```jsx
const editedHobbies = hobbies.map((item) => ({text: item}));
console.log(editedHobbies);//키가 텍스트인 오브젝트 리스트 출력
```

<br/>

- 코딩 연습2 : 배열
![연습2배열](https://github.com/sujinann/ReactStudy/assets/139312979/7cb6480f-cc19-49df-bad2-555da0e73e42)

<br/>

## 디스트럭처링

```jsx
const userNameData = ["Max", "Schwarzmuller"];

const firstName = userNameData[0];
const lastName = userNameData[1];
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&darr;

```jsx
const [firstName, lastName] = ["Max", "Schwarzmuller"];

console.log(firstName);
console.log(lastName);
```

- 객체는 중괄호 사용

```jsx
const user = {
  name: "Max",
  age : 34
};
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&darr;

```jsx
const {name: userName, age} = {
  name: "Max",
  age : 34
};

console.log(userName);
console.log(age);
```

<br/>

## 함수 매개변수 목록에서 디스트럭처링

디스트럭처링 구문은 함수 매개변수 목록에서도 사용할 수 있습니다.

예를 들어, 함수가 객체를 포함하는 매개변수를 수락하는 경우, 객체 프로퍼티를 꺼내어 로컬 범위 변수로 사용할 수 있도록 함수를 디스트럭처링할 수 있습니다.

```jsx
function storeOrder(order) {
  localStorage.setItem('id', order.id);
  localStorage.setItem('currency', order.currency);
}
```
storeOrder 함수 본문 내부의 "점 표기법"을 통해 order 프로퍼티에 접근하지 않고, 다음과 같이 디스트럭처링을 사용할 수 있습니다:

```jsx
function storeOrder({id, currency}) { // 디스트럭처링
  localStorage.setItem('id', id);
  localStorage.setItem('currency', currency);
}
```
디스트럭처링 구문은 이전 강의에서 배운 것과 같습니다. 상수나 변수를 수동으로 생성하지 않을 뿐입니다.

대신, 들어오는 객체(즉, storeOrder 에 인수로 전달된 객체)에서 id와 currency 를 꺼내어 사용합니다.

이 예제에서 storeOrder 는 여전히 하나의 매개변수만 받는다는 점이 매우 중요합니다! 매개변수는 두 개가 아니라, 하나의 매개변수, 즉 내부적으로 디스트럭처링된 객체만 받습니다.

함수는 여전히 다음과 같이 호출됩니다

```jsx
storeOrder({id: 5, currency: 'USD', amount: 15.99}); // 1개의 매개변수 / 값!
```

<br/>

## 스프레드 연산자

...을 이용하여 스프레드  
객체, 배열 전부에 적용 가능  
아래는 활용 예시  

```jsx
const hobbies = ["Sports", "Cooking"];
const user = {
  name: "Max",
  age: 34
};

const newHobbies = ["Reading"];

const mergeHobbies = [...hobbies, ...newhobbies];
console.log(mergeHobbies);

const extendedUser = {
  isAdmin: true,
  ...user
};

console.log(extendedUser);
```





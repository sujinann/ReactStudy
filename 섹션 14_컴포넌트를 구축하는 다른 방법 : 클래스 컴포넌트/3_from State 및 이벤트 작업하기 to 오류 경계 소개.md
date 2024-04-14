## State 및 이벤트 작업하기

- 상태 초기화, 상태 정의
- 상태 관리 시 무조건 하나의 객체 형태 안에 모두 묶어 관리
- 상태 변경 시 꼭 setState 사용
    - 이전 나머지 상태와 자동으로 결합(함수형과의 차이)
    - 여기도 객체 형태 필수
- this.toggleUsersHandler.bind(this) 함수 실행 차이점
- super(); 추가 필요

```jsx
import { Component } from 'react';

import User from './User';
import classes from './Users.module.css';

const DUMMY_USERS = [
  { id: 'u1', name: 'Max' },
  { id: 'u2', name: 'Manuel' },
  { id: 'u3', name: 'Julie' },
];

class Users extends Component {
  constructor() {
    super();
    this.state = {
      showUsers: true,
      more: 'Test',
    };
  }

  toggleUsersHandler() {
    // this.state.showUsers = false; // NOT!
    this.setState((curState) => {
      return { showUsers: !curState.showUsers };
    });
  }

  render() {
    const usersList = (
      <ul>
        {DUMMY_USERS.map((user) => (
          <User key={user.id} name={user.name} />
        ))}
      </ul>
    );

    return (
      <div className={classes.users}>
        <button onClick={this.toggleUsersHandler.bind(this)}>
          {this.state.showUsers ? 'Hide' : 'Show'} Users
        </button>
        {this.state.showUsers && usersList}
      </div>
    );
  }
}

// const Users = () => {
//   const [showUsers, setShowUsers] = useState(true);

//   const toggleUsersHandler = () => {
//     setShowUsers((curState) => !curState);
//   };

//   const usersList = (
//     <ul>
//       {DUMMY_USERS.map((user) => (
//         <User key={user.id} name={user.name} />
//       ))}
//     </ul>
//   );

//   return (
//     <div className={classes.users}>
//       <button onClick={toggleUsersHandler}>
//         {showUsers ? 'Hide' : 'Show'} Users
//       </button>
//       {showUsers && usersList}
//     </div>
//   );
// };

export default Users;
```

<br/>

## 클래스 컴포넌트 수명 주기

클래스 컴포넌트에서는 useEffect 같은 훅 사용 불가

![class_component.PNG](https://prod-files-secure.s3.us-west-2.amazonaws.com/ca254afe-fecd-4396-92df-1ab2cb1215ad/3476e237-c9bc-4abc-8f92-268754c94fca/class_component.png)

<br/>

## 사용 중인 수명 주기 방법

- useEffect 를 componentDidUpdate 로 대체할 때 의존성을 조건문으로 추가해 줘야 무한 루프를 방지함
- 그냥 코드가 조금 더 길 뿐 useEffect 생명 주기 모델과 같은 기능을 수행함

```jsx
import { Fragment, useState, useEffect, Component } from 'react';

import Users from './Users';
import classes from './UserFinder.module.css';

const DUMMY_USERS = [
  { id: 'u1', name: 'Max' },
  { id: 'u2', name: 'Manuel' },
  { id: 'u3', name: 'Julie' },
];

class UserFinder extends Component {
  constructor() {
    super();
    this.state = {
      filteredUsers: [],
      searchTerm: '',
    };
  }

  componentDidMount() {
    // Send http request...
    this.setState({ filteredUsers: DUMMY_USERS });
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.searchTerm !== this.state.searchTerm) {
      this.setState({
        filteredUsers: DUMMY_USERS.filter((user) =>
          user.name.includes(this.state.searchTerm)
        ),
      });
    }
  }

  searchChangeHandler(event) {
    this.setState({ searchTerm: event.target.value });
  }

  render() {
    return (
      <Fragment>
        <div className={classes.finder}>
          <input type='search' onChange={this.searchChangeHandler.bind(this)} />
        </div>
        <Users users={this.state.filteredUsers} />
      </Fragment>
    );
  }
}

// const UserFinder = () => {
//   const [filteredUsers, setFilteredUsers] = useState(DUMMY_USERS);
//   const [searchTerm, setSearchTerm] = useState('');

//   useEffect(() => {
//     setFilteredUsers(
//       DUMMY_USERS.filter((user) => user.name.includes(searchTerm))
//     );
//   }, [searchTerm]);

//   const searchChangeHandler = (event) => {
//     setSearchTerm(event.target.value);
//   };

//   return (
//     <Fragment>
//       <div className={classes.finder}>
//         <input type='search' onChange={searchChangeHandler} />
//       </div>
//       <Users users={filteredUsers} />
//     </Fragment>
//   );
// };

export default UserFinder;
```

<br/>

## 클래스 컴포넌트 및 컨텍스트

- 보통 하나의 컨텍스트만 사용하기에 문제 될 일은 잘 없지만 만약 두 개 이상의 컨텍스트를 사용해야 할 경우에는 다른 방법을 찾아봐야 함
    - class component 방식으로는 컨텍스트를 하나만 연결할 수 있기 때문

```jsx
import { Fragment, useState, useEffect, Component } from 'react';

import Users from './Users';
import classes from './UserFinder.module.css';
import UsersContext from '../store/users-context';

class UserFinder extends Component {
  static contextType = UsersContext;

  constructor() {
    super();
    this.state = {
      filteredUsers: [],
      searchTerm: '',
    };
  }

  componentDidMount() {
    // Send http request...
    this.setState({ filteredUsers: this.context.users });
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.searchTerm !== this.state.searchTerm) {
      this.setState({
        filteredUsers: this.context.users.filter((user) =>
          user.name.includes(this.state.searchTerm)
        ),
      });
    }
  }

  searchChangeHandler(event) {
    this.setState({ searchTerm: event.target.value });
  }

  render() {
    return (
      <Fragment>
        <div className={classes.finder}>
          <input type='search' onChange={this.searchChangeHandler.bind(this)} />
        </div>
        <Users users={this.state.filteredUsers} />
      </Fragment>
    );
  }
}

// const UserFinder = () => {
//   const [filteredUsers, setFilteredUsers] = useState(DUMMY_USERS);
//   const [searchTerm, setSearchTerm] = useState('');

//   useEffect(() => {
//     setFilteredUsers(
//       DUMMY_USERS.filter((user) => user.name.includes(searchTerm))
//     );
//   }, [searchTerm]);

//   const searchChangeHandler = (event) => {
//     setSearchTerm(event.target.value);
//   };

//   return (
//     <Fragment>
//       <div className={classes.finder}>
//         <input type='search' onChange={searchChangeHandler} />
//       </div>
//       <Users users={filteredUsers} />
//     </Fragment>
//   );
// };

export default UserFinder;
```

<br/>

## 클래스 컴포넌트 대 함수형 컴포넌트

- 오류 경계 경우를 제외하고 함수형 컴포넌트에 미세한 장점들이 더 많음
- 하지만 둘 다 같은 기능을 수행할 수 있기 때문에 취향에 따라 사용해도 상관 없음

![class_vs_function.PNG](https://prod-files-secure.s3.us-west-2.amazonaws.com/ca254afe-fecd-4396-92df-1ab2cb1215ad/89d53a11-27ae-41f0-b921-62450b915f77/class_vs_function.png)

<br/>

## 오류 경계 소개

서버 다운과 같은 예상치 못한 오류에 대응하는 수단으로 활용할 수 있음

- 함수형 컴포넌트에서 사용하는 try & catch 와 동일한 역할을 수행
    - 에러가 발생하더라도 앱 전체를 중단시키지 않고 상황에 대한 미리 상정해 둔 대처를 수행함
- 클래스 컴포넌트여야 하고 생명 주기 메소드를 가지고 있는 컴포넌트여야 함

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor() {
    super();
    this.state = { hasError: false };
  }

  componentDidCatch(error) {
    console.log(error);
    this.setState({ hasError: true });
  }

  render() {
    if (this.state.hasError) {
      return <p>Something went wrong!</p>;
    }
    return this.props.children;
  }
}

export default ErrorBoundary;
```

```jsx
import { Fragment, useState, useEffect, Component } from 'react';

import ErrorBoundary from './ErrorBoundary';

class UserFinder extends Component {
  static contextType = UsersContext;

  ...

  render() {
    return (
      <Fragment>
        <div className={classes.finder}>
          <input type='search' onChange={this.searchChangeHandler.bind(this)} />
        </div>
        // 이런식으로 감싸서 사용
        <ErrorBoundary>
          <Users users={this.state.filteredUsers} />
        </ErrorBoundary>
      </Fragment>
    );
  }
}
```

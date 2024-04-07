## 첫 컴포넌트 및 상태 몇 가지

```jsx
// Header.jsx
import logoImg from '../assets/quiz-logo.png';

export default function Header() {
  return (
    <header>
      <img src={logoImg} alt="Quiz logo" />
      <h1>ReactQuiz</h1>
    </header>
  );
}
```

- 이미지와 로고 출력

```jsx
// Quiz.jsx
import { useState } from 'react';

export default function Quiz() {
  const [activeQuestionIndex, setActiveQuestionIndex] = useState(0);
  const [userAnswers, setUserAnswers] = useState([]);

  return <p>Currently active Question</p>;
}
```

- 일단 현재 퀴즈 인덱스와 유저의 답안 상태 관리가 목표임

</br>

## 값 파생시키기, 질문 출력 및 답변 등록

```jsx
export default [
  {
    id: 'q1',
    text: 'Which of the following definitions best describes React.js?',
    answers: [
      'A library to build user interfaces with help of declarative code.',
      'A library for managing state in web applications.',
      'A framework to build user interfaces with help of imperative code.',
      'A library used for building mobile applications only.',
    ],
  },
  {
    id: 'q2',
    text: 'What purpose do React hooks serve?',
    answers: [
      'Enabling the use of state and other React features in functional components.',
      'Creating responsive layouts in React applications.',
      'Handling errors within the application.',
      'Part of the Redux library for managing global state.',
    ],
  },
  
  ...]
```

- 제공되는 퀴즈 정보.
- id와 퀴즈, 답안지로 구성된 배열.
- 데이터 처리가 편하도록 답안지의 첫 번째 답이 항상 정답임. 따라서 이용자에게 보여줄 때는 답안지를 무작위로 섞어서 보여줄 필요가 있음.

userAnswers 에서 activeQuestionIndex 값을 파생시켜 가져올 수 있기 때문에 위의 로직은 최선이 아님. 로직을 개선할 것임

```jsx
// Quiz.jsx
import { useState } from 'react';

import QUESTIONS from '../questions.js';

export default function Quiz() {
  const [userAnswers, setUserAnswers] = useState([]);

  const activeQuestionIndex = userAnswers.length;

  function handleSelectAnswer(selectedAnswer) {
    setUserAnswers((prevUserAnswers) => {
      return [...prevUserAnswers, selectedAnswer];
    });
  }

  return (
    <div id="quiz">
      <div id="question">
        <h2>{QUESTIONS[activeQuestionIndex].text}</h2>
        <ul id="answers">
          {QUESTIONS[activeQuestionIndex].answers.map((answer) => (
            <li key={answer} className="answer">
              <button onClick={() => handleSelectAnswer(answer)}>
                {answer}
              </button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

개선 후 퀴즈 출력 및 답안지 선택 기능 추가.

- () => handleSelectAnswer(answer) 화살표 함수로 만들어야 바로 실행하지 않고 클릭 시에 실행.

</br>

## 답변 셔플 & 퀴즈 로직 추가

```jsx
// Quiz.jsx
import { useState } from 'react';

import QUESTIONS from '../questions.js';
import quizCompleteImg from '../assets/quiz-complete.png';

export default function Quiz() {
  const [userAnswers, setUserAnswers] = useState([]);

  const activeQuestionIndex = userAnswers.length;
  const quizIsComplete = activeQuestionIndex === QUESTIONS.length;

  function handleSelectAnswer(selectedAnswer) {
    setUserAnswers((prevUserAnswers) => {
      return [...prevUserAnswers, selectedAnswer];
    });
  }

  if (quizIsComplete) {
    return (
      <div id="summary">
        <img src={quizCompleteImg} alt="Trophy icon" />
        <h2>Quiz Completed!</h2>
      </div>
    );
  }

  const shuffledAnswers = [...QUESTIONS[activeQuestionIndex].answers];
  shuffledAnswers.sort(() => Math.random() - 0.5);

  return (
    <div id="quiz">
      <div id="question">
        <h2>{QUESTIONS[activeQuestionIndex].text}</h2>
        <ul id="answers">
          {shuffledAnswers.map((answer) => (
            <li key={answer} className="answer">
              <button onClick={() => handleSelectAnswer(answer)}>
                {answer}
              </button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

- 답안지 랜덤으로 섞는 기능 추가
- quizIsComplete 추가해서 퀴즈 끝인지 판단 로직 추가. 에러 발생 방지 위해 로직 순서 변경.

</br>

## 질문 타이머 추가

```jsx
// QuestionTimer.jsx
import { useState, useEffect } from 'react';

export default function QuestionTimer({ timeout, onTimeout }) {
  const [remainingTime, setRemainingTime] = useState(timeout);

  useEffect(() => {
    console.log('SETTING TIMEOUT');
    setTimeout(onTimeout, timeout);
  }, [timeout, onTimeout]);

  useEffect(() => {
    console.log('SETTING INTERVAL');
    setInterval(() => {
      setRemainingTime((prevRemainingTime) => prevRemainingTime - 100);
    }, 100);
  }, []);

  return <progress id="question-time" max={timeout} value={remainingTime} />;
}
```

- 무한 루프 방지를 위해 useEffect 사용
- 아직 로직 미완성

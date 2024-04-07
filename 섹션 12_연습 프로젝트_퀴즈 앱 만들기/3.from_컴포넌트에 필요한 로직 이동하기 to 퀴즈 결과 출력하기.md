### 컴포넌트에 필요한 로직 이동하기

- 분리를 열심히 했더니 전달할 Prop이 매우 많아짐
- answerState가 모든 컴포넌트에서 관리되고 있음
    - 가장 하위 컴포넌트에서만 관리하는 편이 나을듯! (State Lifting이 과도함)
- Questions에서 문제 데이터인 QUESTION을 직접 사용하면 key prop 하나만으로 여러 prop을 전달받지 않고 직접 사용할 수 있음(QuestionText, answer 등)

```jsx
// Question.jsx

import QuestionTimer from "./QuestionTimer";
import Answers from "./Answers";
import { useState } from "react";
import QUESTIONS from "../../questions";

export default function Question({
  key,
  onSelectAnswer,
  onSkipAnswer,
}) {
  const [answer, setAnswer] = useState({
    selectedAnswer: "",
    isCorrect: null, // boolean
  });

  function handleSelectAnswer(answer) {
    setAnswer({
      selectedAnswer: answer,
      isCorrect: null, // boolean
    });

    setTimeout(() => {
      setAnswer({
        selectedAnswer: answer,
        isCorrect: QUESTIONS[key].answers[0] === answer,
      });
    }, 1000);

    setTimeout(() => {
        onSelectAnswer(answer);
    }, 2000);
  }

  let answerState = '';

  if (answer.selectedAnswer) {
    answerState = answer.isCorrect? 'correct' : 'wrong';
  }

  return (
    <div id="question">
      <QuestionTimer timeout={10000} onTimeout={onSkipAnswer} />
      <h2>{QUESTIONS[key].text}</h2>
      <Answers
        answers={answer}
        selectedAnswer={answer.selectedAnswer}
        answerState={answerState}
        onSelect={onSelectAnswer}
      />
    </div>
  );
}

```

- Quiz에서도 불필요한 Prop을 제거할 수 있음

```jsx
<Question
      key={activeQuestionIndex}
      // questionText={QUESTIONS[activeQuestionIndex].text}
      // answers={QUESTIONS[activeQuestionIndex].answers}
      onSelectAnswer={handleSelectAnswer}
      // selectedAnswer={userAnswers[userAnswers.length - 1]}
      // answerState={answerState}
      onSkipAnswer={handleSkipAnswer}
/>
```

- answer를 Question에서 관리하기 떄문에 Quiz에서 answer 관련 로직을 제거할 수 있음

```jsx
import { useCallback, useState } from "react";

import QUESTIONS from "../../questions.js";
import quizCompleteImg from "../assets/quiz-complete.png";
import Question from "./Question.jsx";

export default function Quiz() {
  const [userAnswers, setUserAnswers] = useState([]);
//   const [answerState, setAnswerState] = useState("");

  const activeQuestionIndex = userAnswers.length;

  const quizIsComplete = activeQuestionIndex === QUESTIONS.length;

  const handleSelectAnswer = useCallback(
    function handleSelectAnswer(selectedAnswer) {
    //   setAnswerState("answered");
      setUserAnswers((prevUserAnswers) => {
        return [...prevUserAnswers, selectedAnswer];
      });

    //   setTimeout(() => {
    //     if (selectedAnswer === QUESTIONS[activeQuestionIndex].answers[0]) {
    //       setAnswerState("correct");
    //     } else setAnswerState("wrong");

    //     setTimeout(() => {
    //       setAnswerState("");
    //     }, 20000);
    //   }, 1000);
    },
    [activeQuestionIndex]
  );

  const handleSkipAnswer = useCallback(() => handleSelectAnswer(null), []);

  if (quizIsComplete) {
    return (
      <div id="summary">
        <img src={quizCompleteImg} />
        <h2>Quiz done</h2>
      </div>
    );
  }
  const shuffledAnswers = [...QUESTIONS[activeQuestionIndex].answers];
  shuffledAnswers.sort(() => Math.random() - 0.5);

  return (
    <div id="quiz">
      <Question
        key={activeQuestionIndex}
        // questionText={QUESTIONS[activeQuestionIndex].text}
        // answers={QUESTIONS[activeQuestionIndex].answers}
        onSelectAnswer={handleSelectAnswer}
        // selectedAnswer={userAnswers[userAnswers.length - 1]}
        // answerState={answerState}
        onSkipAnswer={handleSkipAnswer}
      />
    </div>
  );
}

```

- key를 prop으로 전달할 수 없다는 에러 발생
- Question에서 key prop의 이름을 index로 바꿔줌 + Quiz에서 index prop을 추가로 전달
- Answer에서 추가 작업 : 선택한 답변이 없을 때만 disabled 속성을 줄 것임

```jsx
return (
    <li key={answer} className="answer">
        <button onClick={() => onSelect(answer)} 
        className={cssClass}
        disabled={answerState !== ''}
        >
            {answer}
        </button>
    </li>
);
```

### 선택한 답변 기준 타이머 설정

- 답변을 조금 늦게 고르면 오답 여부를 표시하기 전에 만료됨
- + 동시에 여러 타이머가 실행됨

```jsx
setTimeout(() => {
      setAnswer({
        selectedAnswer: answer,
        isCorrect: QUESTIONS[index].answers[0] === answer,
      });
    }, 1000);

    setTimeout(() => {
        onSelectAnswer(answer);
    }, 2000);
```

- Question에서 동시에 두 개의 타이머를 실행하고 있기 때문
    - Timer가 완료되면 부모 컴포넌트로 정답 여부를 보냄
    - 정답 여부를 확인 전에 타이머가 만료되어서 부모 컴포넌트로 전달되어서 다음 문제로 교체되는 이슈
    - 이는 의도와 다름(선택하지 못하고 넘어갈 때만 null이 전달되어야 하지만 조금 늦게 선택해도 동일)
- 답변을 선택시 타이머를 업데이트하는 방식으로 해결 가능

```jsx
let timer = 10000;

if (answer.selectedAnswer) {
  timer = 1000;
}

if (answer.isCorrect !== null) {
  timer = 2000; // 다음 질문으로 넘어가는 시간
}
```

- QuestionTimer에 mode prop 전달 (스타일링용)
    - answered 시 다른 스타일 적용
    - key를 전달해서 timer 리셋 시마다 스타일 초기화

```jsx
// Question
	<QuestionTimer
	  timeout={timer}
	  onTimeout={answer.selectedAnswer === "" ? onSkipAnswer : null}
	  mode={answerState}
	  key={timer}
	/>
	
// QuestionTImer
return (
    <progress
      id="question-time"
      max={timeout}
      value={remainingTime}
      className={mode}
    />
  );
```

### 퀴즈 결과 출력하기

- 퀴즈 완료 시 정답과 오답 수를 출력하고 싶음
- Quiz에서 complete 시 return하는 부분을 가져옴

```jsx
// Quiz
if (quizIsComplete) {
  return <Summary />
}
// Summary
export default function Summary() {
    return (
        <div id="summary">
          <img src={quizCompleteImg} />
          <h2>Quiz done</h2>
          <div id="summary-stats">
            <p>
                <span className="number">10%</span>
                <span className="text">skipped</span>
            </p>
            <p>
                <span className="number">10%</span>
                <span className="text">correct</span>
            </p>
            <p>
                <span className="number">10%</span>
                <span className="text">incorrect</span>
            </p>
          </div>
          <ol>
            <li>
                <h3>2</h3>
                <p className="question">question text</p>
                <p className="user-answer">user's answer</p>
            </li>
          </ol>
        </div>
      );
}
```

- 필요한 데이터를 싹다 가져올거임
- Quiz에서 UserAnswers를 전달하면 산출할 수 있음

```jsx
import quizCompleteImg from "../assets/quiz-complete.png";
import QUESTIONS from "../../questions";

export default function Summary({ userAnswers }) {
  const skippedAnswers = userAnswers.filter((answer) => answer === null);
  const correctAnswers = userAnswers.filter(
    (answer, index) => answer === QUESTIONS[index].answers[0]
  );

  const skippedAnswersShare = Math.round(
    (skippedAnswers.length / userAnswers.length) * 100
  );

  const correctAnswersShare = Math.round(
    (correctAnswers.length / userAnswers.length) * 100
  );

  const wrongAnswersShare = 100 - skippedAnswersShare - correctAnswersShare;

  return (
    <div id="summary">
      <img src={quizCompleteImg} />
      <h2>Quiz done</h2>
      <div id="summary-stats">
        <p>
          <span className="number">{skippedAnswersShare}%</span>
          <span className="text">skipped</span>
        </p>
        <p>
          <span className="number">{correctAnswersShare}%</span>
          <span className="text">correct</span>
        </p>
        <p>
          <span className="number">{wrongAnswersShare}%</span>
          <span className="text">incorrect</span>
        </p>
      </div>
      <ol>
        {userAnswers.map((answer, index) => {
          let cssClass = "user-answer";

          if (answer === null) cssClass += " skipped";
          else if (answer === QUESTIONS[index].answers[0]) {
            cssClass += " correct";
          } else {
            cssClass += " wrong";
          }
          return (
            <li key={answer}>
              <h3>{index + 1}</h3>
              <p className="question">{QUESTIONS[index].text}</p>
              <p className={cssClass}>{answer ?? "skipped"}</p>
            </li>
          );
        })}
      </ol>
    </div>
  );
}

```

- two children with same key 에러가 뜸
    - li의 key값이 answer임
        - skip이 두개 이상이면 null값이 key인 element가 두 개 이상이게 됨
        - key를 index로 바꿔주면 해결
- map 함수에서 index 사용을 하지 않는 것이 좋다
    - 데이터에 연결된 것이 아니고 위치에 연결되어 있기 때문
    - 배열 내 데이터의 위치가 교환되면 인덱스는 바뀌지 않음

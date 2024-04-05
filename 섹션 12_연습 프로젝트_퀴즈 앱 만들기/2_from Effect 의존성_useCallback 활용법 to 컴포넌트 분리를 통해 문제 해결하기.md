# Effect 의존성 & useCallback 활용법 ~ 컴포넌트 분리를 통해 문제 해결하기

### Effect 의존성 & useCallback 활용법

- 현재 상황:
    - 이론: useEffect 함수가 재실행되어서는 안됨
    - 실제: setTimeout를 wrap한 useEffect만 재실행되고 있음
    - 원인: 의존성 배열 중 timeout은 변치 않는 상수이므로 onTimeout 쪽 때문
    
    ```jsx
    # Quiz.jsx
    
    <QuestionTimer timeout={10000} onTimeout={() => handleSelectAnswer(null)} />
    ```
    
    ```jsx
    # QuestionTimer.jsx
    
        useEffect(() => {
            console.log('SETTING TIMEOUT');
            setTimeout(onTimeout, timeout);
        }, [onTimeout, timeout]);
    ```
    
- 해결법: useCallback 활용
    - 실행: 이후가 아닌 초기에 timeout과 interval이 설정되는 것으로 변경
        - 다음 질문으로 넘어갈 때 timeout이 재설정되지 않아 정상적으로 작동하게 됨
        - BUT 타이머 만료 후 이상한 정지 기간은 여전히 존재
    
    ```jsx
    # Quiz.jsx
    
    import { useCallback, useState } from 'react';
    
    // 중략
    
        const handleSelectAnswer = useCallback(function handleSelectAnswer(selectedAnswer) {
            setUserAnswers((prevUserAnswers) => {
                return [...prevUserAnswers, selectedAnswer];
            });
        }, []);
    
        const handleSkipAnswer = useCallback(() => handleSelectAnswer(null), [handleSelectAnswer]);
        
    // 중략
        
                        <QuestionTimer timeout={10000} onTimeout={handleSkipAnswer} />
    ```
    

### Effect Cleanup 함수 활용 & 컴포넌트 초기화 Key(키) 사용법

- 현재 상황: 타이머는 10초로 설정했는데 5초만에 progress bar가 만료됨
    - 원인: setInterval이 두 번 실행되기 때문 (Strict Mode 때문에 버그가 있음을 인지)
- 해결법: cleanup 함수를 추가
    
    ```jsx
    # QuestionTimer.jsx
    
        useEffect(() => {
            console.log('SETTING TIMEOUT');
            const timer = setTimeout(onTimeout, timeout);
            
            return () => {
                clearTimeout(timer);
            };
        }, [onTimeout, timeout]);
    
        useEffect(() => {
            console.log('SETTING INTERVAL');
            const interval = setInterval(() => {
                setRemainingTime((prevRemainingTime) => prevRemainingTime - 100);
            }, 100);
            
            return () => {
                clearInterval(interval);
            };
        }, []);
    ```
    
    - 실행:
        - 정상적으로 10초에 걸쳐 progress bar가 만료됨
        - BUT 다음 질문으로 넘어가면 progress bar가 원래대로 초기화되지 않음
        - QuestionTimer 파일이 재생성되지 않기 때문
- 해결법: key 속성을 추가
    - key 속성은 리액트의 빌트인 속성
    - key 속성에 QuestionTimer를 재생성시키기 위해 인덱스 번호를 넣어줌
    
    ```jsx
    # Quiz.jsx
    
                    <QuestionTimer timeout={10000} onTimeout={handleSkipAnswer} key={activeQuestionIndex} />
    ```
    
    - 실행:
        - 정상적으로 다음 질문으로 넘어가면 progress bar가 초기화됨

### 선택된 답변 강조 & 추가 State(상태) 관리

- 목표: 상황에 따른 선택지 색 변경 및 다음 퀴즈로 넘어가도록 구현해보자.
- 해결법: answerState라는 새로운 state를 활용
    - answerState를 useState로 생성
    - 답을 고르기 전에는 빈 값으로 초기화
    - 답을 고르면 handleSelectAnswer(의 useCallback 함수) 안의 setAnswerState를 answered로 변경
    - setTimeout으로 1초 뒤에 정답과 비교한 뒤 correct 혹은 wrong으로 변경하게 함
    - useCallback의 의존성 배열에 현재 퀴즈 번호(activeQuestionIndex)를 집어넣음

```jsx
# Quiz.jsx

    const [answerState, setAnswerState] = useState('');
    
// 중략

    const handleSelectAnswer = useCallback(
        function handleSelectAnswer(selectedAnswer) {
            setAnswerState('answered');
            
// 중략

            setTimeout(() => {
                if (selectedAnswer === QUESTIONS[activeQuestionIndex].answers[0]) {
                    setAnswerState('correct');
                } else {
                    setAnswerState('wrong');
                }
            }, 1000);
        },
        [activeQuestionIndex]
    );
```

- 또다른 문제: 사용자가 답을 선택하자마자 activeQuestionIndex가 곧바로 바뀌어버림
    - 해결법:
        - 인덱스번호를 answerState에 맞춰 조건부로 표기
        
        ```jsx
        # Quiz.jsx
        
            const activeQuestionIndex = answerState === '' ? userAnswers.length : userAnswers.length - 1;
        ```
        
        - answerState 초기화를 위해 setTimeout 안에 새로운 setTimeout을 추가 작성 (정답 확인 후 초기화까지 2초 걸림)
        
        ```jsx
        # Quiz.jsx
        
                    setTimeout(() => {
                        if (selectedAnswer === QUESTIONS[activeQuestionIndex].answers[0]) {
                            setAnswerState('correct');
                        } else {
                            setAnswerState('wrong');
                        }
        
                        setTimeout(() => {
                            setAnswerState('');
                        }, 2000);
                    }, 1000);
        ```
        
        - isSelected 상수: 선택한 보기의 번호와 정답이 같은지 틀린지 T/F 상수로 추가 작성
        - cssClass 변수: 빈 값으로 추가 작성
            - 사용자가 보기 선택 후 selected
            - 선택한 보기와 정답을 비교한 뒤의 answerState 값 그대로 가져오도록
        - 선택지 버튼의 className 속성에도 cssClass 대입
        
        ```jsx
        # Quiz.jsx
        
                            {shuffledAnswers.map((answer) => {
                                const isSelected = userAnswers[userAnswers.length - 1] === answer;
                                let cssClass = '';
        
                                if (answerState === 'answered' && isSelected) {
                                    cssClass = 'selected';
                                }
        
                                if ((answerState === 'correct' || answerState === 'wrong') && isSelected) {
                                    cssClass = answerState;
                                }
        
                                return (
                                    <li key={answer} className="answer">
                                        <button onClick={() => handleSelectAnswer(answer)} className={cssClass}>
                                            {answer}
                                        </button>
                                    </li>
                                );
                            })}
        ```
        
        - 실행:
            - 보기에 하이라이트 효과 적용되었고, 정답 혹은 오답일 때에도 색상 효과 적용됨
            - BUT 클릭마다 여기저기 튀듯이 표현되는 문제

### 컴포넌트 분리를 통해 문제 해결하기

- 현재 상황: 답변을 고를 때마다 여기저기 튀듯이 나타남
    - 원인: answerState가 변할 때마다 선택지들을 다시 섞고 있기 때문
- 해결법: 새로운 state를 추가하거나,
    - 1) 새로운 state 추가:
        - shuffledAnswers라는 state를 추가 작성
        - useEffect 함수를 작성하고 의존성 배열에 activeQuestionIndex를 넣음
        - BUT 이 쪽 방법은 사용하지 않겠음!
            - 리액트 개발자는 useEffect의 사용을 최소화할 필요가 있기 때문
    - 2) 새로운 ref 추가:
        - shuffledAnswers라는 ref를 추가 작성
        
        ```jsx
        # Quiz.jsx
        
            const shuffledAnswers = useRef();
        ```
        
        - 하단에 적어뒀던 const shuffledAnswers를 shuffledAnswers.current로 수정
            - if문으로 undefined가 아닌지 걸러낼 것
        
        ```jsx
        # Quiz.jsx
        
            if (!shuffledAnswers.current) {
                shuffledAnswers.current = [...QUESTIONS[activeQuestionIndex].answers];
                shuffledAnswers.current.sort(() => Math.random() - 0.5);
            }
            
        // 중략
            
                                {shuffledAnswers.current.map((answer) => {
                                
        // 중략
                                
        		        })}
        ```
        
        - 실행:
            - 제대로 상황에 따른 하이라이트 효과가 적용됨
            - BUT 다음 문제로 넘어갈 때 골랐던 답변이 그대로 유지가 되어버리고 수정도 불가능
- 해결법: 새로운 컴포넌트를 작성하자
    - Answers 파일을 새로 생성
    - ul 태그 부분을 잘라내 Answers의 반환값으로 붙여넣기 (필요한 prop들도 가져오기)
    
    ```jsx
    # Answers.jsx
    
    export default function Answers({ answers, selectedAnswer, answerState, onSelect }) {
        return (
            <ul id="answers">
                {shuffledAnswers.current.map((answer) => {
                    const isSelected = selectedAnswer === answer;
                    let cssClass = '';
    
                    if (answerState === 'answered' && isSelected) {
                        cssClass = 'selected';
                    }
    
                    if ((answerState === 'correct' || answerState === 'wrong') && isSelected) {
                        cssClass = answerState;
                    }
    
                    return (
                        <li key={answer} className="answer">
                            <button onClick={() => onSelect(answer)} className={cssClass}>
                                {answer}
                            </button>
                        </li>
                    );
                })}
            </ul>
        );
    }
    
    ```
    
    - Quiz 파일에 Answers 컴포넌트를 가져옴
    
    ```jsx
    # Quiz.jsx
    
        return (
            <div id="quiz">
                <div id="question">
                    <QuestionTimer timeout={10000} onTimeout={handleSkipAnswer} key={activeQuestionIndex} />
                    <h2>{QUESTIONS[activeQuestionIndex].text}</h2>
                    <Answers />
                </div>
            </div>
        );
    ```
    
    - Answers로 넘겨줄 prop들을 작성하고, 보기를 섞는 if문과 관련 코드를 Answers로 옮김
    
    ```jsx
    # Quiz.jsx
    
                    <Answers
                        answers={QUESTIONS[activeQuestionIndex].answers}
                        selectedAnswer={userAnswers[userAnswers.length - 1]}
                        answerState={answerState}
                        onSelect={handleSelectAnswer}
                    />
    ```
    
    ```jsx
    # Answers.jsx
    
        const shuffledAnswers = useRef();
    
        if (!shuffledAnswers.current) {
            shuffledAnswers.current = [...answers];
            shuffledAnswers.current.sort(() => Math.random() - 0.5);
        }
    ```
    
    - 이전에도 활용했던 key 속성을 추가
        - 리액트가 key 속성을 통해 컴포넌트를 강제로 삭제 및 재생성하게 할 수 있음
    
    ```jsx
    # Quiz.jsx
    
                    <Answers
                        key={activeQuestionIndex}
                        answers={QUESTIONS[activeQuestionIndex].answers}
                        selectedAnswer={userAnswers[userAnswers.length - 1]}
                        answerState={answerState}
                        onSelect={handleSelectAnswer}
                    />
    ```
    
    - 실행:
        - 제대로 다음 퀴즈로 넘어가면 선택한 보기가 초기화됨
        - BUT progress bar가 두 개 생기는 버그 발생 (콘솔에 오류도 발생)
            - 원인: 같은 key로 여러 요소를 사용했기 때문
            - QuestionTimer와 Answers에서 동시에 activeQuestionIndex를 key로 사용하는 중
- 해결법: 또 새로운 컴포넌트를 작성하자
    - Question.jsx 파일을 새로 생성
    - id가 question인 div 태그 전체를 잘라내 Question 파일에 붙여넣기
    
    ```jsx
    # Question.jsx
    
    export default function Question() {
        return (
            <div id="question">
                <QuestionTimer timeout={10000} onTimeout={handleSkipAnswer} key={activeQuestionIndex} />
                <h2>{QUESTIONS[activeQuestionIndex].text}</h2>
                <Answers
                    key={activeQuestionIndex}
                    answers={QUESTIONS[activeQuestionIndex].answers}
                    selectedAnswer={userAnswers[userAnswers.length - 1]}
                    answerState={answerState}
                    onSelect={handleSelectAnswer}
                />
            </div>
        );
    }
    
    ```
    
    - Quiz 파일과 Question 파일에서 알맞게 prop을 오가게 작성하고, 기존의 두 key를 삭제하고 부모 컴포넌트에 하나의 key로 재작성
    
    ```jsx
    # Quiz.jsx
    
                <Question
                    key={activeQuestionIndex}
                    questionText={QUESTIONS[activeQuestionIndex].text}
                    answers={QUESTIONS[activeQuestionIndex].answers}
                    onSelectAnswer={handleSelectAnswer}
                    selectedAnswer={userAnswers[userAnswers.length - 1]}
                    answerState={answerState}
                    onSkipAnswer={handleSkipAnswer}
                />
    ```
    
    ```jsx
    # Question.jsx
    
    import QuestionTimer from './QuestionTimer';
    import Answers from './Answers';
    
    export default function Question({ questionText, answers, onSelectAnswer, selectedAnswer, answerState, onSkipAnswer }) {
        return (
            <div id="question">
                <QuestionTimer timeout={10000} onTimeout={onSkipAnswer} />
                <h2>{questionText}</h2>
                <Answers
                    answers={answers}
                    selectedAnswer={selectedAnswer}
                    answerState={answerState}
                    onSelect={onSelectAnswer}
                />
            </div>
        );
    }
    
    ```
    
    - 실행:
        - 정상적으로 오류 없이 실행됨 (key 관련 오류 사라짐)

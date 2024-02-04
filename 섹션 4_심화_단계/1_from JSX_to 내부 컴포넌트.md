### JSX 코드를 사용하지 않아도 되는 이유

- JSX 코드는 리액트가 HTML로 번역하기 전의 중간 코드임
    - Build Process를 거쳐 브라우저가 렌더링 가능한 HTML 코드로 번역
- 이론상 JSX를 필수가 아님
    - 순수 JS로 대체 가능

```jsx
<div id="content">
	<p> Hello World! </p>
</div>

=> 

React.createElement('div', {id='content'}) ..
// 생성 속성 + prop
```

- 당연히 가독성 및 유지보수 측면에서 JSX가 더 좋음

### Fragments

```jsx
.. 
return {
	<div>
		<Header/>
		<section id="main/>
	</div>
}
```

→ 이런 컴포넌트를 export 할떄 최상위 div가 필수가 아니지만 지우면 에러가 발생

→ JSX 요소는 최소 하나의 부모 요소가 필요하기 때문임

- JS function에서 return할 element를 두 개 이상 지정할 수 없는 이유와 동일
- fragment 문 활용 가능

```jsx
import {fragment} from React;

return {
	<Fragment>
		<Header />
		<Section id="main />
	</Fragment>
}
// 불필요한 div 없앨 수 있음
```

- 최신 버전은 Fragment 없이 empty tag 사용 가능 ( <> ) → 자동 Fragment

### 컴포넌트 분리 왜 해요?

- 기능별로 분리하는게 좋음
    - 하나의 컴포넌트가 관리하는 state와 data를 모듈화
    - 같은 데이터를 여러 개의 기능이 재사용할 때 당연히 코드가 꼬일 수 있음

### Feature 및 State로 컴포넌트 분리하기

```jsx
import { useState } from 'react';

import { CORE_CONCEPTS } from './data.js';
import Header from './components/Header/Header.jsx';
import CoreConcept from './components/CoreConcept.jsx';
import TabButton from './components/TabButton.jsx';
import { EXAMPLES } from './data.js';

function App() {
  const [selectedTopic, setSelectedTopic] = useState();

  function handleSelect(selectedButton) {
    // selectedButton => 'components', 'jsx', 'props', 'state'
    setSelectedTopic(selectedButton);
    // console.log(selectedTopic);
  }

  console.log('APP COMPONENT EXECUTING');

  let tabContent = <p>Please select a topic.</p>;

  if (selectedTopic) {
    tabContent = (
      <div id="tab-content">
        <h3>{EXAMPLES[selectedTopic].title}</h3>
        <p>{EXAMPLES[selectedTopic].description}</p>
        <pre>
          <code>{EXAMPLES[selectedTopic].code}</code>
        </pre>
      </div>
    );
  }

  return (
    <div>
      <Header />
      <main>
				// CoreConcepts 분리
        <section id="core-concepts">
          <h2>Core Concepts</h2>
          <ul>
            {CORE_CONCEPTS.map((conceptItem) => (
              <CoreConcept key={conceptItem.title} {...conceptItem} />
            ))}
          </ul>
        </section>
				// <CoreConcepts />
				
				// Examples 분리
        <section id="examples">
          <h2>Examples</h2>
          <menu>
            <TabButton
              isSelected={selectedTopic === 'components'}
              onSelect={() => handleSelect('components')}
            >
              Components
            </TabButton>
            <TabButton
              isSelected={selectedTopic === 'jsx'}
              onSelect={() => handleSelect('jsx')}
            >
              JSX
            </TabButton>
            <TabButton
              isSelected={selectedTopic === 'props'}
              onSelect={() => handleSelect('props')}
            >
              Props
            </TabButton>
            <TabButton
              isSelected={selectedTopic === 'state'}
              onSelect={() => handleSelect('state')}
            >
              State
            </TabButton>
          </menu>
          {tabContent}
        </section>
				// <Examples />
      </main>
    </div>
  );
}

export default App;
```

```jsx
 // CoreConcepts.jsx
import CoreConcept from './CoreConcept.jsx'
import { CORE_CONDEPTS } from '../data.js'

export default function CoreConcepts() {
	return {
		<section id="core-concepts">
          <h2>Core Concepts</h2>
          <ul>
            {CORE_CONCEPTS.map((conceptItem) => (
              <CoreConcept key={conceptItem.title} {...conceptItem} />
            ))}
          </ul>
        </section>
	}
}
```

```jsx
// Examples.jsx
// Examples를 feature 및 state 별로 분리하기
import { useState } from 'react';
import {Examples} from '../data.js';

export default function Examples() {
	cosnt [selectedTopic, setSelectedTopic] = useState()

	function handleSelect(selectedButton) {
		setSelectedTopic(selectedButton)
	}

	let tabContent = <p>Please Select a topic</p>
	if (selectedTopic) {
		tabContent= {
			<div id="tab-content">
				<h3>{Examples[selectedTopic].title}</h3>
				<p>{Examples[selectedTopic].description}</p>
				<pre>
					<code>{Examples[selectedTopic].code}</code>
				</pre>
			</div>

		return {
		
		<section id="examples">
          <h2>Examples</h2>
          <menu>
            <TabButton
              isSelected={selectedTopic === 'components'}
              onSelect={() => handleSelect('components')}
            >
              Components
            </TabButton>
            <TabButton
              isSelected={selectedTopic === 'jsx'}
              onSelect={() => handleSelect('jsx')}
            >
              JSX
            </TabButton>
            <TabButton
              isSelected={selectedTopic === 'props'}
              onSelect={() => handleSelect('props')}
            >
              Props
            </TabButton>
            <TabButton
              isSelected={selectedTopic === 'state'}
              onSelect={() => handleSelect('state')}
            >
              State
            </TabButton>
          </menu>
          {tabContent}
    </section>
	}
}
```

### 내부 컴포넌트에 Props가 전달이 안 될 경우

- 위의 Examples.jsx의 section 하위 구조가 동일하므로 이를 반복하는 내부 컴포넌트를 생성한다 가정

```jsx
/// Section.jsx
export default function Section({title, children}) {
	return {
		<h2>Examples</h2>
		<section>
			<h3>{title}</h3>
			{children}
		</section>
	}
}
```

```jsx
// Examples.jsx
import Section from '../Section.jsx';
...
return {
	<Section title="Examples" id="examples">
		<menu>
			<TabButton 
				.....
	</Section>
}
```

→ section 및 내부 스타일링이 적용이 안됨

→ style css에서 id로 스타일을 지정했기 때문

- props = attribute
- prop을 커스텀 컴포넌트에 적용하면 자동적으로 전달되지 않을 수 있음
    - 예제의 id prop은 하위 컴포넌트로 자동 적용되거나 section의 id로 forwading 되지 않음
- forwarded props 혹은 proxy props 사용함

# 모듈 소개 ~ 컨텍스트 소개 및 부여

### 모듈 소개

- 학습 목표:
    - 복잡한 prop drilling 문제가 무엇이며, 왜 문제가 되는지 알아보자
    - 컴포넌트 구성을 보고 문제를 해결해보자
    - 리액트의 context API가 무엇이며, 왜 좋은지 알아보자
        - context API: 여러 컴포넌트에 펼쳐진 상태를 관리할 때 필요
    - reducer를 활용한 새로운 훅을 학습해보자

### Prop Drilling 이해 & 프로젝트 개요

- prop drilling:
    - 다수의 컴포넌트를 거쳐 prop을 전달하는 것
    - 전달을 위해 컴포넌트 구조를 계속 수정해줘야 해서 문제가 됨

### Prop Drilling: 컴포넌트 구성으로 해결하기

- Component Composition (컴포넌트 합성): prop drilling 해결법 1
    - 직접 prop 전달이 가능하도록 거쳐가는 상위 컴포넌트를 wrapper로 감싸 refactor(재조정)
- (1)
    
    ```jsx
    # Shop.jsx (잘라낸 코드)
    
            {DUMMY_PRODUCTS.map((product) => (
              <li key={product.id}>
                <Product {...product} onAddToCart={onAddItemToCart} />
              </li>
            ))}
    ```
    
    ```jsx
    # App.jsx (붙여넣은 후)
    
    import { DUMMY_PRODUCTS } from "./dummy-products.js";
    import Product from "./components/Product.jsx"; // Shop.jsx에서 긁어온 import
    
          {/* 필요없는 onAddItemToCart prop 삭제 */}
          <Shop>
            {DUMMY_PRODUCTS.map((product) => (
                {/* onAddItemToCart를 handleAddItemToCart로 수정 */}
                <Product {...product} onAddToCart={handleAddItemToCart} />
              </li>
            ))}
          </Shop>
    ```
    
    - Shop.jsx에서 Proudct 컴포넌트를 렌더링하는 코드를 잘라낸다. (받아오는 prop도 삭제)
    - App.jsx로 올라와 Shop 컴포넌트 시작과 종료 태그 사이에 붙여넣는다. (import도 맞추기)
    - App.jsx에서 전달해줄 함수명을 (onAddItemToCart를 handleAddItemToCart로) 수정한다.
    - Shop 컴포넌트에 전달할 필요가 없어진 onAddItemToCart라는 prop을 삭제한다.
- (2)
    
    ```jsx
    # Shop.jsx
    
    // children을 받아온다.
    export default function Shop({ children }) {
      return (
        <section id="shop">
          <h2>Elegant Clothing For Everyone</h2>
          {/* children을 동적으로 작성 */}
          <ul id="products">{children}</ul>
        </section>
      );
    }
    ```
    
    - Shop.jsx에서 특수 자식 속성인 children을 받아온다.
    - 원래 Product 컴포넌트가 있던 ul 태그 사이에 children을 동적으로 받아오도록 작성한다.
- 실행 결과
    - 정상적으로 화면이 처음처럼 잘 작동
    - 단, 모든 컴포넌트 층(layer)에 적용할 수는 없다!
        - 모든 컴포넌트가 App 컴포넌트로 들어가고, 나머지 컴포넌트는 wrapper로만 쓰이게 되기 때문
        - App 컴포넌트가 비대해지는 문제 발생 가능

### 컨텍스트 API 소개

- Component Composition (컴포넌트 합성): prop drilling 해결법 1
    - 리액트의 Context API를 활용
- Context API:
    - 컴포넌트 혹은 컴포넌트 레이어 간의 용이한 데이터 공유를 도와줌
    - 다수의 혹은 모든 컴포넌트를 묶는 context 값을 이용
    - context 값의 장점:
        - state(상태)와의 연결이 쉽다.
    - 묶은 하위 컴포넌트 모두에 context 값을 전달
    - 상태의 변경 혹은 조회가 필요한 컴포넌트는 직접 context에 접근 가능하도록 함

### 컨텍스트 소개 및 부여

- 연습 목표: Context API를 활용해보자.
- (1)
    
    ```jsx
    # shopping-cart-context.jsx
    
    import { createContext } from "react";
    
    export const CartContext = createContext({
      // 장바구니 물건들을 넣을 items 배열
      items: [],
    });
    ```
    
    ```jsx
    # App.jsx
    
    import { CartContext } from "./store/shopping-cart-context.jsx"; // context API로 쓸 컴포넌트 import
    
        // CartContext에 들어있는 Provider 속성을 온점으로 찍어 작성해 컴포넌트로 활용
        <CartContext.Provider>
          <Header
            cart={shoppingCart}
            onUpdateCartItemQuantity={handleUpdateCartItemQuantity}
          />
          <Shop>
            {DUMMY_PRODUCTS.map((product) => (
              <li key={product.id}>
                <Product {...product} onAddToCart={handleAddItemToCart} />
              </li>
            ))}
          </Shop>
        </CartContext.Provider>
    ```
    
    - store 폴더를 src 안에 생성한다.
        - context 값들을 저장한 파일들을 모아둔 폴더를 말함
    - store 폴더 안에 shopping-cart-context.jsx 파일을 만든다.
    - shopping-cart-context.jsx에서 createContext를 import한 후 변수(상수)에 저장
        - createContext로 만든 값은 리액트 컴포넌트가 들어가는 객체가 됨
        - 바깥에서도 사용 가능하도록 export
    - 묶어줄 상위 컴포넌트인 App.jsx에서 CartContext 변수(상수)를 import해 컴포넌트로 활용
        - CartContext가 대문자로 시작하는 이유:
            - context 값을 컴포넌트처럼 활용하기 위함
            - 정확히는 그 값의 Provider라는 속성으로 명명된 컴포넌트처럼 활용하기 위함
        - CartContext.Provider로 컴포넌트 태그를 열고 닫아주자

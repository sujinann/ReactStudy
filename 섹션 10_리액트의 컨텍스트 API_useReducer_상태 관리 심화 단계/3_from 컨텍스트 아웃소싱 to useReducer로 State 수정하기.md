## 컨텍스트 아웃소싱 & 분리된 제공자 컴포넌트에 State 부여

현재는 컨텍스트를 통해 공유될 값들의 설정이 하나의 컴포넌트에서 이루어지고 있음

- 여러개의 컨텍스트를 다루게 되면 무거워진다는 단점이 생김
- 각각의 다른 컨텍스트를 이용해야 하기 때문
- 이를 피하기 위해 스토어 사용
- 모든 상태값과 관리코드를 한 곳에서 관리

```jsx
// shopping-cart-context.jsx

import { createContext, useState } from 'react';

import { DUMMY_PRODUCTS } from '../dummy-products.js';

export const CartContext = createContext({
  items: [],
  addItemToCart: () => {},
  updateItemQuantity: () => {},
});

export default function CartContextProvider({children}) {
  const [shoppingCart, setShoppingCart] = useState({
    items: [],
  });

  function handleAddItemToCart(id) {
    setShoppingCart((prevShoppingCart) => {
      const updatedItems = [...prevShoppingCart.items];

      const existingCartItemIndex = updatedItems.findIndex(
        (cartItem) => cartItem.id === id
      );
      const existingCartItem = updatedItems[existingCartItemIndex];

      if (existingCartItem) {
        const updatedItem = {
          ...existingCartItem,
          quantity: existingCartItem.quantity + 1,
        };
        updatedItems[existingCartItemIndex] = updatedItem;
      } else {
        const product = DUMMY_PRODUCTS.find((product) => product.id === id);
        updatedItems.push({
          id: id,
          name: product.title,
          price: product.price,
          quantity: 1,
        });
      }

      return {
        items: updatedItems,
      };
    });
  }

  function handleUpdateCartItemQuantity(productId, amount) {
    setShoppingCart((prevShoppingCart) => {
      const updatedItems = [...prevShoppingCart.items];
      const updatedItemIndex = updatedItems.findIndex(
        (item) => item.id === productId
      );

      const updatedItem = {
        ...updatedItems[updatedItemIndex],
      };

      updatedItem.quantity += amount;

      if (updatedItem.quantity <= 0) {
        updatedItems.splice(updatedItemIndex, 1);
      } else {
        updatedItems[updatedItemIndex] = updatedItem;
      }

      return {
        items: updatedItems,
      };
    });
  }

  const ctxValue = {
    items: shoppingCart.items,
    addItemToCart: handleAddItemToCart,
    updateItemQuantity: handleUpdateCartItemQuantity,
  };

  // 감싸 주는 것도 여기서 일괄 처리
  return <CartContext.Provider value={ctxValue}>
    {children}
  </CartContext.Provider>
}
```

```jsx
// App.jsx 스토어 관리 이전

import { useState } from 'react';

import Header from './components/Header.jsx';
import Shop from './components/Shop.jsx';
import Product from './components/Product.jsx';
import { DUMMY_PRODUCTS } from './dummy-products.js';
import { CartContext } from './store/shopping-cart-context.jsx';

function App() {
  const [shoppingCart, setShoppingCart] = useState({
    items: [],
  });

  function handleAddItemToCart(id) {
    setShoppingCart((prevShoppingCart) => {
      const updatedItems = [...prevShoppingCart.items];

      const existingCartItemIndex = updatedItems.findIndex(
        (cartItem) => cartItem.id === id
      );
      const existingCartItem = updatedItems[existingCartItemIndex];

      if (existingCartItem) {
        const updatedItem = {
          ...existingCartItem,
          quantity: existingCartItem.quantity + 1,
        };
        updatedItems[existingCartItemIndex] = updatedItem;
      } else {
        const product = DUMMY_PRODUCTS.find((product) => product.id === id);
        updatedItems.push({
          id: id,
          name: product.title,
          price: product.price,
          quantity: 1,
        });
      }

      return {
        items: updatedItems,
      };
    });
  }

  function handleUpdateCartItemQuantity(productId, amount) {
    setShoppingCart((prevShoppingCart) => {
      const updatedItems = [...prevShoppingCart.items];
      const updatedItemIndex = updatedItems.findIndex(
        (item) => item.id === productId
      );

      const updatedItem = {
        ...updatedItems[updatedItemIndex],
      };

      updatedItem.quantity += amount;

      if (updatedItem.quantity <= 0) {
        updatedItems.splice(updatedItemIndex, 1);
      } else {
        updatedItems[updatedItemIndex] = updatedItem;
      }

      return {
        items: updatedItems,
      };
    });
  }

  const ctxValue = {
    items: shoppingCart.items,
    addItemToCart: handleAddItemToCart,
    updateItemQuantity: handleUpdateCartItemQuantity,
  };

  return (
    <CartContext.Provider value={ctxValue}>
      <Header />
      <Shop>
        {DUMMY_PRODUCTS.map((product) => (
          <li key={product.id}>
            <Product {...product} />
          </li>
        ))}
      </Shop>
    </CartContext.Provider>
  );
}

export default App;
```

```jsx
// App.jsx 스토어 관리 이후
import Header from './components/Header.jsx';
import Shop from './components/Shop.jsx';
import Product from './components/Product.jsx';
import { DUMMY_PRODUCTS } from './dummy-products.js';
import CartContextProvider from './store/shopping-cart-context.jsx';

function App() {
  return (
    <CartContextProvider>
      <Header />
      <Shop>
        {DUMMY_PRODUCTS.map((product) => (
          <li key={product.id}>
            <Product {...product} />
          </li>
        ))}
      </Shop>
    </CartContextProvider>
  );
}

export default App;
```

<br/>

## useReducer 훅 소개

복잡한 상태 관리 함수들은 거의 항상 이전 상태를 기반으로 새 상태를 업데이트 하는 형식의 패턴을 반복함

- 이 부분을 보완하기 위해 Reducer 이용
- 상태 관리의 목적을 가지고 하나 또는 그 이상의 값을 보다 단순한 하나의 값으로 줄이는 것이 가능

```jsx
  // 리액트 내장 요소인 reduce 토대 개념
  const totalPrice = items.reduce(
    (acc, item) => acc + item.price * item.quantity,
    0
  );
```

```jsx
import { createContext, useState, useReducer } from 'react';

import { DUMMY_PRODUCTS } from '../dummy-products.js';

export const CartContext = createContext({
  items: [],
  addItemToCart: () => {},
  updateItemQuantity: () => {},
});

// 리듀서 사용 기초 양식
function shoppingCartReducer(state, action) {
  return state;
}

export default function CartContextProvider({ children }) {
  //dispatch 로 action 을 받음
  const [shoppingCartState, shoppingCartDispatch] = useReducer(
    shoppingCartReducer,
    {
      items: [],
    }
  );

  const [shoppingCart, setShoppingCart] = useState({
    items: [],
  });

  // ...이전 코드 생략

  
}
```

<br/>

## Action 보내기 & **useReducer로 State 수정하기**

- 상태를 한번에 관리하기 때문에 이전 상태를 불러올 때 걱정이 없어짐
- type 으로 함수 관리
- payload 로 인자 전달

```jsx
import { createContext, useReducer } from 'react';

import { DUMMY_PRODUCTS } from '../dummy-products.js';

export const CartContext = createContext({
  items: [],
  addItemToCart: () => {},
  updateItemQuantity: () => {},
});

function shoppingCartReducer(state, action) {
  if (action.type === 'ADD_ITEM') {
    const updatedItems = [...state.items];

    const existingCartItemIndex = updatedItems.findIndex(
      (cartItem) => cartItem.id === action.payload
    );
    const existingCartItem = updatedItems[existingCartItemIndex];

    if (existingCartItem) {
      const updatedItem = {
        ...existingCartItem,
        quantity: existingCartItem.quantity + 1,
      };
      updatedItems[existingCartItemIndex] = updatedItem;
    } else {
      const product = DUMMY_PRODUCTS.find(
        (product) => product.id === action.payload
      );
      updatedItems.push({
        id: action.payload,
        name: product.title,
        price: product.price,
        quantity: 1,
      });
    }

    return {
      ...state, // not needed here because we have only one value
      items: updatedItems,
    };
  }

  if (action.type === 'UPDATE_ITEM') {
    const updatedItems = [...state.items];
      const updatedItemIndex = updatedItems.findIndex(
        (item) => item.id === action.payload.productId
      );

      const updatedItem = {
        ...updatedItems[updatedItemIndex],
      };

      updatedItem.quantity += action.payload.amount;

      if (updatedItem.quantity <= 0) {
        updatedItems.splice(updatedItemIndex, 1);
      } else {
        updatedItems[updatedItemIndex] = updatedItem;
      }

      return {
        ...state,
        items: updatedItems,
      };
  }
  return state;
}

export default function CartContextProvider({ children }) {
  const [shoppingCartState, shoppingCartDispatch] = useReducer(
    shoppingCartReducer,
    {
      items: [],
    }
  );

  function handleAddItemToCart(id) {
    shoppingCartDispatch({
      type: 'ADD_ITEM',
      payload: id,
    });
  }

  function handleUpdateCartItemQuantity(productId, amount) {
    shoppingCartDispatch({
      type: 'UPDATE_ITEM',
      payload: {
        productId,
        amount
      }
    });
  }

  const ctxValue = {
    items: shoppingCartState.items,
    addItemToCart: handleAddItemToCart,
    updateItemQuantity: handleUpdateCartItemQuantity,
  };

  return (
    <CartContext.Provider value={ctxValue}>{children}</CartContext.Provider>
  );
}
```

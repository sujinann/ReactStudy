### 컨텍스트 소비하기

- 컨텍스트 내용을 사용할거임

```jsx
import { useContext } from "react";

import CartContext from "../store/shopping-card-context";

export default function Cart({ items, onUpdateItemQuantity }) {
  const cartCtx = useContext((CartContext));
  ..
  return (
    <div id="cart">
	    {/* items.length -> cartCtx.items.length */}
      {cartCtx.items.length === 0 && <p>No items in cart!</p>}
      {cartCtx.items.length > 0 && (
```

- ‘value’ prop is required for <Context.Provider> 에러
    - Context를 사용하는 가장 상위 컴포넌트에서 wrapping을 해주면 하위 컴포넌트에서 Context를 사용할 수 있음
    - 한번만 해줄것

```jsx
// App.jsx
    <CartContext.Provider value={{ items : [] }}>
```

- deconstructing 가능

```jsx
  const { items } = useContext(CartContext);
```

### 컨텍스트와 상태 연결

- App의 item 배열과 연결하고 싶음
- shoppingCart에서 제공하는 context를 사용하자

```jsx
return (
    // <CartContext.Provider value={{ items : [] }}>
    <CartContext.Provider value={shoppingCart}>
```

- 상태 전체를 이렇게 설정하면 읽을 수는 있지만 상태 수정은 컨텍스트로 어려움
- 대신 shoppingCart로 prop을 전달해야함
- Context를 직접 사용하면 prop 전달할 필요가 없음

```jsx
const ctxValue = {
    items: shoppingCart.items,
    addItemToCart: handleAddItemToCart,
  };
```

- ctxValue의 addItemToCart를 호출해서 handleAdd.. 가능함

```jsx
import { useContext } from "react";

import { CartContext } from "../store/shopping-card-context";

export default function Product({
  id,
  image,
  title,
  price,
  description,
  // onAddToCart,
}) {
  const {addItemToCart} = useContext(CartContext);

  return (
    <article className="product">
      <img src={image} alt={title} />
      <div className="product-content">
        <div>
          <h3>{title}</h3>
          <p className='product-price'>${price}</p>
          <p>{description}</p>
        </div>
        <p className='product-actions'>
          <button onClick={() => addItemToCart(id)}>Add to Cart</button>
        </p>
      </div>
    </article>
  );
}

```

### 컨텍스트롤 소비하는 여러 가지 방법

- 이제 여러 Prop과 Prop-drilling을 갖다버릴 수 있음
- useContext 방법은 Context 접근하는 일반적인 방법
- 구식의 code base 방법이면 Consumer로 Context를 사용하는 모든 컴포넌트를 묶음 (Provider - Consumer)

```jsx
// Cart.jsx
 return (
    <CartContext.Consumer>
      {(cartCtx) => {
        const totalPrice = items.reduce(
          (acc, item) => acc + item.price * item.quantity,
          0
        );
        return (
          <div id="cart">
          ...
```

- 복잡하고 번거로워서 적합하진 않음
    - 이런게 있더라

### 컨텍스트 값이 바뀌면 생기는 일

- 컨텍스트 value에 접근해서 바뀌면 값을 사용하는 컴포넌트 함수가 리액트에 의해 재실행됨
    - useContext 훅을 사용하거나 관련 값을 사용할 때
    - UI 업데이트를 위함

### 전체 앱에 컨텍스트 적용하기

- 컨텍스트로 전역변수를 관리하므로 일부 Props는 필요없음

```jsx
 // App.jsx
 
  <Header
        // cart={shoppingCart}
        // onUpdateCartItemQuantity={handleUpdateCartItemQuantity}
  />
      ..
		 {/* <Product {...product} onAddToCart={handleAddItemToCart} /> */}
    <Product {...product} />
 
 // Header.jsx
 import { useContext, useRef } from 'react';

import CartModal from './CartModal.jsx';
import { CartContext } from '../store/shopping-card-context.jsx';

// export default function Header({ cart, onUpdateCartItemQuantity }) {
export default function Header() {
  const modal = useRef();
  const {items } = useContext(CartContext);

  const cartQuantity = cart.items.length;

  function handleOpenCartClick() {
    modal.current.open();
  }

  let modalActions = <button>Close</button>;

  if (cartQuantity > 0) {
    modalActions = (
      <>
        <button>Close</button>
        <button>Checkout</button>
      </>
    );
  }

  return (
    <>
      <CartModal
        // ref={modal}
        // cartItems={cart.items}
        // onUpdateCartItemQuantity={onUpdateCartItemQuantity}
        // title="Your Cart"
        // actions={modalActions}
      />
      <header id="main-header">
        <div id="main-title">
          <img src="logo.png" alt="Elegant model" />
          <h1>Elegant Context</h1>
        </div>
        <p>
          <button onClick={handleOpenCartClick}>Cart ({cartQuantity})</button>
        </p>
      </header>
    </>
  );
}

```

- update function이 context에 없어서 context와 App의 ctxValue에 추가해줌

```jsx
const ctxValue = {
    items: shoppingCart.items,
    addItemToCart: handleAddItemToCart,
    updateItemQuantity : handleUpdateCartItemQuantity,
  }
```

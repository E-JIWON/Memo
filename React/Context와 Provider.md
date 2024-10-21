context api와 provider

### 1. 컨텍스트(context)
- context는 데이터 자체를 정의하는 객체이다.
- React.creatContext()를 사용하여 생성된다.
- 주로 데이터 형태와 기본 값을 정의
- 컴포넌트 트리 전체에서 공유될 데이터의 "그릇"이라고 생각 할 수 있다.

### 2. 프로바이더 (Provider)
- Provider는 Context의 값을 제공하는 React 컴포넌트이다.
- Context 객체의 .Provder 속성을 사용하여 생성됩니다.
- 실제 데이터와 그 데이터를 조작하는 함수들은 자식 컴포넌트들에게 제공한다.
- 컴포넌트 트리의 상위에 위치하여 하위 컴포넌트들에게 Context 값을 전달한다.

```jsx
// OrderContext.ts
improt React from 'react';

type OrderType = 'dineIn' | 'takeout' | null;

interface OrderContextType {
  orderType: OrderType;
  setOrderType: (type: OrderType) => void;
}
export const OrderContext = React.createContext<OrderContextType | undefined>(undefined);

// OrderProvider.tsx
import React, { useState } from 'react';
import { OrderContext } from './OrderContext';

export function OrderProvider({ children }: { children: React.ReactNode }) {
  const [orderType, setOrderType] = useState<OrderType>(null);

  // Provider 컴포넌트 정의 및 반환
  return (
    <OrderContext.Provider value={{ orderType, setOrderType }}>
      {children}
    </OrderContext.Provider>
  );
}
```
- React.createContext로 새로운 Context의 인스턴스를 만든다.
- provider는 context의 값을 동적으로 변경할 수 있게 해준다. context만으로는 정적인 값만 제공할 수 있지만, provider를 통해 상태를 관리하고 업데이트 할 수 있다. 또한 Provider는 React의 렌더링 최적화를 위한 메커니즘을 제공한다.
- OrderProvider.tsx는 Provider 컴포넌트 형태를 정의한다. 이 컴포넌트는 실제 상태(orderType)와 상태를 변경하는 함수(setOrderType)를 관리하고, 이를 Context를 통해 자식 컴포넌트한테 제공한다.

### 3. 자식 컴포넌트 사용
```tsx
// App.tsx
import { OrderProvider } from './OrderProvider';

function App() {
  return (
    <OrderContext.Provider value={{ orderType, setOrderType }}>
      {/* 여기에 자식 컴포넌트들 */}
      {children}
    </OrderContext.Provider>
  );
}

// 자식 컴포넌트에서 사용
import { useContext } from 'react';
import { OrderContext } from './OrderContext';

function ChildComponent() {
  const { orderType, setOrderType } = useContext(OrderContext);
  // orderType과 setOrderType 사용
}
```
요약하면, Context는 데이터의 "형태"를 정의하고, Provider는 실제 데이터와 그 데이터를 조작하는 방법을 제공한다.
이 둘을 분리함으로써 코드 관심사를 분리하고 재 사용성을 높일 수 있다.

### 4. 💡 중첩이 길어지면 아래처럼 사용해보자.
```
function CombinedProvider({ children }) {
  return (
    <ContextAProvider>
      <ContextBProvider>
        <ContextCProvider>
          {children}
        </ContextCProvider>
      </ContextBProvider>
    </ContextAProvider>
  );
}

// 사용
<CombinedProvider>
  <App />
</CombinedProvider>
```


### 5. 키오스크 프로젝트 내 - OrderContext에 대해 자세히 뜯어보자.
```jsx
// orderContext.tsx
'use client';

import React, { createContext, useContext, useState, ReactNode } from 'react';

type OrderOption = 'dine-in' | 'takeout' | null;

interface OrderContextType {
  orderOption: OrderOption;
  setOrderOption: (option: OrderOption) => void;
}

const OrderContext = createContext<OrderContextType | undefined>(undefined);

export const OrderProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [orderOption, setOrderOption] = useState<OrderOption>(null);

  return (
    <OrderContext.Provider value={{ orderOption, setOrderOption }}>
      {children}
    </OrderContext.Provider>
  );
};
```
💡`createContext`가 뭐야 ?
- React의 Context API를 사용하기 위한 함수이다.
- 이 함수는 컴포넌트 트리 전체에서 데이터를 공유할 수 있는 Context 객체를 생성합니다.
- 여기서는 OrderContextType 타입의 컨텍스트를 생성하고 있습니다.
- ❓쉽게 말해서 `createContext`는 어플리케이션 전체에서 사용할 수 있는 '공유 저장소'를 만드는 함수이다.

💡 갑자기 `OrderContext.Provider`가 어디서 나온거야?
- `OrderContext.Provider`는 `createContext`로 생성된 컨텍스트 객체의 `Provider` 컴포넌트이다.
- 이 컴포넌트는 자식 컴포넌트들에게 컨텍스트 값을 제공하는 역할을 한다.
- `OrderContext`만으로는 값을 제공할 수 없고, 반드시 `.Provider`를 사용해야 합니다.
- ❓ 쉽게 말해서 `OrderContext.Provider`는 우리가 만든 '공유 저장소'의 내용을 실제로 컴포넌트들에게 전달해주는 특별한 컴포넌트이다. 이걸 사용해야 다른 컴포넌트들이 우리가 공유하고 싶은 데이터에 접근할 수 있다.
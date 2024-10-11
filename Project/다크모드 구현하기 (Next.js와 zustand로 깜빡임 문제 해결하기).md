
😇 ~~어우 'use client' ..~~ 
열받았던 게 부끄러울 정도로 초급자의 시선에서 바라본 글입니다. 이번 다크모드 프로젝트에서는 클라이언트와 서버 간의 데이터 흐름을 유지하면서 깜빡임(Flicker) 문제를 해결하는 데 중점을 두었습니다.

> 😇 **Q: 아니 왜 Vercel에서 지원하는 next-themes를 사용하지 않았나요?**
>>
>**A:** Next-themes는 굉장히 쉽게 적용할 수 있었고, SSR 문제였던 깜빡임도 없었지만, 추후 복잡한 로직 관리나, 직접 만들어보는 보기 위해 적용해보았습니다. + SSR 문제 해결 능력 키우기 ?🌚


#### 적용 라이브러리
> 1. Next.js 14 (app router)
> 2. Tailwnd CSS
> 3. zustand

***
## 시작해보겠습니다.
### 1. 전역 상태를 만들어 보자
이전 프로젝트에서 `jotai`를 써봤기 때문에 `zustand`나 `redux`를 사용해보고 싶었습니다. 하지만 작은 프로젝트에 `redux`를 쓰기에는 기본 구조가 복잡하고(보일러플레이트 코드가 많음), `jotai`를 쓰기엔 상태 변경 로직을 직접 만들고 싶어서 `zustand`로 설정했습니다.

#### 혹시 설치가 안되어있다면?
```bash
npm i zustand
```
굿 설치가 완료됐어요. 바로 적용해 봅시다.

#### 상태관리 코드

 `app/_module/store/themeStore.ts`
```ts
import { ThemeType } from "@/_types/ThemeType";
import { create } from "zustand";  

interface ThemeStore {
  theme: ThemeType;
  setTheme: (newTheme: ThemeType) => void;
  toggleTheme: () => void;
}  

/**
 * @desc 테마 전역 상태
 * @desc setTheme 특정 theme를 적용하기 위함
 * @desc toggle 호출로 토글
 */
const useThemeStore = create<ThemeStore>((set) => ({
  theme: "",
  setTheme: (newTheme: ThemeType) => set({ theme: newTheme }),
  toggleTheme: () =>
    set((state) => ({
      theme: state.theme === "dark" ? "light" : "dark",
    })),
}));

  
export default useThemeStore;
```

이 코드는 전역 상태를 관리하는 zustand의 설정입니다. `setTheme`와 `toggleTheme`를 분리하여 각 함수의 역할을 명확하게 하고, 다크모드를 위한 상태 관리를 간편하게 해줄겁니다.
***
### 2. ### 프로바이더 만들기

먼저 `ThemeProvider`를 생성하여 테마를 관리할 수 있는 환경을 설정할 것입니다.
서버에서 쿠키를 읽어 현재 테마를 결정하고, 이를 `ThemeDetector`에 전달합니다.

`app/_module/provider/(theme)/ThemeProvider.tsx`
```tsx
import React from "react";
import { cookies } from "next/headers";
import ThemeDetector from "./ThemeDetector";
import { ThemeType } from "@/_types/ThemeType";

const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  const serverCookie = cookies();
  const themeCookie = (serverCookie.get("theme")?.value ?? "") as ThemeType;

  return <ThemeDetector defaultTheme={themeCookie}>{children}</ThemeDetector>;
};

export default ThemeProvider;
```
- `server`에서 쿠키를 읽고 `ThemeDetector`에서 초기 값을 알 수 있도록 전달해줍니다.

### 3. 넘겨받은 쿠키를 `ThemeDetector` 에 적용해줍시다.
detector란 '감시자' 라는 뜻입니다. 
이녀석은 감시하는 놈입니다. 코드를 먼저 봅시다.

`app/_module/provider/(theme)/ThemeDetector.tsx`
```jsx
"use client";

import React, { useEffect, useState } from "react";
import useThemeStore from "@/_module/store/themeStore";
import { ThemeType } from "@/_types/ThemeType";

export default function ThemeDetector({
  defaultTheme,
  children,
}: {
  defaultTheme?: ThemeType;
  children: React.ReactNode;
}) {
  const { theme, setTheme } = useThemeStore();
  const [themeStatus, setThemeStatus] = useState(defaultTheme ?? "");

  // 초기값이 비어있지 않을 경우 theme을 적용
  useEffect(() => {
    if (themeStatus !== "") setTheme(themeStatus);
  }, []);

  // 상태가 변경될 경우 테마 상태 업데이트
  useEffect(() => {
    if (theme) {
      setThemeStatus(theme);
    }
  }, [theme]);

  return <div className={themeStatus}>{children}</div>;
}
```

- 이 친구는 client에서 작동하는 녀석이라 `use client` 선언을 해줍시다,
- 초기에 `ThemeProvider` 에서 쿠키에 저장되어있는 `defaultTheme`를 넘겨받습니다.
- `themeStatus` 상태변수를 만들어준 이유는. 서버 쿠키에서 넘어온 `defaultTheme` 를 적용하고, 후에 전역 상태인 `theme`가 변경될 경우 적용해주기 위해서 입니다.
- 추가로 설명드리자면, useEffect에서 themeStatus가 비어있지 않을 경우에 해주는 이유는
- 새로고침 할 때마다 `theme`의 전역 상태가 "" 비어있기 때문에 매칭을 해주기 위해서입니다.

### Tailwind에 다크 클래스 적용하기
방금 보신 코드에 `<div className={themeStatus}>` 까지 한다면 아무 변화도 안되어있을 것 같아요.
이제 Tailwind에 적용해봅시다.

#### 설치가... 안되어있을수도 있을까요? 
[테일윈드 Get Start 링크](https://tailwindcss.com/docs/installation )
```
npm install -D tailwindcss
npx tailwindcss init
```
문서 보고 열심히 새팅을 해봅시다.

`tailwind.config.js`
```js
import type { Config } from "tailwindcss";
const config: Config = {
  content: [ ...생략 ],
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        light: {
          bg: "#FFFFD1",
        },
        dark: {
          bg: "#128390",
  ...생략,
};
export default config;
```
- `darkMode : 'class'` 를 넣어줍시다!
- 추가로, 확인을 위해 `colors : {  }` 안에 `light`, `dark` 를 나누어서 넣어봅시다,
	```js
      colors: {
        light: {
          bg: "#FFFFD1",
        },
        dark: {
          bg: "#128390",
        },
    ```

### layout에 ThemeProvider를 넣고 적용해봅시다!
```tsx
    <html lang="ko">
      <body className={`antialiased`}>
        <ThemeProvider>
          <div className="bg-white text-black dark:bg-blue-950 dark:text-white">
            {children}
          </div>
        </ThemeProvider>
      </body>
    </html>
```

### 토글 버튼을 만들어볼까요?
```jsx
"use client"; 

import React from "react";
import Cookies from "universal-cookie";
import { useEffect } from "react";
import useThemeStore from "@/_module/store/themeStore";

const ThemeButton = () => {
  const cookies = new Cookies();
  const { theme, toggleTheme } = useThemeStore();

  useEffect(() => {
    cookies.set("theme", theme, { path: "/" });
  }, [theme]);

  return (
    <button
      className="bg-light-bg text-black dark:bg-dark-bg dark:text-white"
      onClick={() => { toggleTheme(); }}
      >
      {theme ? theme : "Lading..."}
    </button>
  );
};
export default ThemeButton;
```
- 클릭했을 때 toogleTheme를 호출하고, theme가 변경되었을 때 
- theme의 값을 Cookie에 세팅합니다
- 이때 저는 `universal-cookie` 라이브러리 사용했고 
- `npm i universal-cookie` 하셔서 설치하셔도 되고, 직접 설정해주셔도 됩니다!

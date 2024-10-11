
~~어우 'use client' 놈..~~  🌚고작 이거가지구 열받았던게 부끄러울 정도의 수준의 블로그 글입니다..
초급자임을 이해해주세요..

이번 다크모드 프로젝트에선 클라이언트와 서버간의 데이터 흐름을 유지하면서 깜빡임(Flicker) 문제를 해결하는 데 중점을 두었습니다.

> 아니 왜 vercel에서 지원하는 next-thems가 있는데 왜 안했나요?
>  
> Next-Thems는 
> - 굉장히 쉽게 적용할 수 있었지만, 깜빡임 문제는 없었지만 **SSR** 를 완벽하게 처리하지 못했다는 단점이 있었습니다. ~~솔직히 깜빡임만 없으면 테마 설정이라 꼭 SSR을 해야될 필요가 있나 싶지만~~ 내공을 다지기 위해 적용해보았다 정도로 이해해주시면 될 것 같아요..
> 

> 적용 라이브러리
> 1. Next.js 14 (app router)
> 2. Tailwnd CSS
> 3. zustand


### 1. 전역 상태를 먼저 만들어 줍시다.

전 프로젝트에서 `jotai`를 써봐서 `zustand`나 `redux`를 사용해보고 싶었습니다.
하지만 작은 프로젝트에 redux를 쓰기에는 복잡하고 무거울 것 같지만 

zustand가 없다면.... 설치를 해줍시다!
```bash
npm i zustand
```
굿 설치가 완료됐어요. 바로 적용을 해봅시다.

#### 상태관리 코드
 `app/_module/store/themeStore.ts`

```ts
import { ThemeType } from "@/app/_types/ThemeType";
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
코드를 설명드릴게요.
- `setTheme` 와 `toggleTheme()` 를 따로 둔 이유는 두 함수의 역할을 명백하게 나누기 위해서였어요. 


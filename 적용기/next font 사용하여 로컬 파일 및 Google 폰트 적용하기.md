`next/font`로 로컬 파일 폰트, google 폰트 적용하는 방법에 대해 포스팅 하려 해요.

### ❓`next/font` 왜 사용해요 ?
- FOUC 현상 방지 (자체 호스팅 지원, 레이아웃 안정성)
   -> 보통 웹 폰트는 페이지가 로드 된 후에 비동기적으로 적용되므로, 폰트가 완전히 로드되기 전까지 기본 폰트가 잠시 표시되고, 이로 인해 레이아웃이 바뀌는 "FOUC" (Flash of Unstyled Content) 현상이 발생되는데, `next/font` 는 자체적으로 폰트를 호스팅하고 CSS에 포함된 폰트 변수로 미리 스타일링 하여 이러한 문제를 방지한다.
- 성능 최적화 
  -> `next/font`는 자동으로 필요한 CSS를 생성하여 폰트 로드 시간을 줄이고 페이지 성능을 향상시킵니다.
- 무엇보다 간편 !

### 🚀 로컬 파일 적용하기 
#### 1. 폰트 파일 준비
- 프로젝트의 `public` 폴더에 폰트 파일을 넣는다.
예를 들어 `public/font/GeistVF.woff`에 폰트 파일 저장

#### 2. next/font 사용하여 적용하기 
```tsx
import localFont from "next/font/local";

const fonts = localFont({
  src: "./fonts/GeistVF.woff",
  variable: "--font-geist-sans",
  weight: "100 900",
});

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="ko">
      <body className={`${fonts.variable}  antialiased`}>{children}</body>
    </html>
  );
}
```


### 🚀 google 폰트 적용하기
> **Next.js google font 공식 문서**
> https://nextjs.org/docs/pages/building-your-application/optimizing/fonts#google-fonts

#### 1. 패키지 설치
```bash
npm i @next/font
```

#### 2. 적용하기
저자는 `next 14버전`, `app router` 기반입니다.
저는 전체 레이아웃에 적용할겁니다.

```jsx
// app/layout.tsx
import { Gowun_Batang } from '@next/font/google';

const gowunBatangFont = Gowun_Batang({
  subsets: ['latin'], 			// 필요한 언어 설정
  weight: ['400', '700'],       // 사용할 폰트 굵기 설정
  display: 'swap',              // 텍스트 표시 방식
});

export default function RootLayout({
   children,
}: Readonly<{
   children: React.ReactNode;
}>) {
   return (
      <html lang="ko">
         <body className={`${gowunBatangFont.className} antialiased`}>
               {children}
         </body>
      </html>
   );
}

```




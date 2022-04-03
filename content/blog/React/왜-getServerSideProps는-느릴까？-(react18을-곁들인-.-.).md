---
title: 왜 getServerSideProps는 느릴까？ (react18을 곁들인 . .)
date: 2022-03-31 19:29:05
category: React
thumbnail: { thumbnailSrc }
draft: true
---

![](../../assets/getServerProp.png)

## 가설1. CDN설정..?

1. CDN 문제라면 getStaticProps또한 느려야되는데, 배포된 웹을 보면 되게 빠르다.
2. 캐시 문제 일까도 생각해봤는데
3. 결정적으로 배포를 진행한 플랫폼 netlify는 자체 cdn을 활용한다 . .

<br>

## 가설2. 내부적인 로직의 문제..?

```jsx
리엑트 SSR 과정
1. 서버에서, 전체 앱을 위한 데이터 fetch
2. 서버에서, html을 렌더링 시키고 response를 보냄
3. 클라이언트에서, 필요한 자바스크립트 코드를 다운
4. hydration: 클라이언트에서, 서버가 만든 html 파일에 자바스크립트 파일을 연결

중요한 점: 각 단계 모두 전 단계가 완전히 끝나야 다음 단계를 시작할 수 있다.
전체 앱 중 일부가 다른곳보다 느리다면, 중대한 문제가 될 수 있다.
```

출처: [New Suspense SSR Architecture in React 18](https://github.com/reactwg/react-18/discussions/37)([한글 번역 블로그](https://velog.io/@jay/React-18-%EB%B3%80%EA%B2%BD%EC%A0%90))

두번째 단계가 내부적으로 구현되어 있는게 만약 html을 한방에 바꿔주는 renderToString이라 느려진다면, 노드 스트림을 활용한 renderToNodeStream을 쓰면 기다리지 않고 response를 전송할수 있어지고, 그래서 빨라질수 있지 않을까? 라고 생각했지만,

리액트 18 이전에선 서버단 suspense를 지원하지 않고 있어, 한 부분에서 느려진다면 아무것도 보이지않고 흰 화면만 보이게 되었다.  
getServerSideProps를 통한 API 패칭 시 느리다는 외국어 글도 여럿 있는걸로 보아, 이것 말고도 다른 이유가 있을것 같기도 한데, 더 알아낸다면 추가 예정!

<br>

참고)

1. [Upgrading to React 18 on the server](https://github.com/reactwg/react-18/discussions/22)를 보면 renderToString, renderToNodeStream, renderToPipeableStream을 비교하고 있는데,
   rederToNodeStream의 경우 suspense는 되지만, 스트리밍은 되지 않아 18버전에선 deprecated 될것이라고 한다.  
   어제 릴리즈 된(22.03.30) 리액트18을 보니 renderToPipeableStream이 있는걸로 보아 확실히 deprecated 될것같다.
2. Next 공식문서에도 특별히 꼭 서버에서 가져와야 하는 경우가 아니라면 getServerSideProps의 사용을 지양하라고 한다. ( 캐시 설정을 하지 않는다면 CDN에 캐시 되지 않아 계속 패치되기도 한다.)

<br>

## 결론

1. Next를 통해 SSR을 하려고 하는데
2. getServerSideProps를 쓰고
3. api를 패치 받는 등 시간이 걸리는 작업을 할땐
4. 동기적으로 진행되기 때문에 흰 화면이 지속된다.

Next에서 리액트18에 추가된 스트리밍을 getServerSideProps에 적용해주었으면 . . . 좋겠다 !

<br>

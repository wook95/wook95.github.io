---
title: next 배포중 replaceAll is not a function 에러
date: 2022-02-16 18:49:21
category: Trouble shooting
thumbnail: { thumbnailSrc }
draft: false
---

### 문제

<br>
로컬에선 잘 되던 replaceAll . . .

vercel로도 netlify로도 어떻게 배포를 해도 자꾸 에러가 발생했다.

```jsx
[GET] /_next/data/-CYXDZ_c1lqEurL4uFptp/index.json
03:15:11:00
FUNCTION_RATE_LIMIT: This Serverless Function was rate limited
```

```jsx
2022-02-15T18:15:18.279Z ebaa2d25-b874-48ef-a207-b53b9ea56163 ERROR TypeError: strings[1].replaceAll is not a function
    at /var/task/.next/server/pages/items/[id].js:186:24
    at Array.map (<anonymous>)
    at ItemDetail (/var/task/.next/server/pages/items/[id].js:184:7)
    at d (/var/task/node_modules/react-dom/cjs/react-dom-server.node.production.min.js:33:498)
    at bb (/var/task/node_modules/react-dom/cjs/react-dom-server.node.production.min.js:36:16)
    at a.b.render (/var/task/node_modules/react-dom/cjs/react-dom-server.node.production.min.js:42:43)
    at a.b.read (/var/task/node_modules/react-dom/cjs/react-dom-server.node.production.min.js:41:83)
    at Object.exports.renderToString (/var/task/node_modules/react-dom/cjs/react-dom-server.node.production.min.js:52:138)
    at Object.renderPage (/var/task/node_modules/next/dist/server/render.js:736:46)
    at Object.defaultGetInitialProps (/var/task/node_modules/next/dist/server/render.js:368:51) {
  page: '/items/[id]'
}
RequestId: ebaa2d25-b874-48ef-a207-b53b9ea56163 Error: Runtime exited with error: exit status 1
Runtime.ExitError
```

<br>

[에크마 스크립트 2022 명세](https://tc39.es/ecma262/multipage/#sec-intro)를 보면 2021년, es12에 추가된 메소드로 `replaceAll`을 소개하고 있다. 추가된지 얼마 되지 않은 메소드여서, 폴리필을 추가해주지 않아 문제가 생겼다.

<br>

### 해결

1. 다른 메소드를 통해 해결한다.

   정규표현식과 replace 메소드를 통해 replaceAll 기능을 구현한다.

   ```javascript
   replaceAll('-','') -> replace(/-/g,'');
   ```

2. core-js를 이용한 폴리필

   `import 'core-js/features/string/replace-all'`

3. .babelrc에서 preset-env 프리셋을 사용한 폴리필

```json
{
  "presets": [
    [
      "next/babel",
      {
        "@babel/preset-env": {
          "targets": "> 0.25%, not dead", //시장 점유율25%이상의, 지원이 끊기지 않은 브라우져
          "useBuiltIns": "entry",
          "corejs": { "version": 3, "proposals": true }
        }
      }
    ]
  ]
}
```

# 49장 Babel과 Webpack을 이용한 ES6+/ES.NEXT 개발 환경 구축

대부분 최신 브라우저에서는 ES6의 지원률이 높지만, IE 11의 지원율은 약 11%이다. 그리고 매년 새롭게 도입되는 (ES6+ )이상 버전과 제안 단계에 있는 ES 제안 사항 은 브라우저에 따라 지원율이 제각각이다.

때문에 IE를 포함한 구형 브라우저에서 문제 없이 동작시키기 위한 개발 환경을 구축해야 한다

ES6 모듈은 대부분의 모던 브라우저에서 사용할 수 있다. 하지만 다음과 같은 이유로 **별도의 모듈 로더**를 사용하는 것이 일반 적이다.

-   IE를 포함한 구형 브라우저는 ESM 지원 X
-   ESM을 사용하더라도 트랜스파일링이나 번들링이 필요한 것은 변함이 없다.
-   ESM이 아직 지원하지 않는 기능이 있고 점차 해결되고는 있지만 아직 몇가지 이슈가 존재한다.

## 1. Babel

Babel을 사용하면 ES6+/ES.NEXT(다음버전)로 구현된 최신 사양의 소스코드를 IE 같은 구형 브라우저에서도 동작하는 ES5 사양의 소스코드로 변환(트랜스파일링)시킬 수 있다.

**브라우저에서 모듈 로딩 테스트**

Node.js 환경에서는 CommonJS 방식의 모듈 로딩 시스템을 따라서 정상적으로 잘 동작되지만, 브라우저는 CommonJS의 require 함수를 지원하지 않으므로 트랜스파일링된 결과를 그대로 실행하면 에러가 발생한다.

브라우저의 ES6 모듈을 사용하도록 Babel을 설정할 수 있으나, ESM은 앞서 말한 3가지 문제점이 존재하기 때문에 WebPack을 사용한다.

## 2.**Webpack**

웹팩은 의존 관계에 있는 JS, CSS, 이미지 등의 리소스들을 하나의 파일로 번들링하는 모듈 번들러다.

**Webpack 설치**

```jsx
npm install --save-dev webpack webpack-cli
```

**babel-loader 설치**

```jsx
npm install --save-dev babel-loader
```

npm scripts를 변경하여 Babel 대신 Webpack을 실행하도록 수정

```jsx
"scripts": {
 "build": "webpack -w"
}
```

**webpack.config.js 설정 파일 작성**

**babel-polyfill 설치**

Promise, Object.assign, Array.from 등은 트랜스파일링이 되지 못하고 그대로 남아서 이를 위해 babel-polyfill을 설치한다.

```jsx
npm install @babel/polyfill
```

→ 이는 개발환경에서만 사용하는 것이 아닌 실제 운영환경에서도 사용해야하기 때문에 -save-dev 옵션을 지정하지 않는다.

바벨만 사용하는 경우 진입점 파일의 선두에 `import @babel/polyfill";`을 작성한다.

Webpack을 사용하는 경우 entry 배열에 폴리필을 추가하여 사용한다. `entry: ['@babel/polyfill', './src/js/main.js'],`

이후 명령어로 실행 후, 번들링된 파일을 확인해보면 폴리필이 추가된 것을 확인할 수 있음

---
layout: post
title: Webpack 실습 2
published: true
---

Webpack 실습 Series의 2편: `webpack-dev-server` 사용하기

웹개발을 하면서 HTML, CSS, 또는 JS를 고치고 브라우저를 refresh하여 바뀐것을 반영하는경우가 많았다. 

우리나라에서는 최근들어 live-reload라는 plugin을 사용하기 시작한것으로 안다. (이미 외국에서는 2년전부터 사용함.)

Webpack으로도 live-reload같은 효과를 plugin을 install하여 사용할수 있다. 그러나 그럴 필요가 없는것이 webpack은 live-reload보다 더 좋은 기능을 사용할수 있기 때문이다. 

Webpack을 실행할시 watch option을 줄수 있다. `webpack --watch`는 live-reload랑 상관없이 file이 바뀌면 bundle을 자동으로 만드는 역활을 한다.

`webpack-dev-server`는 이 `webpack --watch`기능을 사용하여 편한 개발환경을 제공한다.

`webpack-dev-server`는 browser refresh 없이 update된 내용을 보영줄수 있다. 이것은 SPA를 만들때 정말로 필요한 기능이다. Browser refresh를 SPA에서 하게되면 현제 program state가 바뀌기 때문이다. program state의 바뀜 없이 update를 적용할수 있다면 SPA 개발환경이 정말 편해질듯 하다.

browser refresh 없이 update를 적용하는것을 *Hot Module Replacement (HMR)* 라고 한다.

## `webpack-dev-server` 시작하기

`webpack-dev-server`는 npm module이다. install하자. 

```bash
npm install webpack-dev-server --save-dev
```

`webpack-dev-server`는 `webpack`가 마찬가지로 npm bin directory에 실행가능한 command를 만든다.

`webapck-dev-server`와 `webpack`은 command line에서 `--env` argument로 받은 값을 `webpack.config.js`에서 `module.exports`하는 function의 first argument로 받는다.

```js
// package.json
{
  "scripts": {
    "start": "webpack-dev-server --env development",
    "build": "webpack --env.target production"
  }
}
```

위처럼 `--env`의 value는 string또는 object일수 있다. `--env development`로 사용하면 string이 전달되고 `--env.target production`으로 사용하면 object이 전달된다.

예)

![Webpack with funciton](/images/webpack1-with-env.png)

위처럼 `webpack.config.js`를 바꿔 보았다. 마지막에 export되는 function을 보면 첫번째 argument로 env를 받는다. 이것을 실험삼아 console.log하고 있다.

아래처럼 다르게 실행시 `env`가 object일수도 있고 string일수도 있는것을 볼수 있다.


![webpack-env-prod](/images/webpack2-env-prod.png)

![webpack-env-dev](/images/webpack2-env-dev.png)

`npm start`를 하게되면 `webpack-dev-server`를 이용하여 file이 바뀔시 마다 bundle이 recompile되고 browser가 refresh 된다.

지금 까지는 Hot Module Replacement가 작동 안하고 있다. (browser refresh 없이 JavaScript를 update하는것.)

이제 HMR를 setting해보자.

## HMR configure하기.
`webpack-dev-server`에 Hot Module Replacement를 setting하기 위해서는 configuration에 `devServer`와 `HotModuleReplacementPlugin`을 사용하여야 한다.

한가지 방법으로는 이렇게 하는수가 있다.(아직 HMR이 제대로 작동 안한다.)

![webpack-hmr](/images/webpack2-naive-HMR.png)

이렇게 하게되면 일단 webpack이 error를 뛰우고 update가 hot load 되지 않는다. 아래와 같은 message가 console에 뜬다.

![webpack-hmr-error](/images/webpack2-HMR-error.png)

일단 error message에 `39 -> 36 -> 88`이라는 알아 들을수 없는 내용이 쓰이고 있다. 

이 메세지를 조금도 잘알아 듣기 위해 `NamedModules` plugin을 깔아보자.

![webpack2-named-modules](/images/webpack2-named-modules.png)

다시 code를 고치면 숫자로 나왔던 부분이 이제는 file 이름으로 나와 아래처럼 알아 들을수있는 message가 나온다.

![webpack2-named-modules-log](/images/webpack2-named-modules-log.png)




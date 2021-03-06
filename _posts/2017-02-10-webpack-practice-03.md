---
layout: post
title: Webpack 실습 3
published: true
---

Webpack 실습 Series의 3편: CSS Loading 기초

Webpack은 CSS를 hard refresh 없이 바꿀수 있다. 이글에서는 hot loading CSS를 배워 보도록 하겠다.

Webpack에서 CSS를 loading하기 위해서는 `css-loader`를 사용하여야 한다. `css-loader`는 `@import`와 `url()`을 ES6 `import`처럼 lookup을 한다.

만약 `@import`가 external resource를 loading한다면 `css-loader`는 그 `@import`를 skip한다. 오로지 internal resource만 webpack이 processing하는 것이다.

`css-loader`가 할일을 다 했다면 `style-loader`가 `css-loader`로 받은 output을 받아 bundle에 CSS를 inject한다.

이렇게 inject된 CSS는 JavaScript로 inlining 된다. 그리고 HMR interface를 사용한다. CSS가 JavaScript로 변하고 적용되는것은 Best Practice가 아니다. 그렇기 때문에 `ExtractTextPlugin`을 사용하여 분리된 CSS file을 만드는 것이 좋다. 일단은 CSS를 JavaScript로 inlining 해보자.

일단 webpack core loader가 아닌것들은 install해야 한다.

```bash
npm install css-loader style-loader --save-dev
```

아래 처럼 `webpack.config.js`를 바꿔보자. 중요한것은 `css-loader`와 `style-loader`는 이 config파일에서 require로 loading 할 필요가 없다는 것이다.

![module loader](/images/webpack2-module-loader.png)

그리고 아래 처럼 `app/main.css`를 만들고;

![main css](/images/webpack3-main-css.png)

`app/index.js`에서 CSS를 import하면 CSS가 inline JavaScript로 import 되어 반영 된다.

![import css](/images/webpack3-import-css.png)

CSS를 JS로 바꿔 반영하는게 신기 해서 build된 code를 보면;

`build/app.js`중 일부;

![build app](/images/webpack3-inline-css.png)

위 처럼 JavaScript file안에 우리의 CSS rule이 반영 된것을 볼수 있다. (정확이 어떻게 apply되는지는 알바 아니고)

`webpack-dev-server`를 사용하면 CSS를 마음대로 고치며 browser refresh없이 바로 반영 된다.


## CSS Scoping
CSS의 가장 큰 단점은 global scope에 있다는 것이다. 이것을 local scope으로 즉 `import` statement별로 그 import한 file에서만 반영되게 하는 기능이 CSS Modules에서 가능하다.

`css-loader`는 import별로 CSS를 반영할수 있게하는 option이 있다.
여기서 중요한것은 이 CSS scope은 *class* selector에만 적용된다.
(TODO: pseudo selector?)
또한 이 local class는 element에 binding을 직접 해주어야만 한다.
내가 생각했던 file별로 import하면 자동으로 그 file에만 작동 되는것이 아닌가 보다.

CSS module을 반영하려면 `css-loader`를 사용하는 부분에서 `css-loader?modules` option을 setting해야한다.

예) 아래의 CSS 규칙이 있다면;

```css
/* app.main.css */

body {
  background: cornsilk;
}

.redButton {
  background: red;
}
```

위 `.redButton`을 local로 반영하려면 다음과 같이 하면 된다.

```js
// app/component.js

import styles from './main.css';

element.className = styles.redbutton;
```

`body` CSS 규칙은 아직도 global 로 반영된다.(class가 아니어서 그렇다)
반면 `.redButton`은 local scope으로 오로지 `app/component.js`에서만 반영이 되고 global scope으로 적용되지 않는다.

단점은 element에 직접적으로 assign을 하지 않으면 반영이 않된다는것. (이거 너무 불편해서 쓸라나?)

## Sass Loading하기

`loader`는 array의 뒤에서부터 앞으로 반영된다.
더 쉽게 생각하는 방법은 `styleLoader(cssLoader())` 이런 식으로 반영된다 생각하면 편하다.

`sass-loader`는 `.sass` 또는 `.scss`를 css로 바꾼다 즉.
`styleLoader(cssLoader(sassLoader()))`이런 식으로 반영되는것이다. <small>왠지 elm같음</small>

```bash
npm install --save-dev sass-loader node-sass
```

`webpack.config.js`

```js
{
  module: {
    rules: [
      {
        test: /\.css$|\.sass$|\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader']
      }
    ]
  }
}
```

## Webpack의 CSS import 이해하기
Webpack은 css또는 sass/less에서 `@import`를 보면 JS에서의 import statement처럼 lookup을 시작한다.

```css
@import "./variables.scss";
```

위는 그냥 보통 sass 만으로도 되는것이라 별로 특별한것도 아니다. 허나 Webpack은 `node_modules` folder안에 있는 scss를 가지고 올수 있다. 
`node_modules`안에서 `scss`를 찾으라는 명령은 다음과 같이 `~`를 prefixing함으로서 가능하다.

```css
@import "~bootstrap/sass/bootstrap"
```

## Source map 사용하기
Source map은 `css-loader?sourceMap`으로 setting후 `output.publicPath`를 development server로 setting하면 된다.
(TODO: example)

이상 Webpack css loading 기초였다.











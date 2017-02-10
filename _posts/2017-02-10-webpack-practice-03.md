---
layout: post
title: Webpack 실습 3
published: true
---

Webpack 실습 Series의 3편: CSS Load하기

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





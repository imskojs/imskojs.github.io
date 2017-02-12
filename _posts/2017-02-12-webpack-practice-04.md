---
layout: post
title: Webpack 실습 4
published: true
---

Webpack 실습 Series의 4편: CSS loading 최적화

## CSS file 따로 save하기
지금것 모든 style은 JS에 inline으로 반영됬다. 이대로 사용하면 않된다. HTML, CSS, JS는 따로 관리하는게 best practice다. (react는 이걸 무시한다. 그래서 별로 정이 안간다. 그래도 따라야지 대세인데 음?)

Webpack은 `ExtractTextPlugin`을 사용하여 이것을 achieve한다.

```bash
npm install --save-dev extract-text-webpack-plugin@beta
```

이 `ExtractTextPlugin`은  `ExtractTextPlugin.extract`라는 loader도 (plugin이 아닌 loader!)도 포함하고 있다.

`ExtractTextPlugin.extract` loader는 plugin으로 extract할 file을 marking하는 역할을 한다. 그리고 `ExtractTextPlugin` plugin은 막상 marking된 css/scss를 process하여 output을 emit한다.
(emit이란? 그냥 만든다라고 이해하자)

아래 예를 보자.

![extract-css-loader](/images/webpack3-extract-css-loader)

(TODO: fallback 정확히 뭐지? 없어도 잘만돌아감)

위는 loader를 setting하고, 아래는 plugin을 setting 한다.

![extract-css-plugin](/images/webpack3-extract-css-plugin)

그리고 아래 처럼 `app/main2.scss`를 만들고

![main2-scss](/images/webpack3-main2-scss)

`app/component.js`에서 아래처럼 import를 하자.

![component-js](/images/webpack3-component-js)

그러면 아래처럼 `build/app.css`가 생기게 된다.

![app-css](/images/webpack3-app-css)

여기서 build된 css이름은 `webpack.config.js`의 `entry`이름 마다 하나씩 생기는 것이다. 이것을 세팅해주는 것은 `plugin` key에 `new ExtractTextPlugin('[name].css')`의 `[name]`부분이다.
지금 같은 경우 `entry:{app: PATH.app}`으로 되어있는것을 보고 `build/app.css`를 만드는 것이다.

한가지더 기존에 쓰고 있던 
```js
  new HtmlWebpackPlugin({
    title: 'Webpack demo',
  }),
```

이 알아서 `app.css`를 `build/index.html`에 loading해주는것을 볼수있다. 아래처럼;

![index-html](/images/webpack3-index-html)
---
layout: post
title: Webpack 실습 4
published: true
---

Webpack 실습 Series의 4편: CSS loading 최적화 기초

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

![extract-css-loader](/images/webpack3-extract-css-loader.png)

(TODO: fallback 정확히 뭐지? 없어도 잘만돌아감)

위는 loader를 setting하고, 아래는 plugin을 setting 한다.

![extract-css-plugin](/images/webpack3-extract-css-plugin.png)

그리고 아래 처럼 `app/main2.scss`를 만들고

![main2-scss](/images/webpack3-main2-scss.png)

`app/component.js`에서 아래처럼 import를 하자.

![component-js](/images/webpack3-component-js.png)

그러면 아래처럼 `build/app.css`가 생기게 된다.

![app-css](/images/webpack3-app-css.png)

여기서 build된 css이름은 `webpack.config.js`의 `entry`이름 마다 하나씩 생기는 것이다. 이것을 세팅해주는 것은 `plugin` key에 `new ExtractTextPlugin('[name].css')`의 `[name]`부분이다.
지금 같은 경우 `entry:{app: PATH.app}`으로 되어있는것을 보고 `build/app.css`를 만드는 것이다.

한가지더 기존에 쓰고 있던 

```js
  new HtmlWebpackPlugin({
    title: 'Webpack demo',
  }),
```

이 알아서 `app.css`를 `build/index.html`에 loading해주는것을 볼수있다. 아래처럼;

![index-html](/images/webpack3-index-html.png)

## JavaScript를 통하지 않은 CSS loading
솔직히 CSS가 `.js` file에서 import 되고 또 잠시 동안이나마 JS로 inline styling이 된다는것은 뭔가 좀 아닌거 같다. CSS는 `.css`의 file안에서 (또는 scss/less etc) 시작하여 `.css` file로 끝나는 것을 원할수 있다.

css file들만 globbing해서 `entry`로 주면 그 css file을 하나의 file로 만들어 주는것은 조금만 생각해보면 이해가 가는 Webpack의 방식이다.

일단 `.css` 또는 `.scss`의 file들을 가지고 오려면 `glob`을 install하자.

```bash
npm install --save-dev glob
```

```js
const glob = require('glob');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
  // array로 resolve된다.
  style: glob.sync('./app/**/*.css'),
};

const common = {
  entry: {
    app: PATHS.app,
    style: PATHS.style
  }
  ...
}
```

위처럼 config하고 build하면 `build/style.css`가 만들어진다. 추가로 `build/style.js`가 만들어 지는데 이건 무시하자(별 쓸모 없음).

`glob`은 alphabetic순으로 file들을 가지고 온다. 순서가 중요하다면 `.css` file하나만 entry에 넣고 `@import`로 그 file들을 순서대로 가지고 올수 있다.

## Autoprefixing
(TODO: autoprefixing 설명 구찮음)

autoprefixing을 사용하려면 여러 방법이 있지만 config가 조금더 자유로운 PostCSS loader를 사용해서 setting해보자.

```bash
npm install --save-dev postcss-loader autoprefixer
```

아래처럼 `webpack.config.js`를 만들면된다.

![autoprefixer](/images/webpack4-autoprefixer.png)

위 예에서 loader로 string이 아닌 Object가 들어간것을 볼수 있다. 그리고 이 loader 자체에 plugins를 넣는것도. 나중에 자세히 보자.

postcss는 sass-loader가 output으로 준 css file을 가지고 작업하는것을 주목하자.
(TODO: use postcss as a main rather than sass)

`app/main2.scss`를 아래와 같이 바꾸고;

![main2-scss](/images/webpack4-main2-scss.png)

build를 하면 아래 처럼 prefixing되어 나오는것을 볼수 있다.

![app-css](/images/webpack4-app-css.png)

원하는 browser를 setting하는 방법으로는 `package.json`에 `browserslist` key를 만들어 만들어 원하는 browser를 array에 넣어주면 된다.

예) 아래는 Internet Explorer를 무시하라고 autoprefixer에게 전달하는 방법이다.

![browserslist](/images/webpack4-browserslist.png)

## 쓸모 없는 CSS 지우기
보통 CSS framework를 사용하면 쓰는 부분보다 않쓰는 부분이 더 많이 있다.
않쓰는 부분을 자동으로 지워줬으면 할때 `PurifyCSS`를 사용하면 된다.

일단 간단한 CSS library인 purecss를 install해보자.

```bash
npm install --save purecss
```

그리고 `purifycss-webpack`을 install하자.

```bash
npm install --save-dev purifycss-webpack
```

간단한 예제로 test를 해보면;

`app/index.js`

```js
import 'purecss';
import './main.css';
import component from './component';

...
```

그리고 `purecss`에 있는것을 반영하자.

`app/component.js`

```js
module.exports = function () {
  const element = document.createElement('h1');

  element.className = 'pure-button';
  element.innerHTML = 'Hello world';

  return element;
};
```

build를 해보면 아래 처럼 `build/app.css`가 커지는것을 볼수 있다.

![purecss](/images/webpack4-purecss.png)

위 코드는 약 700줄이 넘는 CSS rule들이다.

이제 `purifycss-webpack`을 사용해보자.

```js
const PurifyCSSPlugin = require('purifycss-webpack');
```

![purifycss](/images/webpack4-purifycss.png)

이 setting후 build를 하면 사용하지 않는 css rule들은 전부 지워진다. 우리 코드에서 `purecss` framework에서 사용한 class는 `pure-button` 밖에 없다.

그렇기에 아래 처럼 `.pure-button`과 사용하는 tag들만 빼고 전부다 지워진다.

![app-css](/images/webpack4-app-css-purified.png)

약 700줄의 코드에서 140줄로 줄어 들었다.

이상 CSS loading 최적화 기초 였다.

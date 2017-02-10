---
layout: post
title: Webpack Development Environment
published: true
---

이 글에서는 Webpack으로 개발시 쓰는 tool들을 본다.

## Source Maps
모든 JavaScript가 `bundle.js`에 있기 때문에 error가 나면 어느 파일을 고처야 될지 모르는경우가 많다. 이런것을 해결하기 위해 나온것이 바로 source map이다.

Source map을 사용하려면 `webpack.config.js`에 아래를 쓰면 된다. (TODO: Explain source map)

```js
{
  devtool: "cheap-eval-source-map"
}
```

## `webpack-dev-server`

`webpack-dev-server`는 서버와 live reloading을 제공한다.

일단 install을 하자.

```bash
npm install webpack-dev-server --save-dev
```

`webpack-dev-server`를 사용하고 싶으면 `index.html`에서 `output.filename`을 pointing하도록 하자.

예)

```html
<script src="dist/bundle.js"></script>
```

## `webpack-dev-middleware`

만약 이미 express server가 있다면 webpack-dev-middleware를 이용해 webpack-dev-server를 직접 만들어 쓸수 있다.

이것의 좋은점은 기존에 있는 서버의 기능들을 전부다 사용하면서 webpack에서 제공하는 개발자 도구들을 쓸수 있는것이다.

### 사용방법

```bash
npm install express webpack-dev-middleware --save-dev
```

위 처럼 install후 middleware를 다음과 같이 사용하면 된다.

```js
var express = require("express");
var webpackDevMiddleware = require("webpack-dev-middleware");
var webpack = require("webpack");
var webpackConfig = require("./webpack.config");

var app = express();
var compiler = webpack(webpackConfig);

app.use(webpackDevMiddleware(compiler, {
  publicPath: "/" // 보통 `output.publicPath`와 같은 path다.
  filename: 'bundle.js'
}));

app.listen(3000, function () {
  console.log("Listening on port 3000!");
});
```
bundle은 `http://localhost:3000/bundle.js`에서 가지고 올수 있다.








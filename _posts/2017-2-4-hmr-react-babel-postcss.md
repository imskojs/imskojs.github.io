---
layout: post
title: Webpack HMR with babel and React
published: true
---

이 글은 Webpack의 HMR을 React, babel, postcss를 사용하여 설정하는 글이다.

## install해야 하는것들

```bash
npm i --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react babel-preset-stage-2

npm i --save-dev webpack webpack-dev-server

npm i --save-dev css-loader postcss-loader react-hot-loader style-loader

npm i --save react react-dom
```

## Babel Config
아래는 `.babelrc` 입니다.

```js
{
  "presets": [
    // Webpack은 import syntax를 이미 이해하고 있습니다. 그래서 babel이 
    //import를 handling하지 않도록 { modules: false }를 줍니다.
    ["es2015", {"modules": false}],

    // Stage 2는 "draft", 4 는 완료, 0은 그냥 아이디어.
    // https://tc39.github.io/process-document/
    "stage-2",

    // React.Component를 JavaScript로 Transpile할때 사용.
    "react"
  ],

  "plugins": [
    // React code가 HMR를 사용할수 있도록 합니다.
    "react-hot-loader/babel"
  ]
}
```

## Webpack Config

```js
const { resolve } = require('path');
const webpack = require('webpack');

module.exports = {
  entry: [
    // React를 위해 HMR을 activate 합니다.
    'react-hot-loader/patch',

    // webpack-dev-server 의 client를 위해 bundle
    // 그리고 설정된 endpoint에 connect하기.
    'webpack-dev-server/client?http://localhost:8080',

    // client를 hot reload되도록 bundle합니다.
    // `only-`는 hot reload를 성공한 update만 하게 합니다.
    'webpack/hot/only-dev-server',

    // app의 entry point
    './index.js'
  ],
  output: {
    // output bundle
    filename: 'bundle.js',

    path: resolve(__dirname, 'dist'),

    // HMR이 hot update chunk들을 load하기 위해 알아야 하는 path
    publicPath: '/'
  },

  context: resolve(__dirname, 'src'),

  devtool: 'inline-source-map',

  devServer: {
    // server에 HMR을 사용할수 있도록 설정.
    hot: true,

    // output path를 match하도록 합니다. 
    contentBase: resolve(__dirname, 'dist'),

    // `publicPath`의 ouput가 같게 설정.
    publicPath: '/'
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          'babel-loader',
        ],
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader?modules',
          'postcss-loader',
        ],
      },
    ],
  },

  plugins: [
    // HMR을 전체적으로 사용할수 있도록 설정.
    new webpack.HotModuleReplacementPlugin(),

    // 읽기 좋은 module이름을 browser의 console에 보이게 하기
    new webpack.NamedModulesPlugin(),
  ],
};
```

`webpack-dev-server`가 `dist` folder에 있는것들을 쏴주기 때문에 `dist` folder에 아래 file을 넣어 주어야 합니다.

```html
<!-- index.html -->

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Example Index</title>
</head>
<body>
  <div id="root"></div>
  <script src="bundle.js"></script>
</body>
</html>

```

`package.json`에 아래 코드를 써주세요.
```js
{
  "scripts" : {
    "start" : "webpack-dev-server"
  }
}
```

`npm start`를 치면 이제 `webpack-dev-server`가 작동합니다.

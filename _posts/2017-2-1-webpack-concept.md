---
layout: post
title: Webpack 개념.
published: true
---

Webpack을 사용하기 전에 아래 4가지 개념을 알면 편합니다.


## Entry
Webpack은 bundle을 만들기 위해 어떤 파일들을 사용하는지 recursion을 통해 `import` statement를 봅니다. `import` statement를 타고 load해야하는 file들을 찾아 가는것이지요. 이렇게 load해야하는 file들을 찾으면 dependency graph를 만들어 관리를 합니다. 그러기 위해서는 어디서 시작할지를 정해 주어야 합니다. 그 시작 포인트를 설정해 주는것이 `webpack.config.js`에서 `entry` property이지요.

예)

```js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

위 entry point를 시작으로 `import` statement를 타며 dependency graph를 만듭니다.


## Output
Bundle을 만들었으면 webpack에게 어디다가 bundle된 file을 놓을지 정해 주어야 합니다.
`output` property는 bundle된 code를 어떻게 할지 정해줍니다.

예)

```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

위 예제에서 `output.path`는 folder를 설정해주고 `output.filename`은 file이름을 설정해 줍니다.

## Loaders
Webpack 자체는 오직 JavaScript만 handle할수 있습니다. 그러나 loader를 사용하면 JavaScript가 아닌 다른 file들도 module로 만들어 dependency graph에 더할수 있습니다.
Loader는 Webpack에게 JavaScript가 아닌 module들을 dependency에 포함하여 bundle하는 방법을 알려줍니다.

Loader는 딱 2가지 일을 합니다.

1. 어떤 file들이 특정 loader로 변환될지 정합니다..
2. 정한 file들을 변환하여 dependency graph에 추가 합니다.

Loader를 사용하기 위해서는 `webpack.config.js`에서 `module` property에 다음과 같이 설정해 주어야 합니다.

```js
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  }
};

module.exports = config;
```

위 config의 rules property는 webpack에게 다음과 같이 명령을 합니다.

"Webpack compiler야, 만약 `.js` 또는 `.jsx` 를 import하면 bundle로 만들기 전에 `babel-loader`를 사용하여 transform 해줘!"


## Plugins
TODO: 설명더 필요

Plugin은 bundling이 완료된 chunk에 통채로 적용되는 것입니다.
Plugin을 사용하는 방법은 npm install로 우선 install한후 require로 import하여 사용하면 됩니다. 

또한 webpack자체에 내장되어 있는 core plugin도 있습니다.

예)

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```





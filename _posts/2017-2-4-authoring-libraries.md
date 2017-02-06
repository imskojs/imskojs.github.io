---
layout: post
title: Webpack Authoring Libraries
published: true
---

이 글에서는 Webpack 환경을 포함 어느 환경에서나 쓸수있는 library만드는 법을 본다.

우리는 여기서 Webpack을 사용하여 library를 bundle할것이다. 그리고 이렇게 bundle된 library는 여러 환경에서 쓸수 있게 하는 것이 목표이다.

아래의 file은 우리가 bundle하고 싶은 library이다.

*src/index.js*

```js
import _ from 'lodash';
import numRef from './ref.json';

export function numToWord(num) {
  return _.reduce(numRef, (accum, ref) => {
    return ref.num === num ? ref.word : accum;
  }, '');
};

export function wordToNum(word) {
  return _.reduce(numRef, (accum, ref) => {
    return ref.word === word && word.toLowerCase() ? ref.num : accum;
  }, -1);
};
```

위 `src/index.js`를 보면 `lodash`와 `./ref.json`을 import하고 있는것을 볼수 있다.
위 library를 우리는 브라우저 환경에서 script tag로 아래와 같이 사용할 것이다.
우리가 원하는것은 `lodash`를 포함한 bundle이 아니다. `lodash` library는 따로 사용자가 script tag로 또는 `npm install`로 load해야 한다.

*index.html*

```html

<html>
<script src="https://unpkg.com/lodahs.js"></script>
<script src="https://unpkg.com/webpack-numbers.js"></script>
<script>
    webpackNumbers.wordToNum('Five') //output is 5
</script>
</html>

```

또한 browser환경이 아닌 Node 환경에서도 아래와 같이 사용할 것이다.

```js
// Node에서 사용시 `lodash`가 `node_modules`에 있어야 한다.
// `lodash`는 `webpack-numbers`에서 `require`로 import하도록 쓰여있다.
// 즉 import _ from 'lodash' 는 Wepback이 var _ = require('lodash')로
// bundle된 file에 transpile하며 이후 Node.js가 require를 handling한다.
// 이때 `node_modules`에 `lodash`가 없다면 error가 날것이다. (TODO: error or warn like unmet dependency)
var webpackNumbers = require('webpack-numbers');
webpackNumbers.wordToNum('Five'); // out is 5
```

브라우저와 node환경에서 위에서 만든 `webpack-numbers`를 사용하려면 Webpack Config를 다음과 같이 쓰면 된다.

*webpack.config.js*

```js

var path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'webpack-numbers.js',
    // 브라우저 환경에서 `webpack-numbers.js`를 `script` tag로 load 하면
    // `webpackNumbers`라는 global `var`iable을 만든다.
    library: 'webpackNumbers',
    libraryTarget: 'var'
  },

  // 아래 code는 lodash가 install또는 script tag로 먼저 가지고 와야한다고 설정하는 것이다.
  externals: {
    lodash: {
      commonjs: 'lodash',
      commonjs2: 'lodash', // TODO: import statement?
      root: '_'
    }
  }
};

```

위 Webpack setting은 `dist/webpack-numbers.js`로 bundle을 만든다.

이제 이 `dist/webpack-numbers.js` 는 브라우저 환경에서 `script` tag로 사용할수 있다.

Node 환경에서도 `require`로 사용할수 있게 하기 위해서는 단 `package.json`에 `main` file (Node의 `entry` 같은 녀석)을 설정해 주면 된다.

*package.json*

```js
{
  "main": "dist/webpack-numbers.js"
}
```

위 와 더불어 미래 Node가 Front-end JavaScript 세상을 다루게 될것이다. 그리고 브라우저 환경에서도 `ES2015` JavaScript module import를 할수 있게 될것을 대비하여 아래와 같이 `package.json`에 Front-End용 `entry`를 설정하자. `module` property는 지금 안사용되지만 미래를 위한것이다.

그럼 위 `package.json`은 아래 처럼 `module` property가 하나 더 생기게 된다.

*package.json*

```js
{
  "main": "dist/webpack-numbers.js",
  "module": "src/index.js"
}
```





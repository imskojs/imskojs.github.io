---
layout: post
title: Webpack Shimming
published: true
---

오래된(legacy) 3rd Party Library는 global variable에 의존하는 경우가 많다. 이런 경우 Webpack이 잘 작동 안할수 있다. 이글은 이런 고장난 module을 어떻게 handling하는지 알아보는 글이다.

Webpack은 module loader로 import를 통하여 사용하는 file들을 가지고 온다. 
그러나 3rd party library는 global에 있는 file에 의존할수 있다. 이것을 handling하기 위해 webpack은 여러 option을 제공한다.

## Provide Plugin
`ProvidePlugin`은 이곳에 설정한 module를 모든 file에 Webpack에 의해 require된것 처럼 해줍니다.
다른 file에서 이곳에 지정해준 key를 variable로 사용하면 그때 import합니다.

예를 들어 아래 webpack config는 모든 JavaScript file에서  `$` 나 `jQuery`를 사용하면 file상단에 `var $ = require('jquery')`를 해줍니다.

```js
module.exports = {
  plugins: [
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery'
    })
  ]
};
```


## `imports-loader`
CommonJS context에서는 `this`가 `module.exports`이다. 이러면 다른 library에서는 문제가 될수 있다. 그래서 이런 library에 this를 window로 세팅을 해주는 config가 Webpack에 존재한다.

`imports-loader`는 3rd party library에 `this`를 원하는 object으로 설정해줄수 있다.

아래 코드 같은경우 `test`로 설정된 `module`의 `this`를 `window`로 설정해 준다.

```js
module.exports = {
  module: {
    rules: [{
      test: require.resolve("some-module"),
      use: 'imports-loader?this=>window'
    }]
  }
};
```

요즘은 JavaScript module들이 어느 환경에서나 쓸수 있도록 check를 해주는 경우가 있다. 보통 이런 경우 `define`을 먼저 check하고 이상한 방법으로 code를 export한다.

이런 경우 `define = false`를 하여 CommonJS를 사용하도록 경로를 정해주는것이 강제로 설정해줄수 있다.

예

```js
module.exports = {
  module: {
    rules: [{
      test: require.resolve("some-module"),
      use: 'imports-loader?define=>false'
    }]
  }
};
```


## `exports-loader`
library가 global variable을 만든다고 가정해보자.
이런 경우 우리는 `exports-loader`를 사용하여 CommonJS 형식으로 export할수 있다.

예를 들어 3rd party library가 global variable인 `file`과 `helpers.parse`를 만든다면 그리고 우리가 원래는 global variable이 되야 하는 `file`을 `file`로 CommonJS형식으로 export하고 `helpers.parse`를 `parse`로 CommonJS형식으로 export하려면 아래처럼 쓰면 된다.

*webpack.config.js*

```js
module.exports = {
  module: {
    rules: [{
      test: require.resolve("some-module"),
      use: 'exports-loader?file,parse=helpers.parse'
      // file 소스코드에 아래 코드를 더한다:
      //  exports["file"] = file;
      //  exports["parse"] = helpers.parse;
    }]
  }
};
```

## `script-loader`
`script-loader`는 global context에서 file안의 내용을 `eval()`하는것과 같다.
이 모드에서는 모든 legacy 3rd party library가 잘 작동할것이다.
단 이것을 사용하면 webpack이 제공하는 dev툴에서 열외가 된다. 예를 들어 minify마저도 못하는것이다.

script loader의 예)

```js
// legacy.js
var globalStuff = {a: 1};
```

`script-loader`를 다음과 같이 사용하면 (TODO:entry에서 사용하겠지?)

```js
require('sciprt-loader!legacy.js')
```

그럼 아래와 같이 eval이 된다.

```js
eval("var globalStuff = {a: 1};")
```

## `noParse` option
CommonJS version이 없는 legacy code를 `dist` folder에 그냥 넣고 싶다면 그 module을 `noParse`에 flag하면 된다. 그렇게 하면 webpack은 그 module을 parsing않하고 그냥 넣을것이다.

예)

```js
module.exports = {
  module: {
    noParse: /jquery|backbone/
  }
};
```

## minify되지 않은 CommonJS/AMD file을 사용하라.
보통 `node_modules`에 보면 library마다 `package.json`이 있다. 이곳에 보면 `main` property가 있는데 이것은 배포용 코드의 file path를 reference하는 것이다.

Webpack은 이 `node_modules`안에 들어있는 `require`를 다른 file에 있는 `require` 처럼 사용할수 있습니다. 그래서 Webpack을 사용시 `dist` version말고 `src` 버전을 가지고 오라고 config를 다음과 같이 하면 됩니다.

예)

```js
// webpack.config.js

module.exports = {
  resolve: {
    alias: {
      jquery: "jquery/src/jquery"
    }
  }
};
```






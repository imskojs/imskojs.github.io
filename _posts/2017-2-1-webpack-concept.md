---
layout: post
title: Webpack 개념.
published: true
---

Webpack을 사용하기 전에 아래 4가지 개념을 알면 편합니다.

## Entry
Webpack은 bundle을 만들기 위해 어떤 파일들을 사용하는지 recursion을 통해 `import` statement를 봅니다. `import` statement를 타고 load해야하는 file들을 찾아 가는것이지요. 그러기 위해서는 어디서 시작할지를 정해 주어야 합니다. 그 시작 포인트를 설정해 주는것이 `webpack.config.js`에서 `entry` property이지요.

예)

```js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```




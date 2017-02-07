---
layout: post
title: Webpack 실습 1
published: true
---

Webpack 실습 Series의 1편: 기초 개발환경 Setup

## Project Folder Setting 하기

아래 command를 실행햐여 `webpack-demo`라는 folder를 만들고 `npm init`을 실행하여 `package.json`을 만들어 보자. 이왕 하는김에 `git init`을 실행하여 git으로 version control을 하자.

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y
git init
```

## Webpack install하기

Webpack을 global install할수 있지만 local로 install하면 다른 환경에서도 똑같은 Webpack version을 사용하기 때문에 골치 아픈일이 덜할것이다 그러니 `npm install -g webpack`을 사용하지 말고 아래 처럼 local dev dependency로 install을 하자.

```bash
npm install --save-dev webpack
```

## Webpack 실행하기

Webpack을 global로 install했다면 terminal에서 `webpack` command로 실행할수 있다. 그라나 우리는 위에서 설명한 이유 때문에 Webpack을 local로 install했다.

이렇게 하면 단점이 webpack을 실행하기 위해서는 아래와 같이 써야 한다.

```bash
node_modules/.bin/webpack
```

이렇게 길게 쓰기 싫다면 `package.json`의 `scripts` property에 webpack command를 refer하면 node는 현제 프로젝트의 `node_modules/.bin`에서 webpack을 찾아 사용한다.

*package.json*

```js
{
  "scripts": {
    "build": "webpack"
  }
}
```

## 폴더 구조
`webpack-demo`의 folder구조는 다음과 같다.

![Folder Structure](/images/webpack1-folder-structure.png)

*app/component.js*

![component.js](/images/webpack1-component.png)

*app/index.js*

![index.js](/images/webpack1-index.png)

우리가 하려는것은 `app/`에 있는 file들을 bundle하여 `build/`로 넘기는 것이다.

그렇게 하기 위해서는 `wepback.config.js`를 건드려야 한다.

## `index.html`이 없다?!
Webpack을 처음 접할때 가장 당황스러운것은 `index.html`이 없다는 것이다. `index.html`이 없어 어디서 어떻게 `script` tag와 `link` tag를 가지고 오는지 도저히 이해를 못하겠다. 

이것은 webpack이 직접 `index.html`을 관리한다. 관린한다라는 뜻은  `script` tag와 `link` tag를알아서 넣어준다는것.

Webpack이 이렇게 `index.html`을 generate하기 위해서는 `html-webpack-plugin`을 사용하여야 한다.

```bash
npm install --save-dev html-webpack-plugin
```

## `webpack.config.js`
`app` folder에 있는 file들을 bundle하기 위해서는 `webpack.config.js`에 config setting을 해주어야 한다.

최소한 setting을 해주어야 하는것은 `entry`와 `output`이다.

`webpack.config.js`

![webpack-config](/images/webpack1-webpack-config.png)

`entry`의 `app` key의 value는 file을 가르켜야 한다 하지만 위에서 보면 `app` key의 value는 file이 아닌 folder이다. 이렇게 folder를 가르키는 경우 Webpack은 Node.js의 convension에 따라 `index.js`를 가르킨다.

## Webpack 실행

이제 아래 command를 사용하여 webpack을 실행하면 `build` folder에 bundling 된 app.js가 생긴다. 이 이름은 `webpack.config.js`의 `output`에 정의된 `[name].js` 이 정해 준것이다. `entry`에서의 key name이 output file의 [name]으로 정해지는것이다.

```bash
npm run build
```

![build structure](/images/webpack1-build-files.png)

아래 `build/index.html`은 `HtmlWebpackPlugin`이 만들어 준것이다. `HtmlWebpackPlugin`이 알아서 script tag를 넣어준다.

![build index](/images/webpack1-build-index.png)

이상 Webpack 실습 1: 기초 개발환경 Setup이었다.
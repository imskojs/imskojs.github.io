---
layout: post
title: Webpack Code 나누기(Splitting).
published: true
---

Webpack의 기초 사용 용도는 여러 `<script>`를 하나의 `bundle.js`로 만드는것이다. 그러나 이렇게 하게되면 `bundle.js`는 엄청 커질수가 있다. 처음에는 이렇게 생각할수 있다. `bundle.js`가 커지더라도 서버에 요청 한번만 보내니 여러 script를 나누어 요청을 보내는것 보다 한번 요청을 보내어 `bundle.js`를 가지고 오는것이 빠를 것이라고. 그러나 그렇지가 않다. (TODO: max cachable size)

일단 브라우저는 여러 file들을 cache할수 있다. 그러나 만약 file size가 커지면 브라우저는 그 file을 cache하지 않는다. 그렇다면 website에 다시 들어 올때 마다 다시 그 script를 load해야 하는것이다.

또 한가지 여러 파일을 sync로 요청 하지 않고 parellel로 loading할수도 있다. (물론 이건 modern browser에서만 가능하다.) (TODO: which browser support parallel script requesting)

Code Splitting을 해야 하는 이유는 위 2가지말고 여러가지가 더 있다. 그중 Webpack이 해주는것 중에 하나가 on-demand asset loading이다.
Code Splitting을 하게되면 특정 코드들은 유저가 특정 행동을 할때 load할수 있는것이다. 미리 load를 하지 않아 initial load time이 줄어 드는것이다.

## CSS Code Splitting
Webpack을 `css-loader`를 사용하여 CSS를 JavaScript file에 import 할수 있다. 그리고 이 JavaScript file loading되는 HTML에 이 CSS가 반영 된다. (TODO: how css gets loaded)

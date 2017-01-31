---
published: true
---
## 왜 Webpack을 써야하나.
Webpack은 bundle loader이다. 웹개발자로서 bundle loader라고 들으면 알아 듣기가 힘들다. 완전 새로운 개념이기 때문이다. 

bundle loader가 무엇인지 설명하기 전에 웹개발하면서 이건 아닌거 같은면 한가지의 예를 들어 보자.

웹브라우저에서는 jQuery같은 library를 `<script>` tag를 사용하여 사용한다. `<script>` tag를 사용하여 library를 load하면 jQuery 같은경우 `$` 라는 object이 window에 달라 붙는다. (`window.$` 가 생기는 것이다.) 

브라우저에서 `window`는 어느 파일에서나 보이는 global variable이다. 이런 기능을 사용하여 서로 다른 JavaScript file에서 jQuery간은 libarary를 사용 할수 있는것이다.
예)
```html
<html>
  <head>
    <title>webpack 2 demo</title>
    <script src="https://code.jquery.com/jquery-3.1.1.js"></script>
  </head>
  <body>
    <script src="app/index.js"></script>
  </body>
</html>
```

위 예에서 `index.js`안에서 `$`라는 object을 사용할수 있다. jQuery를 `<script>` tag로 `app/index.js`전에 loading했기 때문이다.

이것에 단점은 `<script>` tag의 순서가 중요하다는 것이다. 지금 같은 경우 jQuery만 사용한다면 그리 문제가 않되지만 많은 library를 사용하고 많은 CSS file을 사용하는 순간부터 많은 `<script>` tag와 `<link>` tag 들의 순서를 잘 정리 해야만 한다. 그리고 이렇게 되는 순간부터 `index.html`에 `<script>` tag와 `<link>` tag들이 난발하기 시작한다. 그리고 이때 부터 2가지의 문제가 생긴다. jQuery를 예를 들어 생각해 보자.

1. jQuery를 `<script>` tag로 가지고 오지 않고 다른 JavaScript file에서 jQuery를 사용하려는 순간 에러가 뜬다.
2. jQuery를 `<script>` tag로 가지고 왔는데 쓰지 않아 쓰잘때기 없이 브라우저가 안쓰는 script를 download받아야 한다.

이때 이것을 해결해 주는것이 bundle loader다.

bundle loader는 위 로딩되는 `<script>`들을 하나의 file로 묶는(bundle)다.
예를 들어 위에서 보인 HTML 예제는 다음과 같이 바뀔수 있다.
```html
<html>
  <head>
    <title>webpack 2 demo</title>
  </head>
  <body>
    <script src="dist/bundle.js"></script>
  </body>
</html>
```

위 예제의 포인트는 `<script>` tag가 기존에 아무리 많이 있다해도 하나로 바꿀수 있다는것이다. 그리고 중요한것은 순서를 신경써지 않아도 되고 사용하지 않는 script는 알아서 빼준다는것! 왜 이제 나온것이냐 할정도로 참 깔끔하다.


그리고 이 `<script>` tag의 순서가 굉장히 중요하다. 
## Webpack install하기.
```bash
## webpack 2 install하기.
npm install webpack@beta --save-dev

## 또는 특정 version을 install하려면.
npm install webpack@<version> --save-dev

## 또는 완전_최신(Bleeding Edge)을 install하려면.
npm install webpack/webpack#<tagname/branchname>
```

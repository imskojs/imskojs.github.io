---
layout: post
title: Webpack 실습 5
published: true
---

Webpack 실습 Series의 5편: Image loading

자질 구레한 asset들을 request하는것 만큼 app을 느리게 만드는 방법도 없다.

Webpack은 `url-loader`를 사용하여 image를 inline 즉 base64 string으로 JS bundle을 만들수 있다. 이렇게 하면 bundle 크기가 커지겠지만 request수는 적어 질수 있다.

Webpack은 모든 image loading을 `url-loader`를 사용하여 inlining한다. 그리고 `url-loader`는 특정 option을 통해 `file-loader`로 task를 넘길수 있다.

`file-loader`는 image path를 return해주고 다른 종류의 asset도 loading할수 있다.

POINT: `url-loader`가 모든 asset을 handling하려 하고 handling하기를 원하지 않는다면 그때 `file-loader`에게 넘긴다.

## `url-loader` setting하기
`url-loader`는 limit option을 사용할수 있다. 이 limit option을 사용하여 image handling을 `file-loader`로 넘길수 있는것이다.

이렇게 하면 작은 image는 inlining해버리고 큰 image만 file로 만들어 주면 된다.

webpack은 기본적으로 모든 css 규칙의 `url()`를 handling하는것은 물론 JS에서 image asset setting하는것도 handling해준다. `url()`은 css-loader가 lookup 해준다.

일단 loader들을 install하자.

```bash
npm install --save-dev file-loader url-loader
```

사이즈가 5kB(5000 byte)이하인 `.jpg`나 `.png` 파일을 로딩 하기 위해서는 `url-loader`를 다음과 같이 세팅 해주어야 한다.
이렇게 하면 `file-loader`는 알아서 5kB보다 큰 file을 `[name].[ext]`로 output해준다. 

즉 `url-loader`의 option으로 setting해준 `name` key의 value가 `file-loader`로 넘어 가는것이다. 

여기서 중요한것은 `file-loader`를 어디에도 사용 안했다는것 오로지 `npm install file-loader`를 했다는것에 있다.

Output file은 다음과 같이 `[path][name].[hash].[ext]` option들이 있다. 

예) `./images/[name][hash].[ext]`로 설정해 줄수 있다.

예)
`test.png` image를 만들어 아래처럼 `app/test.png`로 저장하자. 이 이미지는 5kB이다

![webpack5-app-test-image](/images/webpack5-app-test-image.png)

CSS에 `url()`을 사용하여 위 저장된 `test.png`를 가지고 오자.
`app/main.css`;

![webpack5-main-css-bg-url](/images/webpack5-main-css-bg-url.png)

`webpack.config.js`를 아래와 같이 setting하고 build하면

![webpack5-webpack-config-url-loader](/images/webpack5-webpack-config-url-loader.png)

아래처럼 `build/images/test.png`가 만들어 진다.

![webpack5-build-images-test](/images/webpack5-build-images-test.png)

기존에 css 규칙은 `build`로 넘어오면서 url path가 바뀐것을 아래처럼 볼수가 있다.

![webpack5-path-resolve](/images/webpack5-path-resolve.png)

JavaScript코드에서도 된다는것! 이건 직접 해보삼.
(TODO: HTML src에 있는 녀석도 되나?)


## Image 최적화 하기.
간단하게 loader loading하면 끝. 설명 구찮음.

```bash
npm install --save-dev image-webpack-loader
```

![webpack5-image-opt](/images/webpack5-image-opt.png)





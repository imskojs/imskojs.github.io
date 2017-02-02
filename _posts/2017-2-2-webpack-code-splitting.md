---
layout: post
title: Webpack Code 나누기(Splitting).
published: true
---
이 글에서는 CSS Code splitting, Library Code Splitting, `require.ensure`를 사용한 Code Splitting을 다룬다.

Webpack의 기초 사용 용도는 여러 `<script>`를 하나의 `bundle.js`로 만드는것이다. 그러나 이렇게 하게되면 `bundle.js`는 엄청 커질수가 있다. 처음에는 이렇게 생각할수 있다. `bundle.js`가 커지더라도 서버에 요청 한번만 보내니 여러 script를 나누어 요청을 보내는것 보다 한번 요청을 보내어 `bundle.js`를 가지고 오는것이 빠를 것이라고. 그러나 그렇지가 않다. (TODO: max cachable size)

일단 브라우저는 여러 file들을 cache할수 있다. 그러나 만약 file size가 커지면 브라우저는 그 file을 cache하지 않는다. 그렇다면 website에 다시 들어 올때 마다 다시 그 script를 load해야 하는것이다.

또 한가지 여러 파일을 sync로 요청 하지 않고 parellel로 loading할수도 있다. (물론 이건 modern browser에서만 가능하다.) (TODO: which browser support parallel script requesting)

Code Splitting을 해야 하는 이유는 위 2가지말고 여러가지가 더 있다. 그중 Webpack이 해주는것 중에 하나가 on-demand asset loading이다.
Code Splitting을 하게되면 특정 코드들은 유저가 특정 행동을 할때 load할수 있는것이다. 미리 load를 하지 않아 initial load time이 줄어 드는것이다.

## CSS Code Splitting
Webpack을 `css-loader`를 사용하여 CSS를 JavaScript file에 import 할수 있다. 그리고 이 JavaScript file loading되는 HTML에 이 CSS가 반영 된다. (TODO: how css gets loaded)
JS와 함께 bundle되어 loading될시의 단점은 Browser가 기본 적으로 가지고 있는 asyncronouse loading과 parallel loading 기능을 못쓴다는 것이다.

Webpack은 CSS를 `extract-text-webpack-plugin` 과 `css-loader`를 사용하여 따로 bundling할수 있다.

예) [Webpack Doc 참조]

```js
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var path = require('path');

module.exports = function () {
    return {
        entry: './main.js',
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'bundle.js'
        },
        module: {
            rules: [{
                test: /\.css$/,
                exclude: /node_modules/,
                use: ExtractTextPlugin.extract({
                    loader: 'css-loader',
                    options: {
                      sourceMap: true
                    }
                })
            }]
        },
        devtool: 'source-map',
        plugins: [
            new ExtractTextPlugin({ filename: 'bundle.css', disable: false, allChunks: true })
        ]
    }
}
```

위 처럼 `module.rules.use`에 `ExtractTextPlugin.extract`를 사용하면 app에서 사용하는 모든 `.css`를 `plugins`의 `filename`에서 define한 `bundle.css` file로 bundle한다.
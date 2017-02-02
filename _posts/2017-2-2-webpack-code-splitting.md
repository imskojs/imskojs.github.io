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
            new ExtractTextPlugin({ 
              filename: 'bundle.css',
              disable: false,
              allChunks: true 
            })
        ]
    }
}
```

위 처럼 `module.rules.use`에 `ExtractTextPlugin.extract`를 사용하면 app에서 사용하는 모든 `.css`를 `plugins`의 `filename`에서 define한 `bundle.css` file로 bundle한다. (TODO: path for bundle.css)


## Library Code Splitting
보통 app은 많은 third party library를 사용한다. 이런 library code들은 전혀 자주 바뀌지 않는다. 우리가 쓰는 코드들만 자주 바뀔 뿐이다.

이렇게 자주 바뀌지 않는 코드를 궂이 자주 바뀌는 개발자의 code와 함께 bundling할 필요는 없는것이다. 왜 그러냐면 바뀌지 않는 코드는 cache를 할수 있기 때문이다. cache를 하면 서버에 따로 요청을 보내 새로 script를 가지고 올 필요가 없다. 

vendor file과 app file을 나누기 위해서 Webpack은 일단 2개의 file을 만든다. 그리고 이 2개의 file에서 공통된것들을 빼와 `CommonChunkPlugin` plugin을 사용하여 2개의 file에 공통적으로 있는 file을 빼와 새로운 file을 만든다.

여기서 중요한것은 Webpack은 build할때만다 새로운 hash를 file에 준다. 이렇게 되면 app이 만들어 질때마다 공통된 library를 만들지만 browser는 이것을 cache못한다. hash가 바뀌는 같은 file이 아니기 때문이다.

이것을 방지 하기위해 우리는 `CommonChunkPlugin` 에 공통적인 file을 bundling하는 `vender`와 runtime에 생성되는 hash를 manifest file에 저장 하면 된다. 이렇게 되면 3개의 file(`vender`, `main`, `manifest`)이 생기지만 우리가 필요한 `vendor`와 `main`만 사용하면 되는것이다.

예)

```js
var webpack = require('webpack');
var path = require('path');

module.exports = function(env) {
    return {
        entry: {
            main: './index.js',
            vendor: 'moment'
        },
        output: {
            filename: '[chunkhash].[name].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                // Specify the common bundle's name.
                names: ['vendor', 'manifest'] 
            })
        ]
    }
};
```




---
layout: post
title: Webpack Production and Multiple Environments
published: true
---

Webpack을 사용하여 개발했으면 이제 Webpack을 사용하여 production으로 코드를 바꿔보자.


## 바로 production 코드 나오게 하기.

간단하게 필수 production때 필수 적인것만 bundle하고 싶으면 아래 코드를 쓰자;

```bash
webpack -p
```

위 코드는 아래 코드를 쓴것과 1도 다르지 않다.

```bash
webpack --optimize-minimize --define process.env.NODE_ENV="'production'"
```

위 코드는;

1. `UglifyJsPlugin`을 사용하여 minify를 한다.
2. `LoaderOptionsPlugin`을 사용한다.
3. Node environment를 production으로 바꾼다.

단 PROCESS.env.NODE_ENV는 `webpack.config.js`에서의 build script에서 "production"으로 되어 있지 않다 그러니 `webpack.config.js`에서 이 NODE_ENV를 사용하지 말자.


## Webpack 여러 environment에서 사용할수 있게 하기.

여러 config가 있는데 이것들을 다른 environment마다 다르게 적용하려면 가장 쉬운 방법은 다른 environment마다 서로 다른 js file들을 적용 시키게 `webpack.config.js`를 설정 하는것이다.

아래는 `dev`용 config 파일 입니다.

```js
// dev.js

module.exports = function (env) {
  return {
    devtool: 'cheap-module-source-map',
    output: {
        path: path.join(__dirname, '/../dist/assets'),
        filename: '[name].bundle.js',
        publicPath: publicPath,
        sourceMapFilename: '[name].map'
    },
    devServer: {
        port: 7777,
        host: 'localhost',
        historyApiFallback: true,
        noInfo: false,
        stats: 'minimal',
        publicPath: publicPath
    }
  }
}
```

아래는 `prod` 용 config 파일 입니다.

```js
// prod.js

module.exports = function (env) {
  return {
    output: {
        path: path.join(__dirname, '/../dist/assets'),
        filename: '[name].bundle.js',
        publicPath: publicPath,
        sourceMapFilename: '[name].map'
    },
    plugins: [
        new webpack.LoaderOptionsPlugin({
            minimize: true,
            debug: false
        }),
        new webpack.optimize.UglifyJsPlugin({
            beautify: false,
            mangle: {
                screw_ie8: true,
                keep_fnames: true
            },
            compress: {
                screw_ie8: true
            },
            comments: false
        })
    ]
  }
}
``

아래 코드를 `webpack.config.js`에 써주세요.

```js
function buildConfig(env) {
  return require('./config/' + env + '.js')({ env: env })
}

module.exports = buildConfig;
```

그리고 `package.json`에서 build script를 만들자;
```js
{
  "scripts": {
    "build:dev": "webpack --env=dev --progress --profile --colors",
    "build:dist": "webpack --env=prod --progress --profile --colors",
  }
}
```

위 처럼 하는것도 좋지만 더 고급스러운 방법은 공통적으로 사용하는 코드를 하나의 file에 넣어두고 prod나 dev마다 다른 file로 update하는것도 base file을 업데이트 하는 더 좋은 방법도 있다.

*base.js*

```js
module.exports = function() {
  return {
    entry: {
      'polyfills': './src/polyfills.ts',
      'vendor': './src/vendor.ts',
      'main': './src/main.ts'

    },
    output: {
      path: path.join(__dirname, '/../dist/assets'),
      filename: '[name].bundle.js',
      publicPath: publicPath,
      sourceMapFilename: '[name].map'
    },
    resolve: {
      extensions: ['', '.ts', '.js', '.json'],
      modules: [path.join(__dirname, 'src'), 'node_modules']

    },
    module: {
      loaders: [{
        test: /\.ts$/,
        loaders: [
            'awesome-typescript-loader',
            'angular2-template-loader'
        ],
        exclude: [/\.(spec|e2e)\.ts$/]
      }, {
        test: /\.css$/,
        loaders: ['to-string-loader', 'css-loader']
      }, {
        test: /\.(jpg|png|gif)$/,
        loader: 'file-loader'
      }, {
        test: /\.(woff|woff2|eot|ttf|svg)$/,
        loader: 'url-loader?limit=100000'
      }],
    },
    plugins: [
      new ForkCheckerPlugin(),

      new webpack.optimize.CommonsChunkPlugin({
        name: ['polyfills', 'vendor'].reverse()
      }),
      new HtmlWebpackPlugin({
        template: 'src/index.html',
        chunksSortMode: 'dependency'
      })
    ],
  };
}
```

위 *base.js*를 `webpack-merge`로 merge하면 된다.

*prod.js*

```js
const webpackMerge = require('webpack-merge');

const commonConfig = require('./base.js');

module.exports = function(env) {
  return webpackMerge(commonConfig(), {
    plugins: [
      new webpack.LoaderOptionsPlugin({
        minimize: true,
        debug: false
      }),
      new webpack.DefinePlugin({
        'process.env': {
            'NODE_ENV': JSON.stringify('prod')
        }
      }),
      new webpack.optimize.UglifyJsPlugin({
        beautify: false,
        mangle: {
          screw_ie8: true,
          keep_fnames: true
        },
        compress: {
          screw_ie8: true
        },
        comments: false
      })
    ]
  })
}

```

이렇게 하면 `base.js`에 `prod.js`의 option들이 들어가게 됩니다.


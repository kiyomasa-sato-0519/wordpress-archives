---
title: "rails5.2とDockerとwebpackerとbootstrap4【設定編】"
date: "2019-03-03"
categories: 
  - "未分類"
tags: 
  - "docker"
  - "rails"
  - "rails5-2"
---

前回の[インストール編](https://www.null-engineer.com/2019/03/03/rails5-2%e3%81%a8docker%e3%81%a8webpacker%e3%81%a8%e3%80%90%e3%82%a4%e3%83%b3%e3%82%b9%e3%83%88%e3%83%bc%e3%83%ab%e7%b7%a8%e3%80%91/)の続き

最終的には `bin/webpack` を実行してcssファイルが出来上がり、 `public/packs/manifest.json` にcssファイルのパスが出ることがゴールである。

まずは `config/webpacker.yml` の 該当の値を変更する。  
これがまず大前提

```none
#extract_css: false
extract_css: true
```

ただしこれだけではcssは静的ファイルとして出力されない。

`.babelrc` が `webpacker.yml` と競合するので最低限に設定

```none
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "entry"
      }
    ]
  ]
}
```

次にrails/webpack標準の設定を見てみる。

**config/webpack/development.js**

```none
process.env.NODE_ENV = process.env.NODE_ENV || 'development'

const environment = require('./environment')

module.exports = environment.toWebpackConfig()
```

environmentをrequireしてそのままmodule.exportsしているだけである。

**config/webpack/environment.js**

そしてenvironment.jsは下記のようになっている。

```none
const { environment } = require('@rails/webpacker')

module.exports = environment
```

ここにpluginや設定を追加していく。

```none
$ yarn add -D babel-core \
  babel-polyfill \
  babel-preset-env \
  babel-loader@8  \
  clean-webpack-plugin \
  mini-css-extract-plugin \
  import-glob-loader \
  css-loader \
  sass-loader \
  webpack-merge
```

# 設定変更

**config/webpack/environment.js** を下記のように書き換える。

```none
const { environment } = require('@rails/webpacker')
const { resolve } = require('path')
const webpack = require('webpack')
const CleanWebpackPlugin = require('clean-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const rootPath = resolve(__dirname, '../..')
const pathsToClean = [ 'packs', 'packs-test' ]
const cleanOptions = {
  root: resolve(rootPath, 'public'),
  verbose: true,
}

// defaultのcss関連のloaderをまとめて消す
environment.loaders.delete('css')
environment.loaders.delete('sass')
environment.loaders.delete('moduleSass')
environment.loaders.delete('moduleCss')

environment.loaders.prepend('scss', {
  test: /\.(css|sass|scss)$/,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    'sass-loader',
    'import-glob-loader'
  ]
})

environment.plugins.prepend(
  'Provide',
  new webpack.ProvidePlugin({
    $: 'jquery/dist/jquery',
    jQuery: 'jquery/dist/jquery',
    Popper: 'popper.js/dist/popper'
  })
)

if (process.env.NODE_ENV !== 'test') {
  environment.plugins.insert('CleanWebpackPlugin', new CleanWebpackPlugin(pathsToClean, cleanOptions))
}

environment.plugins.insert('MiniCssExtractPlugin', new MiniCssExtractPlugin({filename: '[name]_[contentHash].css'}))

module.exports = environment
```

**config/webpack/development.js** を以下のようにする

```none
process.env.NODE_ENV = process.env.NODE_ENV || 'development'
const merge = require('webpack-merge')
const environment = require('./environment')
const config = environment.toWebpackConfig()

module.exports = merge(config, {
  mode: process.env.NODE_ENV,
  devtool: 'inline-source-map'
})
```

**config/webpack/production.js** も同じように書き換えるが `inline-source-map` は有効にしない

# 最後に

version依存などの関係でyarn install時にwarningが多発する場合、モジュールをアップデートしておくといいと思う。

```none
$ yarn upgrade
```

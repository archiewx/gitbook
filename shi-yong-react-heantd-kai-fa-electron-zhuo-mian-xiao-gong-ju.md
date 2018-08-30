---
categories:
  - name: electron
    img: 'https://'
description: electron和nw.js有的一拼，后来看到钉钉和ice都使用electron，甚至社区也在使用electron开发了atom游戏的编辑器
---

# 使用React和Antd来开发桌面小工具

![](/assets/syrhk/electron-logo.png)

electron和nw.js有的一拼，后来看到钉钉和ice都使用electron，甚至社区也在使用electron开发了atom游戏的编辑器，于是也来学习一波.

在electron提供一个nodejs和浏览器结合体，称之为类nodejs环境，拥有使用系统权限的能力，可以使用文件，执行系统命令等等，显示的内容的确是html，下面就看我们如何开发一款小引用，首先看文件结构

![](/assets/syrhk/1.png)

这是原来没有react的项目，其中有一个`render.js`和 `main.js`。在electron中，有两大进程，一个是渲染进程，一个是主线程，渲染进程可以说在浏览器环境，而主进程则是在nodejs环境下，当然两个进程可以通信，稍后都会设计这部分内容的。

有了react项目，我们项目得改变一下了。

项目目录:

```txt
├── README.md
├── app.icns
├── dist # 打包目录
│   └── electron
├── icons.png
├── index.html
├── logo.png
├── logo.psd
├── package.json
├── releases # 发行内容打包目录
├── src # 源码目录
│   ├── main # 主线程开发文件 
│   ├── renderer # 渲染进程开发文件
│   └── uploads
├── static # 静态资源文件
├── test # 测试文件
└── yarn.lock # yarn lock 文件
```

## webpack 配置

---

### render 配置

renderer线程配置和普通网页配置其实差不多，首先查看

```js
const path = require('path')
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const postcssFlexBugsFixs = require('postcss-flexbugs-fixes')
const autoprefixer = require('autoprefixer')
const isDev = require('./isDev')

const renderConfig = {
  mode: 'development',
  entry: {
    renderer: path.resolve(__dirname, '../src/renderer/main.js')
  },
  output: {
    path: path.resolve(__dirname, '../dist/electron'),
    filename: '[name].js',
    libraryTarget: 'commonjs2'
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json', '.ts', '.tsx'],
    alias: {
      'react-native': 'react-native-web',
      '@': path.resolve(__dirname, '../src/renderer')
    }
  },
  target: 'electron-renderer',
  module: {
    rules: [
      {
        test: /\.(js)|(jsx)$/,
        exclude: [path.resolve(__dirname, '../node_modules')],
        loader: 'babel-loader'
      },
      {
        test: /\.(jpe?g)|(png)|(gif)$/,
        loader: 'file-loader'
      },
      {
        test: /\.(css)|(scss)$/,
        use: [
          isDev() ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader?sourceMap',
          'sass-loader?souceMap',
          {
            loader: 'postcss-loader',
            options: {
              ident: 'postcss',
              plugins: () => [
                postcssFlexBugsFixs,
                autoprefixer({
                  browsers: ['>1%', 'last 4 versions', 'Firefox ESR', 'not ie < 9'],
                  flexbox: 'no-2009'
                })
              ]
            }
          }
        ]
      }
    ]
  },
  node: {
    __dirname: isDev(),
    __filename: isDev()
  },
  plugins: [
    new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: path.resolve(__dirname, '../src/renderer/templates/index.html'),
      minify: {
        collapseWhitespace: true,
        removeAttributeQuotes: true,
        removeComments: true
      },
      nodeModules:
        process.env.NODE_ENV !== 'production' ? path.resolve(__dirname, '../node_modules') : false
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin()
  ]
}

if (!isDev()) {
  renderConfig.mode = 'production'
  renderConfig.plugins.push(
    new MiniCssExtractPlugin({
      filename: '[name].[hash].css',
      chunkFilename: '[id].[hash].css'
    }),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"production"'
    }),
    new webpack.LoaderOptionsPlugin({
      minimize: true
    })
  )
}

module.exports = exports = renderConfig
```

我们配置中正常配置了webpack 的配置的mode(webpack4), resove, module,plugins

### 

### main 配置

## 开发脚本dev-runner.js

---

```js
/*
 * @Author: zhenglfsir@gmail.com
 * @Date: 2018-08-16 16:03:51
 * @Last Modified by: zhenglfsir@gmail.com
 * @Last Modified time: 2018-08-29 14:59:03
 */
const path = require('path')
const webpack = require('webpack')
const isDev = require('./isDev')

const useEslint = true

const mainConfig = {
  mode: 'development',
  entry: {
    main: path.resolve(__dirname, '../src/main/main.js')
  },
  output: {
    libraryTarget: 'commonjs2',
    filename: '[name].js',
    path: path.join(__dirname, '../dist/electron')
  },
  externals: [],
  resolve: {
    extensions: ['.js', '.json']
  },
  target: 'electron-main',
  module: {
    rules: (useEslint
      ? [
          {
            test: /\.(js)$/,
            enforce: 'pre',
            exclude: /node_modules/
          }
        ]
      : []
    ).concat([
      {
        test: /\.(js)$/,
        loader: 'babel-loader',
        exclude: /node_modules/
      }
    ])
  },
  plugins: [new webpack.NoEmitOnErrorsPlugin()]
}

if (isDev()) {
  mainConfig.plugins.push(
    new webpack.DefinePlugin({
      __static: `${path.join(__dirname, '../static')}`.replace(/\\/g, '\\\\')
    })
  )
} else {
  mainConfig.mode = 'production'
}

module.exports = exports = mainConfig

```

## 

## 编辑脚本build.js

---

```js
/*
 * @Author: zhenglfsir@gmail.com
 * @Date: 2018-08-29 19:53:38
 * @Last Modified by: zhenglfsir@gmail.com
 * @Last Modified time: 2018-08-30 10:46:23
 */
process.env.NODE_ENV = 'production'

// const childProcess = require('child_process')
const webpack = require('webpack')
const chalk = require('chalk')
const Listr = require('listr')
const execa = require('execa')
const del = require('del')
const path = require('path')

const mainConfig = require('./webpack.config.main')
const rendererConfig = require('./webpack.config.renderer')
const packagerConfig = require('./packager.config')

const doneLog = chalk.bgGreen.white(' DONE ') + ' '
const errorLog = chalk.bgRed.white(' ERROR ') + ' '
const okayLog = chalk.bgBlue.white(' OKAY ') + ' '

const pack = function pack(config) {
  return new Promise((resolve, reject) => {
    webpack(config, (err, stats) => {
      if (err) {
        return reject(err)
      } else if (stats.hasErrors()) {
        let errors = ''
        stats
          .toString({
            chunks: false,
            colors: true,
          })
          .split(/\r?\n/)
          .forEach(line => {
            err += `    ${line}\n`
          })

        reject(err)
      } else {
        resolve(
          stats.toString({
            chunks: false,
            colors: true,
          })
        )
      }
    })
  })
}

const clean = args => del(args)

const packagerApp = () => {
  return new Promise((resolve, reject) => {
    packager(packagerConfig, (err, appPaths) => {
      if (err) {
        console.log(`\n${errorLog}${chalk.yellow('`electron-packager`')} says...\n`)
        console.log(err + '\n')
        reject(err)
      } else {
        console.log(`\n${doneLog}\n`)
        resolve()
      }
    })
  })
}

const build = function build() {
  const taskList = [
    {
      title: 'Clean Build Directory',
      task: () => clean([path.resolve(__dirname, '../dist')]),
    },
    {
      title: 'Renderer Build',
      task: () => {
        return pack(rendererConfig).then(result => {
          result += result + '\n\n'
          console.log('\n\n' + result)
        })
      },
    },
    {
      title: 'Main Build',
      task: () => {
        return pack(mainConfig).then(result => {
          result += result + '\n\n'
          console.log('\n\n' + result)
        })
      },
    }
  ]
  const tasks = new Listr(taskList)
  tasks.run().catch(err => {
    console.error(chalk.red(err))
  })
}

build()

```




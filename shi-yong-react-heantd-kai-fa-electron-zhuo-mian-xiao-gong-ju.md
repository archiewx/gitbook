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

我们配置中正常配置了webpack 的配置的mode\(webpack4\), resove, module,plugins

### 

### main 配置

---

```js
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

main 开发脚本配置配置需要target修改为 `electron-main`以外， 还需要增加babel-loader让我们的main线程也支持es6语法。
接下来就开发我们dev-runner脚本，这个脚本为运行脚本，运行webpack 提供的API

## 开发脚本

### 导入依赖

```js
const chalk = require('chalk') // 为了让打印的有颜色
const electron = require('electron') // 导入electron
const path = require('path')
const childProcess = require('child_process') // 运行命令
const webpack = require('webpack')
const webpackHotMiddleware = require('webpack-hot-middleware') // webpack中间件
const WebpackDevServer = require('webpack-dev-server') // webpack开发服务器

const mainConfig = require('./webpack.config.main') // main基本配置 
const rendererConfig = require('./webpack.config.renderer') // renderer配置

```

### 增加打印stat函数

```js
const logStats = function logStats(proc, data) {
  let log = ''

  log += chalk.yellow.bold(`┏ ${proc} Process ${new Array(19 - proc.length + 1).join('-')}`)
  log += '\n\n'

  if (typeof data === 'object') {
    data
      .toString({
        colors: true,
        chunks: false,
      })
      .split(/\r?\n/)
      .forEach(line => {
        log += '  ' + line + '\n'
      })
  } else {
    log += `  ${data}\n`
  }

  log += '\n' + chalk.yellow.bold(`┗ ${new Array(28 + 1).join('-')}`) + '\n'

  console.log(log)
}

```

stat内容就是webpack 增加 --progress 的内容

### 增加开始renderer线程打包的脚本


```js
const startRenderer = function startRenderer() {
  return new Promise(resolve => {
    rendererConfig.entry = [path.join(__dirname, 'dev-client')].concat(
      rendererConfig.entry.renderer
    )
    const complier = webpack(rendererConfig)
    hotMiddleware = webpackHotMiddleware(complier, {
      log: false,
      heartbeat: 2500,
    })
    complier.hooks.compilation.tap('compilation', compilation => {
      compilation.hooks.htmlWebpackPluginAfterEmit.tap(
        'html-webpack-plugin-after-emit',
        (data, cb) => {
          hotMiddleware.publish({ action: 'reload' })
        }
      )
    })
    complier.hooks.done.tap('done', stats => {
      logStats('Renderer', stats)
    })
    const webServer = new WebpackDevServer(complier, {
      contentBase: path.join(__dirname, '../'),
      quiet: true,
      before: (app, ctx) => {
        app.use(hotMiddleware)
        ctx.middleware.waitUntilValid(() => {
          resolve()
        })
      },
    })
    webServer.listen(9080)
  })
}

```
做的工作: 

* 增加entry，将 `dev-client.js`增加到renderConfig的 entry中
* 使用webpack 读取配置文件生成complier对象
* 将热更新添加到 complier对象中
* 监听 complier 对象的compilation 钩子和html-webpack-plugin的after-emit钩子，并告诉热更改中间件，修改html后reload 浏览器
* 监听Renderer打包完成后，打印 stat
* 最后启动webServer, 并将热更新中间件加入到devServer中
* 监听9080端口

### 增加Main 线程启动你那个脚本

```js
const startMain = function startMain() {
  return new Promise(resolve => {
    mainConfig.entry = [path.join(__dirname, '../src/main/main.dev.js')].concat(
      mainConfig.entry.main
    )

    const complier = webpack(mainConfig)

    complier.hooks.watchRun.tapAsync('watch-run', (compilation, done) => {
      hotMiddleware.publish({ action: 'compiling' })
      done()
    })
    complier.watch({}, (err, stats) => {
      if (err) {
        console.log(err)
        return
      }
      logStats('Main', stats)

      if (electronProcess && electronProcess.kill) {
        process.kill(electronProcess.pid)
        manualRestart = true
        electronProcess = null
        startElectron()
        setTimeout(() => {
          manualRestart = false
        }, 5000)
      }
      resolve()
    })
  })
}

```

## 构建脚本build.js

---

```js
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




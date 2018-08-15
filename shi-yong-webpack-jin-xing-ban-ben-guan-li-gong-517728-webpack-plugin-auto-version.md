---

categories:
  - name: webpack 
    img: https://cdn-cos.luoyangfu.com/2018-07-30/cate/webpack-logo.png
  - name: javascript
    img: https://cdn-cos.luoyangfu.com/2018-07-30/cate/JavaScript-logo.png
title: 使用webpack进行版本管理工具(webpack-plugin-auto-version)

---

# 使用webpack进行版本管理工具(webpack-plugin-auto-version)

- GitHub [webpack-plugin-auto-version] https://github.com/zsirfs/webpack-plugin-auto-version
- npm [webpack-plugin-auto-version] https://www.npmjs.com/package/webpack-plugin-auto-version
- issue https://github.com/zsirfs/webpack-plugin-auto-version/issues

[![](https://img.shields.io/npm/v/webpack-plugin-auto-version.svg?style=flat-square)](https://www.npmjs.com/package/webpack-plugin-auto-version)
[![](https://img.shields.io/github/commit-activity/y/zsirfs/webpack-plugin-auto-version.svg?style=flat-square)](https://www.npmjs.com/package/webpack-plugin-auto-version)
[![](https://img.shields.io/github/last-commit/zsirfs/webpack-plugin-auto-version/master.svg?style=flat-square)](https://github.com/zsirfs/webpack-plugin-auto-version)
[![](https://img.shields.io/npm/l/webpack-plugin-auto-version.svg?style=flat-square)](https://github.com/zsirfs/webpack-plugin-auto-version)
[![](https://img.shields.io/github/commit-activity/y/zsirfs/webpack-plugin-auto-version.svg?style=flat-square)](https://github.com/zsirfs/webpack-plugin-auto-version)

距离1.2.3版本以及好几个月了，当时是因为在前端开发中经常会因为缓冲问题，导致最新版本和线上版本或者测试版本出现不同，bug得不到及时有效的修复，手动修改版本可以，但是费时费力，容易出错，设计到动态加载的位置，动态创建脚本的时候就会出错，所以有效的版本管理工具就是能够有效的利用version，自动的根据package.json中version更换major,minor,patch版本。看效果图:
![](media/15326555617513/15326711404526.jpg)


## 插件提供配置项

```js
{
    // 文件名替换标记 [version] -> v1.2.2
      filenameMark: options.filenameMark,
      // 版权名称
      copyright: options.copyright || '[webpack-plugin-auto-version]',
      // 保存的时候格式化package.json的indent
      space: options.space || 2,
      // 是否自动清理老版本
      cleanup: options.cleanup || false,
      // 是否检测资源内的标签
      inspectContent: options.inspectContent || !!options.template,
      // 自定义资源内版本替换模板 [VERSION]version[/VERSION]
      template: options.template || `[${this.copyright}]version[/${this.copyright}]`,
      // 自定义忽略后缀，默认是['.html']忽略html文件打入版本文件夹
      ignoreSuffix: [], // 忽略的后缀或者文件关键词
      isAsyncJs: false,
      htmlTempSuffix: ['.html', '.vm', '.ejs', '.handlbars']
}
```

- filename: 文件名称标记
- copyright: 版权声明会出现在js，css，html文件头部
![](media/15326555617513/15326713793121.jpg)
- space: 版本回存到package.json中的时候格式化间距默认为2字符(之后版本会默认读取.editconfig)
- cleanup: 自动清除
![](media/15326555617513/15326715774000.jpg)
存在多个版本的时候可以设置为`true`，自动清除比当前版本低的文件夹
- inspectContent: 检测代码内是否有可替换为版本的模板，看设置option.template
- template: 替换的模板，默认是[webpack-plugin-auto-version]version[/webpack-plugin-auto-version]
- ignoreSuffix: 配置忽略后缀的文件存放到编译根目录下
- isAsyncJs: 配置是否是动态创建script的，动态创建script如果是更具publicPath则可用，会在publicPath加入版本号
- htmlTempSuffix: 按照html语法的后缀，默认是: html, ejs, handlebars,vm 文件等

## 插件使用方法

### 安装

npm/yarn:

```bash
npm i -D webpack-plugin-auto-version # yarn add -D webpack-plugin-auto-version
```

配置webpack.config.prod.js

```js
const WebpackPluginAutoVersion = require('webpack-plugin-auto-version')

module.exports = {
    // ...忽略其他配置
    plugins: [
        // ...其他插件
        new WebpackPluginAutoVersion()
    ]
    
}

```

## webpack 插件开发

使用webpack插件开发的话需要一个函数对象，函数对象对外暴露`apply`方法，apply参数有webpack的编译对象`complier`,其中包含webpack的配置项，编译支援，编译分块，还有webpack的钩子以及其他插件暴露的钩子函数。

看下面代码:

```js
if (complier.hooks) {
      // 兼容webpack ^4.0.0
      complier.hooks.compilation.tap('WebpackAutoVersionPlugin', (compliation) => {
        compliation.hooks.htmlWebpackPluginBeforeHtmlGeneration &&
          compliation.hooks.htmlWebpackPluginBeforeHtmlGeneration.tapAsync(
            'WebpackAutoVersionPlugin',
            htmlWebpackPluginBeforeHtmlGeneration(this)
          )
        compliation.hooks.htmlWebpackPluginAlterAssetTags &&
          compliation.hooks.htmlWebpackPluginAlterAssetTags.tapAsync(
            'WebpackAutoVersionPlugin',
            htmlWebpackPluginAlterAssetTags(this)
          )
        compliation.hooks.htmlWebpackPluginAfterHtmlProcessing &&
          compliation.hooks.htmlWebpackPluginAfterHtmlProcessing.tapAsync(
            'WebpackAutoVersionPlugin',
            htmlWebpackPluginAfterHtmlProcessing(this)
          )
      })
    } else {
      // 如果存在html-webpack-plugin 则监听
      complier.plugin('compilation', (compilation) => {
        compilation.plugin(
          'html-webpack-plugin-before-html-generation',
          htmlWebpackPluginBeforeHtmlGeneration(this)
        )
        compilation.plugin(
          'html-webpack-plugin-alter-asset-tags',
          htmlWebpackPluginAlterAssetTags(this)
        )
        compilation.plugin(
          'html-webpack-plugin-after-html-processing',
          htmlWebpackPluginAfterHtmlProcessing(this)
        )
      })
    }
```
这里对`html-webpack-plugin`的钩子函数监听，配合html-webpack-plugin继续网页资源的构造,同时兼容webpack4.x || webpack3.x两种版本。


详细代码查看: [L161](https://github.com/zsirfs/webpack-plugin-auto-version/blob/master/src/index.js#L161)

GitHub仓库: [https://github.com/zsirfs/webpack-plugin-auto-version](https://github.com/zsirfs/webpack-plugin-auto-version) 欢迎star，fork

问题反馈: [issues](https://github.com/zsirfs/webpack-plugin-auto-version/issues)


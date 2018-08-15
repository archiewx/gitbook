---

categories:
    - name: tool
      img: https://cdn-cos.luoyangfu.com/2018-08-13/cate/iterm-logo.jpg
title: 使用Nvm，在vscode继承终端警告:[nvm is not compatible with the npm config "prefix" option]

---


# 使用Nvm，在vscode继承终端警告:[nvm is not compatible with the npm config "prefix" option]

报错图:
![](media/15331737075301/15331737928285.jpg)

原因:
![](media/15331737075301/15331739147599.jpg)

地址: https://github.com/Microsoft/vscode-docs/blob/master/docs/editor/integrated-terminal.md#why-is-nvm-complaining-about-a-prefix-option-when-the-integrated-terminal-is-launched

意思说:

 - 系统中存在多个node版本，npm 是其中某一个安装的(反正不是现在使用的), npm执行命令存在$PATH 路径中，例如/usr/local/bin, /usr/bin之类的。
 - 在开发工具中得到这些路径，vscode会在启动的时候登录bash shell.我使用的事zsh，意味着集成终端启动的时候我们的bash配置文件.bash_profile已经运行，启动一个新的终端的时候回打印文件

vscode给了一个解决方案: 
![](media/15331737075301/15331745566580.jpg)
首先找到用户路径的npm，

```bash
$ ls -la /usr/local/bin | grep npm
```

然后得到下面

```bash
...
...  npm -> ../lib/node_modules/npm/bin/npm-cli.js
...
```

npm是系统某个路径的链接。
然后移除这个脚本

```bash
rm -R /usr/local/bin/npm /usr/local/lib/node_modules/npm/bin/npm-cli.js
```

在查看issue中我发现了另外一个方案，不过也差不多, 使用nvm 的话就不需要在系统中安装node了,版本全部使用nvm调制，但是我们使用yarn的话，使用brew安装会依赖node，所以直接写在node 又会出现下面问题:

![](media/15331737075301/15331747855565.jpg)

我们首先使用:

```bash
brew uninstall yarn --force
```
--force 原因是卸载掉所有yarn 版本，可能升级前的版本也存在

然后要先卸载这些全局的小东西。不然就会遗留到后面的环境中
![](media/15331737075301/15331751403293.jpg)

```bash
brew uninstall node --force # 同理
```

继续:

```bash
brew install yarn --without-node # 安装不带node了
```

但是打开的时候还是存在警告，这个时候就需要清盘了。

这样又可以得到一个干净整洁的nodejs的环境了。☺



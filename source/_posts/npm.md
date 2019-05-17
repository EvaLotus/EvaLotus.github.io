---
title: npm常用命令一网打尽
date: 2018-11-30 18:02:09
tags: Tool
---

npm笔记，纯粹记录下，哪天电脑坏了快速重装一遍不走弯路。
<!-- more -->
####  nvm 管理 node 版本

首先先把全局安装的 packages 记录下来

```bash
# 列出所有全局安装 package
npm ls -g --depth=0
```

/Users/eva/.nvm/versions/node/v10.15.3/lib
├── @vue/cli@3.5.2
├── eslint@5.15.3
├── express@4.16.4
├── gulp-cli@2.1.0
├── hexo-cli@1.1.0
├── npm@6.9.0
├── nrm@1.1.0
├── webpack@4.29.6
├── webpack-cli@3.3.0
└── yo@2.0.5

然后把原来全局安装的 node 删掉，反正以后也不用了

```bash
# 删除全局安装的 package
sudo rm -rf /usr/local/lib/node_modules
# 删除 node
sudo rm /usr/local/bin/node
# 删除全局 node 模块注册的软链
cd  /usr/local/bin && ls -l | grep "../lib/node_modules/" | awk '{print $9}'| xargs rm
```

安装 nvm

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
```

修改环境变量

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

查看手册

```bash
# 列出所有命令
nvm -h
# 安装最新Long-term-support版
nvm install --lts
# 比如10.5.3
nvm use 10.5.3
```

LTS是基础库的开发者对库的使用者的一个承诺，保证某个版本的库发布之后的很长一段事件之内都得到支持。如果此版本发现一些紧急问题需要修复，那么就会在这个版本上进行更新。通常这些问题的修复都不会导致 API 变化（API 保证长期兼容），所以版本号的前两位是不变的，通常只变化第三位。

#### 安装全局 package

```bash
npm i -g @vue/cli eslint express nrm webpack webpack-cli gulp
# 自己安装自己来更新 npm
npm i -g npm
```

一般不要随便切换 node 版本，如果要切换，记得把全局安装的 packge 重新安装在那个 node 版本下。

```bash
nvm install v12.0.0 --reinstall-packages-from=8.0.0
```

#### nrm 管理 npm 源

```bash
nrm ls
nrm use cnpm
```

  npm ---- https://registry.npmjs.org/

\* cnpm --- http://r.cnpmjs.org/
taobao - https://registry.npm.taobao.org/
nj ----- https://registry.nodejitsu.com/
npmMirror  https://skimdb.npmjs.com/registry/
edunpm - http://registry.enpmjs.org/

这样就可以方便的管理 npm 的源。

#### npm私服

如果公司内部搭建了私有的 npm，也使用 nrm 也可以更好切换和管理。

比如我有个 npm 私服

```bash
nrm add eva http://registry.npm.evacoder.com/ 
npm use eva
```

指定好源之后，还是用 npm 命令，而不用使用 cnpm 等命令。

私服不会全量同步 npm 包，有时候找不到时需要手动同步下 package。

```
curl -X PUT http://npm.evacoder.com/sync/[package] 
# 或者浏览器访问 http://npm.evacoder.com/sync/[package]
```

#### 发布自己的 npm package

其实 npm package 就是一个自己写的 js export 出来而已。发布一个 package 只需要

1.写代码，2.注册 npm 帐号(私服就addUser)，3.发布

```bash
# 注册登录
npm login
# 填写 username，password，email
# 新建项目
npm init --yes
```

package.json

```json
{
  "name": "@eva/hello-world",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
  	"test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

代码

```js
module.exports = 'hello Eva';
```

发布

```bash
# 发布
npm publish
```

使用

```bash
cd demo
# -S是保存到 package.json中
# -D 是指是 devDependency，只是工具，不会打包到代码中去的
npm i -S -D @eva/hello-world
node -e 'console.log(require("@eva/hello-world"))'
```

具体的 编写高质量npm package 的详细内容将在 sdk 编写心得中展开来说
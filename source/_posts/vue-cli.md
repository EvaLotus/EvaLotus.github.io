---
title: 从vue-cli说到webpack浪漫史
date: 2018-11-18 17:29:06
tags: webpack
---

（啊！这篇文章写了快两个月了，涉及的东西太多了，逃~）

#### 安利

虽然现在已经是19年了，还不懂webpack，es6吗？可能每次的借口就是，没有实践的机会。

那么现在的我真的建议你，**使用element-ui开始写你下一个后台管理系统吧**。

但是一定要谨记一个开发的原则：**不拿外部有影响力的项目来练手，而是先在内部系统中进行有趣的尝试，成熟踩坑之后再用在外部项目中**。

这篇文章可能没那么高大上，如果你对webpack已经烂熟于心，也可以看着玩玩，权当回顾webpack的历史，也可以帮我指出些问题。
<!-- more -->

element-ui有多好用呢？其实还是搞安全的后端同学推荐给我的，你就能知道，完全不懂前端的人都能玩转。

如果在看这篇文章的你是一个不懂前端的后端同学，也可以试试这个框架，以后开发后台管理系统不用再求人啦。

element-ui 有完善的 [api文档](http://element-cn.eleme.io/#/zh-CN/component/installation)，比bootstrap复制粘贴起来还畅快，而且使用 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin) 更是分分钟能搭建一个后台管理系统。

等你搭积木似的完成一个项目，以上那些新技术（vue，webpack，es6，nodeJS，npm）对你来说就不再陌生了（虽然不完全懂原理，但是至少算是碰过了）。

vue的好，在于它是一个**渐进式**的丰俭由人的框架，可以直接html里引入vue.js开箱即用写起来，也可以使用vue-cli提供的一整套开发流程。使用了vue-cli，我们就可以轻轻松松窥见**现代的先进的前端开发项目**是怎么样的。

这里是vue-cli的 [文档](https://cli.vuejs.org/guide/)

```bash
# 安装
npm i -g @vue/cli
# 新建项目
vue create hello-world
# 选择默认的配置即可，安装好依赖之后，直接进入目录
# dev模式运行起项目
npm run serve
# 打包项目发布
npm run build
```

#### vue-cli做了什么？

what，因为拖得太久，vue-cli现在都升级成3.0的@vue/cli把build文件直接去掉了，这篇文章就是为了介绍vue-cli的打包配置的，以后都看不到webpack的打包配置源代码了，这怎么学习哦。

先抛开 vue 不谈，我们先思考一下最简单的完整前端项目可能会有哪些工程化方面需求。

##### js 方面：

1.eslint 代码规范，不以规矩不成方圆。

2.通过 babel 将ES6+转为大多数浏览器可识别的ES5。

3.文件拼接，如果是 ES5写的可能需要将多个js文件concat起来，外面套个

```js
(function(){
  // here is the content
})();
```

来通过IIFE自执行匿名函数来隔离作用域。

如果是 ES6+写的，babel 转码也转不了`import 'a.js'`，后面再说怎么办。

4.js 压缩，为了性能优化，加快加载时间，节省用户流量。

##### css 方面：

1.scss/less 转为 css。

2.自动处理浏览器兼容性，加前缀，autoprefixer。

3.压缩 css

这些需求并不过分吧，都是很常见的。

其实vue-cli的webpack配置也主要实现了以下功能：

1.使用vue-loader将vue文件转为js文件

2.使用babel-loader将es6转es5

3.帮忙本地起一个node server来运行代码

4.使用eslint来规范我们的代码

5.hot module replacement\(HMR\)可以做到在应用运行时无需刷新页面即可替换模块来热更新。简言之，每次写完代码按ctrl+s不用手动刷新，浏览器自动就更新了。

6.production环境和dev环境的切换

7.production环境帮忙压缩js，把用到的第三方库抽离出来生成vendor.js（动静分离，不常修改的抽离出来），为打包出来的js生成一个hash，用来防止缓存。

8.css-loader，style-loader，sass-loader，将scss/less等转成浏览器可解析的css，postcss-loader给 css 加前缀autoprefixer保证兼容性，

回顾一下你平常的前端项目，通过webpack的工程化配置，是不是把你想做的都帮你做好了。

封装的越来越简单，看不到实现细节，以后前端真的变成搬砖工种啦。

> 关于修改，Evan You 是这么说的
>
> 1.vue-cli@3.0修改的方向是逐步成为“config/script in a package”的模型
>
> 2.为避免使用户预先做一些不可逆的设置，
>
> -我们将逐步移除browserify支持，webpack将只基于一个template
>
> -在预设时就可以配置常见的（pwa/ts/ssr）设置
>
> 3.除了封装包，提供通过vue.config.js配置的可能
>
> -为高级功能 如 env variables, css extraction API proxying提供了一个集中的入口
>
> -如本地预设一样，提供了底层的webpack设置支持，可能是webpackchain

#### 前端工程化历史

我们先忽略上文中的那些loaders，只看需求和目标，是不是每个前端项目总会有这些需求。

你可能觉得没啥了不起的，即使没有webpack，我们也有各种scss转css，压缩文件等需求，比如css加前缀保证浏览器兼容性，保存文件自动刷新浏览器，js文件拼接。

我们的项目现在还存在为了打包拼接文件写的**shell**脚本，再通过YUI Compress来压缩。

有些工程化需求也可以使用**sublime插件**来完成，比如：

1.保存文件自动刷新浏览器（livereload插件，需要配合chrome插件livereload来完成，监控ctrl+s就无脑刷新浏览器，或者其他配置）

2.css加前缀保证浏览器兼容性（autoprefixer插件）

3.scss转css（sass插件）

通过编辑器来工程化一点儿也不靠谱，你没办法要求每个人都和你安装一样的环境，一样的版本，完全不可控制。 

用shell脚本同样也一点儿都不cool，就不能有前端自己的构建工具吗？

后来**grunt/gulp**应运而生。

有了这些构建工具，这些构建任务都跟着代码项目走，写好gulpfile，每个人clone下来，只需`npm install`就可以保证环境一致，零碎的打包工作也可以精简成一句`gulp`即可完成。

其实也能满足我的大部分需求了，但是后来我觉得es6写的爽啊，想要使用es6来开发。

没有import/export还好说，这种难道还用gulp-concat吗？no，no，no，这个时候我们的webpack就闪亮登场了。

`npm run build`，写好一次webpack.conf.js之后再也不用每次手动操作，也不需要自己想办法去写shell脚本。

最开始因为webpack的文档不太平易近人，而望而却步，但是通过阮老师的 [webpack-demos](https://github.com/ruanyf/webpack-demos) 很清晰的就能了解到webpack使用进阶，以及一些常用的plugins，非常推荐，有使用基础的同学大概使用十分钟就能看完。

#### webpack和gulp的区别

#### 项目定位

本质上简言之，Gulp是构建工具，webpack是模块化解决方案。

Gulp自己的slogan是The streaming build system，定位是流式构建系统，写好一个个任务（concat，minify，rename等等），通过pipe语法可以自定义构建顺序。

Webpack的介绍是webpack is a module bundler. Its main purpose is to bundle JavaScript files for usage in a browser, yet it is also capable of transforming, bundling, or packaging just about any resource or asset.

webpack的初衷是用来打包js（typescript，ES6等）成浏览器能识别的js，现在可以打包任何资源（css，images）。但是现在webpack有了更多的插件，也可以处理gulp中的大部分任务。

#### 使用现状

现在两者都可以用于前端构建，github上webpack的stars更多一点。

因为大多数项目通过npm scripts + webpack，直接写好npm scripts，其他开发者不需要关心细节，只需要`npm run serve`就可以完成基本需求，如果无法满足还有其他自定义需求，你可以再叠加gulp来完成构建。

gulp是构建工具，和grunt是一个level，grunt的效率比较低，大部分被gulp给取代了。

webpack是模块化方案，和browserify是一个level。

场景实例

比如我需要将一些ES6写的单文件打包并压缩混淆成一个js

最开始使用gulp

```js
const gulp = require('gulp');
const babel = require('gulp-babel');
const concat = require('gulp-concat');
const uglify = require('gulp-uglify');
const pump = require('pump');
const rename = require('gulp-rename');

gulp.task('default', () => {
  pump([
    gulp.src(['src/js/*.js']),
    babel({
      presets: ['@babel/env'],
    }),
    concat('sdk.min.js'),
    gulp.dest('dist'),
    rename({ suffix: '.min' }),
    uglify({
      compress: {
        ie8: true,
      },
      mangle: {
        ie8: true,
      },
      output: {
        ie8: true,
      },
    }),
    gulp.dest('pack'),
  ]);
});
```

当遇到ES6中的import时你会发现仅仅使用gulp无法解决我们的需求

```
import 'a'
```

babel转码之后依然保留了上面的原文。

浏览器还是无法识别这样的语法，而且已经有了import的引入关系，我们不应该再使用gulp-concat这样简单粗暴的拼接方法了，因为ES6里都是文件作用域，并不需要使用IFFE这样的自执行匿名函数或者命名空间来保证作用域的互相影响。

这个时候webpack就可以很简单的满足我们的需求。

```js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/js/index.js',
  output: {
    path: path.resolve(__dirname, 'pack/'),
    filename: 'bundle.js',
  },
  module: {
    rules: [{
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env'],
        },
      },
    }],
  },
};
```

只需要简单一句的`webpack`就可以完成所有的打包构建。

而且gulp需要自己列出哪些文件被引用了，但是webpack会根据import来自动理清文件的依赖关系更符合实际需求。

所以说前端发展的实在是太快了，好多人喊着学不动了，但是架不住确实好用啊，一次配置，终生受益。

这篇文章已经很长了，webpack 的具体使用请参见下一篇，webpack 的使用优化。


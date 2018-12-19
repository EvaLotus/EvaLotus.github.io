---
title: hexo+github pages 搭建 blog及防坑指南
date: 2016-12-18 12:57:11
tags: blog
---

在超哥的安利下，开始使用hexo+githubpages搭建自己的博客。对于前端同学来说，基于Node.js的hexo可谓是非常友好的选择。

前两天超哥很开心的敲下`hexo d`，过了两天整理文件时一不小心手贱删掉了项目，写了4年的博客全都没有啦，只能一个个copy出来，简直是喜闻乐见（超哥不要打我，逃～）

<!-- more -->
基本操作大家都可以在hexo详尽的文档中找到，但是一些文档之外的最佳实践，在此记录下。

#### 基本搭建操作

以下是搭建hexo博客基本操作，只是mark一下

```bash
# 安装hexo命令行工具
nmp i hexo-cli -g

# 在此目录搭建博客，在github创建此项目
cd EvaLotus.github.io
# 新搭建的blog需要这一行
hexo init
# 安装依赖
npm install

# 启动服务，一个基本的博客就已经搭建好了
hexo s

# 选择喜欢的主题，如next，实在是太容易撞衫了，但是确实是极简主义的爱
# 将主题clone至themes目录下
# 每次换电脑重新clone项目都需要执行这一步哦
git clone https://github.com/theme-next/hexo-theme-next themes/next

# 在外层_confif.yml中改theme为next

# 删除generate的文件，切换主题时最好执行下，可以不用自己删掉public文件
hexo clean****

# 生成public目录，可以在本地调试时使用，注意看下.gitignore可以发现git提交时并不提交public目录
hexo g

# 如果只是直接部署
hexo d

# 新建一篇博客
hexo new 'HelloWorld'
```
很容易就搭建好了自己的博客，下面是一些常见问题的解决。
#### FAQ

**1.怎么能将github.io的域名改成自己申请的域名？**

首先你需要在万网或其他平台上买个域名，一般50元/年。购买完成后需要在购买域名的管理后台进行CNAME解析，目的是将gitubpages的地址比如`https://github.com/EvaLotus/EvaLotus.github.io`指向你购买的域名`evacoder.com`。
然后直接在github的project settings里，设置custom domain，这样就会自动在你的项目中增加一个CNAME文件，里面会记录你自己的域名。

**2.如何`hexo d`自动更新部署博客**

需要安装插件`npm i hexo-deployer-git --save`，在`_config.yml`中配置

```bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
type: git
repo: git@github.com:EvaLotus/EvaLotus.github.io.git
branch: master
```
配置完成之后，每次`hexo d`都会自动将编译好的文件提交到master分支上，并部署完成。

还需要在配置\_config.yml时一定要注意此类文件对缩进要求很严格，在sublime中编辑会有适当的颜色提醒你是否缩进正确。

3.每次`hexo d`之后我的CNAME都没了怎么办？

注意看下`.gitignore`,git提交时并不提交public目录

小伙伴们应该都发现了，`hexo d`之后并没有push source目录，只是push生成后的public中的文件，所以我们要注意，CNAME，README，Favicon.ico都应该放在source目录下，这样`hexo d`之后会直接放在public的根目录下

4.换电脑了或者像超哥那样误删了源文件怎么办？

很多小伙伴应该都有同时在公司和在家修改博客的需求，一换电脑就懵逼，github上新clone下来的只有编译后的文件。

或者很久不写博客，整理文件时一不小心把源文件删掉了。

其实我们可以使用**分支来管理我们的博客**，简言之，master上存放生成的静态文件，hexo存放源文件。

操作上并不需要切换分支，按下面的步骤配置之后，每次操作都在hexo分支上修改，不用再管master分支了。

```bash
# 切出hexo分支
git checkout -b hexo

# 以后都在此分支上修改，虽然hexo d可以帮你将生成的静态文件自动push到master
git checkout master
hexo d

# 但是每次修改之后一定要记得自己把hexo分支上的源文件的修改push一下！！
git push origin/hexo

# Easy!
# 这样换电脑之后直接很快就可以获取到博客源文件了
git clone git@github.com:EvaLotus/EvaLotus.github.io.git
git checkout hexo
npm install
# 因为theme文件在gitignore下，所以每次还需要重新clone下，不然会遇到no layout的错误。
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

其他问题可以直接去看[hexo的文档](https://hexo.io/zh-cn/docs/index.html)，已经灰常良心了！

搭建博客很简单，而且有很多主题可供大家选择，但是最重要的是保持每天记录的习惯。

如果遇到其他问题可以给我留言，一起来搭建自己的博客吧。

---
layout: post
title: hexo+githubPages建站最佳实践
date: 2017-06-05 14:06:09
tags: hexo github
---
前两天超哥很开心的敲下`hexo d`，再加上手贱莫名其妙删掉了源文件，写了4年的博客全都没有啦，只能一个个copy出来，简直是喜闻乐见（超哥不要打我，逃～）

研究了下hexo+githubPages的最佳实践，也是踩了不少坑才用的比较顺滑。

<!-- more -->

以下是操作必备，只是mark一下

```bash
# 安装hexo命令行工具
cnmp i hexo-cli -g
# 在此目录搭建博客，在github创建此项目
cd EvaLotus.github.io //
heox init 
npm install
# 启动服务
hexo s
# 选择喜欢的主题，如next，实在是太容易撞衫了，但是确实是极简主义的爱
git clone https://github.com/iissnan/hexo-theme-next themes/next //将主题clone至themes/next目录下
# 删除generate的文件，切换主题时最好执行下，可以不用自己删掉public文件
hexo clean
# 生成静态页面，可以在本地调试时使用
hexo g 
# 如果只是直接部署
hexo d
```

### FAQ 常见问题解决
1. 怎么能解析到自己的域名上去？

    直接在github的project settings里，设置custom domain，这样就会自动在你的项目中增加一个CNAME文件，里面会记录你自己的域名

2. 如何`hexo d`自动更新部署博客
    
    需要安装插件`npm i hexo-deployer-git --save`，在`_config.yml`中配置
    ```
    # Deployment
    ## Docs: https://hexo.io/docs/deployment.html
    deploy:
    type: git
    repo: git@github.com:EvaLotus/EvaLotus.github.io.git
    branch: master
    ```
    在配置`_config.yml`时一定要注意此类文件对缩进要求很严格，在sublime中编辑会有适当的颜色提醒你是否缩进正确

3. 每次`hexo d`之后我的CNAME都没了怎么办？
    
    小伙伴们应该都发现了，`hexo d`之后并没有提交source目录，只是提交了生成后的public中的文件，所以我们要注意，CNAME，README，Favicon.ico都应该放在source目录下，这样`hexo d`之后会直接放在public的根目录下

4. 换电脑了或者像超哥那样误删了源文件怎么办？
    
    很多小伙伴应该都有同时在公司和在家修改博客的需求，其实我们可以使用分支来管理我们的博客，简言之，master上存放生成的静态文件，hexo存放源文件。

    操作上并不需要切换分支，第一次大致配置完毕之后，
    ```bash
    # 切出hexo分支
    git checkout -b hexo
    #以后都在此分支上修改，虽然hexo d可以帮你将生成的静态文件自动push到master
    git checkout master
    hexo d
    #但是每次修改之后还是记得自己把源文件的修改push一下
    git push origin/hexo
    
    #Easy!
    # 这样换电脑之后直接很快就可以获取到博客项目
    git clone git@github.com:EvaLotus/EvaLotus.github.io.git
    
    git checkout hexo
    
    npm install
    # 很Easy的获取到了源文件～
    ```
其他不会的可以直接去看hexo的文档，已经灰常良心了！
    



---
title: web开发提升效率
date: 2017-07-23 15:12:42
tags: 效率
---
自从入了极简主义的坑，觉得自己开始无欲无求了。极简主义就是寻找生活的最优解，作为了一个专业的web 开发者，每天要花很多时间在开发工作上，事先花点时间打造一个高效舒适的开发环境，会让你的开发效率如飞，以下是我的装机必备清单。

<!-- more -->

有道云笔记

mac

## iterm 2
修改iterm2主题

下载[配色方案](http://iterm2colorschemes.com)
cmd+i选择color，选择color preset，import 3024 night ,text把字体改为36px,window修改transparent。重启iterm 2 ，记得在general中选择把当前风格设置为默认的

双击选中，三击选中整行，选中即复制

cmd+t 新建tab

cmd+w 关闭tab

cmd+arrow 左右切换tab

cmd+/ 会显示你的鼠标

cmd+R 清屏

cmd+enter进入/退出全屏模式

查找：command + f

分屏

垂直分屏：command + d

水平分屏：command + shift + d

切换屏幕：command + option + 方向键 command + [ 或 command + ]

查看历史命令：command + ;

查看剪贴板历史：command + shift + h

ctrl + u 清空当前行，无论光标在什么位置

## Browser
chrome extension：
- React Dev Tools:react必备
- FE助手：包括color picker，json format，ajax等
- Postman：发送request
- Page load time（计算网页load时间）
- jsonViewer
- Page Ruler
- Livereload(sublime中也装)

firefox,safari,ie

## IDE & 编辑器
### sublime
### 安装package control
从菜单 View ->Show Console 或者 ctrl + ~ 快捷键，调出 console。将以下 Python 代码粘贴进去并 enter 执行
```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

解决utf-8问题

sublime plugin
1. 【emmet】
2. 【html-css-js prettify】格式化代码
3. 【snippet】查看snippet中的快捷键并添加自己的snippet
4. 【image2Base64】 
5. 【sass】
6. 【saveOnBuild】
7. 【autoprefixer】：在css中直接使用ctrl+alt+shift+p即可autoprefixer
8. 【sublime server】可以不需要hbuilder了：右键view in sublime server
9. 【sideBarEnhance】
10. 【git】ctrl+shift+p然后输命令
11. 【livereload】实现自动刷新：只有静态服务器时可以实现自刷新
- sublime中package control安装的livereload有问题，在github中下载后可以使用，删掉ST3-livereload
- 在chrome中安装插件livereload，点击右侧开启

12. 【bootstrap 3 snippet】
13. 【gulp】在control+shift+p可以运行各种gulp
14. 【DocBlockr】代码注释生成
15. 【tomorrow color scheme】直接preferences->color scheme选择tomorrow night
16. 【markdown preview】
17. 【autoFilename】路径自动补全

用dropbox来同步不同机器里的sublime text设置

From 
```
cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/
mkdir ~/Dropbox/Sublime
mv User ~/Dropbox/Sublime/
ln -s ~/Dropbox/Sublime/User
```
To

```
cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/
rm -r User
ln -s ~/Dropbox/Sublime/User`

```


### vs code



## homebrew：mac下的包管理工具
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
```
brew install git
brew install node
gem install sass
brew update
brew install mongo

```


```
npm install -g cnpm --registry=https://registry.npm.taobao.org //淘宝镜像
npm install nodemon -g//node 开发和部署工具
npm install pm2 -g


//安装我需要的常用npm
npm i -g bower eslint express generator-react-webpack generator-webapp generator-angular generator-express generator-react-fullstack gulp hexo-cli sass swig webpack yo yarn
//分别是：静态包管理bower，语法校验eslint，node 服务器express，yoeman常用的generator，静态博客hexo，sass，后端模版swig，webpack，yoeman，yarn之后可能使用yarn替代npm

npm update -g//全局更新所有过时的
npm outdated -g//列出所有过时的


yo react-webpack/express/reacr-fullstack//生成不同模版

```

```
cd /d/workspace/demo
npm init//生成package.json
npm install express --save //安装express

```

## VMWare，Axure，IDE，Juniper，NptePad++,Hbuilder,好压,winscp,beyond compare,
## 其他调试工具

### switchHosts
### git tortoise (mac 下source tree)
### fiddler (mac 下Charles)

海外版fiddler证书安装不了的方法：将滑动解锁改成数字解锁，从系统安全中安装文件

fiddler->toolbar->textwizard可以进行编码解码

### 环境变量配置：
JAVA_HOME C:\Program Files\Java\jdk1.8.0_111
MAVEN_HOME D:\workspace\maven\apache-maven-3.2.5
NODE_PATH C:\Program Files\nodejs\node_global\node_modules
Path:
C:\ProgramData\Oracle\Java\javapath;%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;C:\Program Files\Java\jdk1.8.0_111\bin;%MAVEN_HOME%\bin;C:\Program Files\nodejs\;C:\Program Files\TortoiseGit\bin;%RUBY_HOME%\bin;C:\Program Files (x86)\Git\bin;C:\Ruby23-x64\bin;

【beyond compare】
【ps】
【wechat】
【outlook】

### Oh my zsh
Terminal里bash的最佳替代品，强大的自动补全功能，能自动补全命令、参数、文件名、进程、用户名、变量、权限符等， 以及各种插件以及主题的支持。
```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
## docker


下载jenkins.war直接java -jar jenkins.war

# Eva's Blog
github clone


# mac浏览器快捷键
cmd+R刷新等于F5

cmd+option+I等于F12

cmd+option+J等于F12

cmd+option+U等于显示源代码

cmd+option+左右

空格向下滚动网页


## 暂定
npm
gitbook-cli,


修改bash的颜色和字体
右键options，



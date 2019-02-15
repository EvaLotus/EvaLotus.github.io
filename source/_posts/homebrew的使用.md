---
title: homebrew的使用
date: 2017-11-18 17:06:31
tags: Tool
---
Home-brew是mac上的软件安装管理工具，有了它每次安装软件都不用去google了，直接`brew install`就可以啦。

<!-- more -->
```
# 安装home-brew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。

```
# 版本
brew -v

# 更新 Homebrew版本
brew update

# 帮助
brew -h

# 检查问题
brew doctor

# 查看所有包
brew list

# 查看你的包是否需要更新
brew outdated

# 更新包
brew upgrade <package_name>

# 安装包
brew install <package_name>
```

Homebrew 将会把老版本的包缓存下来，以便当你想回滚至旧版本时使用。但这是比较少使用的情况，当你想清理旧版本的包缓存时，可以运行：

```
brew cleanup
```

常用包

```
brew install openssl mongodb
brew rm docker

```



---
layout: post
title: rvm 工作原理
date: 2017-05-20 00:00
tags: [ruby, rvm]
---

# 带着几个问题去研究RVM

### rvm方便了我们什么？
- ruby 的版本控制
- 通过 gemset 方便了大家同一个 ruby 环境下不同gem的切换
- ruby 的更新、安装、如此的容易

### rvm怎么工作的？
- rvm 就是一个 shell 脚本，通过修改 PATH 来达到切换ruby环境的目的

##### 举个例子

```  
  使用 
  rvm use 2.3.0
  其实是修改了 PATH ,随之而变的是ruby 的环境 gem 的环境 gemset 的环境
  PATH=~/.rvm/rubies/2.3.0/bin:~/.rvm/gems/2.3.0/bin:~/.rvm/gems/2.3.0@global/bin:$PATH
  默认会生成gloabl的gemset
  Linux命令搜索顺序:
  当我们键入某个命令时, 那么shell会按照alias->keyword->function,->built-in->$PATH的顺序进行搜索, 本着”搜索顺序是从左至右的,先到先得”的原则, 就是说如果有如名为mycmd的命令同时存在于alias和function中的话, 那么肯定会使用alias的mycmd命令(当然, 这不是绝对的, 下面会说到特例).
```  

### rvm的原理？
```
  个人感觉原理很简单，就是通过shell脚本实现了下载，通过修改环境变量来修改ruby版本
```

### rvm gemset是怎么工作的？
```
  gemset的工作原理和ruby的选择原理一直，就是通过修改PATH,当你选择gemset的时候就会把对应的gemset放在$PATH的最前面
  举个例子
  rvm use 2.3.0
  rvm gemset create test
  rvm 2.3.0@test
  gem install oj 这时候就会安装gem到 ~/.rvm/gems/ruby-2.3.0@test/gems
  
  原因是修改了PATH
  
  PATH=~/.rvm/gems/ruby-2.3.0@test/bin::/Users/Will/.rvm/gems/ruby-2.3.0@global/bin...
```
---
layout: post
title: gem install 和 require 工作原理
date: 2017-05-21 00:00
tags: [ruby, gem, require]
---


## gem install 后面执行了什么？

>大家肯定认为 gem 肯定是一个包就类似 bundle 的似得一个gem包，但是当我们执行
which gem -> 

```
require 'rubygems'
require 'rubygems/gem_runner'
require 'rubygems/exceptions'

required_version = Gem::Requirement.new ">= 1.8.7"

unless required_version.satisfied_by? Gem.ruby_version then
  abort "Expected Ruby Version #{required_version}, is #{Gem.ruby_version}"
end

args = ARGV.clone

begin
  Gem::GemRunner.new.run args
rescue Gem::SystemExitException => e
  exit e.exit_code
end
```
> 由此可见 gem 其实就是一个执行 `shell` 里面引用 rubygems 但是rubygems又是什么，其实rubygems 是 rubygem-update 这个gem包定义的，rubygem-update 这个gem 包定义 gem 的下载,解析,安装

### 那我们具体来分析 gem install 具体做了哪几件事？

```
举个例子：
  gem install oj
  先去 rubygem.org 去下载 指定版本的gem
  之后解压gem
  之后判断gem的完整性，就是判断是否是一个完整的gem，是通过gemspec文件来判断
  之后通过Gem::Specification 来查看是否已经注册过此版本的gem，如果注册了直接next
  之后将gemspec 添加到 Gem::Specification 指定的目录 也就是gem_path的相对目录 specifications
  之后查找GEM_HOME GEM_PATH
  之后解压gem到GEM_PATH
  之后cp gem到cache_gem
  之后cp gem的bin文件到GEM_HOME/bin目录
  这就是一个gem install 完成

  对应命令：
    gem unpack oj
    gem environment
```

### 那我们require oj时后台逻辑做了什么呢？

```
  require oj 只是做了这么几步：
    1、首先查找Gem::Specification看这个gem的name是否注册，注册了的话继续, 
      - spec = Gem::Specification.find_by_path('oj')
    2、将 $GEM_PATH/gems/oj/lib >> $LOAD_PATH
      - spec.activate # 这时候会把加到LOAD_PATH
    3、之后会将$GEM_PATH/gems/oj/lib/oj.rb require 到内存，可以直接通过$:来查看
      - kernel原生的require也就是 gem_original_require
```

### 那我们看 rubygem 又对require做了什么？

```
  rubygem又对kernel的require文件做了重写
  下面是 require 的源码
   
   # 首先判断gem_original_require是否定义
   # gem_original_require就是kernel的别名
   if defined?(gem_original_require) then
    # Ruby ships with a custom_require, override its require
    remove_method :require
  else
    ##
    # The Kernel#require from before RubyGems was loaded.

    alias gem_original_require require
    private :gem_original_require
  end
  
  # rubygem 重写的 require
  def require path
    RUBYGEMS_ACTIVATION_MONITOR.enter

    path = path.to_path if path.respond_to? :to_path

  # 首先去查找对应的gemspec是否注册过
    spec = Gem.find_unresolved_default_spec(path)
    if spec
      Gem.remove_unresolved_default_spec(spec)
      # 注册过就直接加载gem
      gem(spec.name)
    end

    # If there are no unresolved deps, then we can use just try
    # normal require handle loading a gem from the rescue below.

  # 如果没有
    if Gem::Specification.unresolved_deps.empty? then
      RUBYGEMS_ACTIVATION_MONITOR.exit
      # 就调用kernel的require
      return gem_original_require(path)
    end
    ..........

   ```
   学习链接： http://www.justinweiss.com/articles/how-do-gems-work//

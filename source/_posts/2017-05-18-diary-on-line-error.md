---
layout: post
title: 上线问题记录
date: 2017-05-18 00:00
tags: [diary, ruby]
---

## 今天遇到的问题？

> 由于自己太过自信，心里想一个小功能怎么会出问题，再说也在alpha环境测了，然而并不是还是因为自己的疏忽出错了

### 上线一个小功能网站怎么会挂掉？
  主要是因为网站的主逻辑依赖的这个功能没有添加catch导致，网站被crash掉,小功能没有完全测试覆盖导致在使用redis.lpush的时候 push 的是空数组，导致报错
### 网站挂掉怎么处理的？rollback怎么会不起作用？
  出了问题第一时间 cap deploy:rollback但是没有起作用，反而出现了更加严重的问题current目录索引的release为空，为什么？？？？？

  原因是：
    并不是rollback不执行，而是之前部署会一半一半的部署，也就是假如四台机器的话会先部署两台机器之后再部署两台机器，所以就导致release版本的时间戳不同，所以在rollback时选择rollback的release版本时导致两台机器link的地址为空导致部署失败

    cap deploy:rollback的原理：
      - 先kill掉当前跑的进程
      - 通过ls -xl . 来选择最近的第二个版本如果没有设置回退的版本的话，假如设置了就会回退设置的版本
      - 软连接到current
      - 启动服务
### 问题在哪？
  首先个人方面，为了赶进度没有补充完测试，再者cap deploy:rollback设计有缺陷（个人观点）

### 总结
  一定要写测试
  假如测试覆盖不到
  一定要加开关随时能关掉小功能
  上线的小功能一定要加catch
  cap console 可以在线调试其实原理很简单，通过程序ssh 多台服务器，等待命令 loop

  
### 上点源码：

  desc "Execute remote commands"
  task :console do
    stage = fetch(:stage)
    puts I18n.t("console.welcome", scope: :capistrano, stage: stage)
    loop do
      print "#{stage}> "

      command = (input = $stdin.gets) ? input.chomp : "exit"

      next if command.empty?

      if %w{quit exit q}.include? command
        puts t("console.bye")
        break
      else
        begin
          on roles :all do
            execute command
          end
        rescue => e
          puts e
        end
      end
    end
  end

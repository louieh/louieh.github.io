---
title: "Debug: Lightsail - yum update"
layout: post
date: 2020-01-13
headerImage: false
tag:
- 生命在于折腾
star: true
category: blog
---



最近再调 fakecoursebook 的数据更新内存占用高的问题，有代码的问题，有mongo的问题，有apscheduler的问题，到现在还没有完全解决，这个之后会单独写一篇记录。这次来说另一个事儿。

最近经常发现 lightsail 机器 ssh 连不上，即使连上也非常卡，ssh 连不上，只能关机再开机（lightsail web 页面上的重启按钮貌似没有作用）。起初以为是内存的问题，可能也是因为这个才开始考虑数据更新的内存问题。后来通过 lightsail web 监控页面发现是 cpu 高使用率，问题出现时 cpu 使用率一度达到 100%。一次偶然开着 htop 在监控内存的时候突然一个高 cpu 进程上来，导致当时 ssh 连接直接卡死，好久后才恢复，只瞟到了一眼那个进程是包含 yum update 什么的，因为当时是 MEM 排序，那个进程马上又掉下去了。当时没有太在意，以为只是 yum 的更新而已，而事实上就是 yum 的定时更新。

直到刚刚再次出现这个问题，cpu 高使用率，ssh连不上，没有重启机器，等待好久之后恢复。查看系统日志 /var/log，先是 cron log 发现居然有 crontab 定时在跑才想起来是很久很久前加的，于是 crontab -r 删掉，之后是 maillog 进去发现很多

Jan 14 03:25:40 ip-172-26-10-190 sendmail[2555]: rejecting connections on daemon MTA: load average: 36

应该是高 load 拒绝连接。之后 message 中找到问题：

Jan 14 03:25:05 ip-172-26-10-190 kernel: [415598.589797] Out of memory: Kill process 10678 (yum) score 452 or sacrifice child Jan 14 03:25:05 ip-172-26-10-190 kernel: [415598.596059] Killed process 10678 (yum) total-vm:575544kB, anon-rss:229016kB, file-rss:0kB, shmem-rss:0kB
 Jan 14 03:25:05 ip-172-26-10-190 kernel: [415598.894526] oom_reaper: reaped process 10678 (yum), now anon-rss:0kB, file-rss:0kB, shmem-rss:0kB

yum 导致 out of memory. 显然是 yum 自动 update 导致。之后 secur 里也很多 disconnected。

之后在 StackExchange 上查到一个问题：How to disable Amazon EC2 instance yum automatic nightly check-update? 说每晚都会有这个进程执行：

/usr/bin/python2.7 /usr/bin/yum --debuglevel 2 --security check-update

答案居然是此目的是生成每次进行 ssh 连接后显示的信息：？？？？

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/ 10 package(s) needed for security, out of 21 available          <<<=== This Run "sudo yum update" to apply all updates.                     <<<=== and this [ec2-user@ip-172-31-11-77 ~]$

它是一个 cron 任务，在 /etc/cron.d/update-motd，停止的话注掉 /etc/update-motd.d/70-available-updates.

https://serverfault.com/questions/932532/how-to-disable-amazon-ec2-instance-yum-automatic-nightly-check-update

https://mlog.club/article/2218488
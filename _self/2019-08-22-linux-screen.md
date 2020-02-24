---
layout: post
category: "mysql"
title:  "使用Linux screen管理远程会话"
---

一个会话可以包含多个窗口。

- 创建会话：screen -S main
- 暂时离开当前会话：Ctrl + a + d
- 进入会话：输入命令screen -r main (名字或者进程ID)
- 删除会话：先进入窗口，然后 Ctrl + a + k，然后输入命令 screen -wipe

- 创建窗口：Ctrl + a + c
- 切换窗口：Ctrl + a + 0（窗口编号）

---
title: Mac OS 下修改Hosts文件的有效方法
date: 2018-03-26 14:37:27
tags: [Mac]
reward: true
---

##### 使用nano 编辑器修改

在Termial中执行以下命令：

```
sudo nano /etc/hosts
```
输入命令后输入认证密码，密码输入后回车换行，打开hosts文件，按照你的需要对该文件进行编辑，编辑完毕后按 command+o 保存，出现"File Name to Write:/etc/hosts"的时候按回车确认，再按 command+x 退出即可。
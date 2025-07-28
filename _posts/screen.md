---
title: 连接服务器时 screen 工具的使用
data: 2025-07-28 22:00:00 +0800
category: 软件
tag: [服务器，screen]
description: 连接服务器时 screen 工具的使用
---

SSH 连接远程服务器跑代码时，常常一个代码要很长时间才能跑完，但本地电脑可能需要断连或关机，此时可以使用 screen。

创建一个名为 “test” 的 screen。
``` sh
screen -S test
```
查看所有创建的 screen
```sh
screen -ls

There is a screen on:
        694091.test     (07/28/2025 09:18:06 PM)   (Attached)
1 Socket in /run/screen/S-123456.
```
退出screen
按 ctrl+a+d，此时会退出当前的 screen，但刚刚的 screen 还是存在的，里面的程序还是在运行的。此时可以关闭 SSH 连接或者关闭本地电脑。

进入名为 “test” 的 screen
```sh
screen -r test
screen -r 694091
```

---
title: Simplicity Studio v5 基础操作
data: 2025-03-15 22:56:00 +0800
category: 软件
tag: [嵌入式, ssv5]
---

## 1. 前言
最近的项目用到了芯科科技（Silicon Labs）的芯片，于是接触到了 Simplicity Studio v5。此博客记录一些相关操作。

持续更新中。

## 2. 改变工作路径
当默认的工作路径中含中文时，创建项目会报错。
![alt text](./ssv5-创建项目报错.png){: height="400"}
```sh
Failed to create new Configurable Project (SLCP)!
Multi-Exceptions available:
  Multi-Exceptions available:
  Problems generating template files from component: bluetooth_stack - Failed to generate template for F:\gecko-sdk\protocol\bluetooth\src\sl_bluetooth.c.jinja... Couldn't open script file.
  Problems generating template files from component: bluetooth_stack - Failed to generate template for F:\gecko-sdk\protocol\bluetooth\src\sl_bluetooth.h.jinja... Couldn't open script file.
  Problems generating template files from component: bootloader_app_properties - Failed to generate template for F:\gecko-sdk\platform\bootloader\app_properties\config\sl_application_type.h.jinja... Couldn't open script file.

…………
```

“文件”-“Switch Workspace …”
![alt text](./ssv5-改变工作路径.png){: height="400"}

## 3. 导入项目
“File”-“Import …”
![alt text](./ssv5-导入项目.png){: height="400"}

## 4. 编译程序
右键项目名-“Bulid Project”
![alt text](./ssv5-编译项目.png){: height="400"}

## 5. 烧录固件
程序编译后项目目录中会有一个“Binaries”文件夹
![alt text](./ssv5-烧录固件.png){: height="400"}

---
layout: post
title:  "eclipse项目迁移到android studio"
date:   2015-11-30 1:38:05
categories: Android
excerpt: android studio
---

# 准备

## 1. 将eclipse项目的项目依赖暂时删除。

## 2. 逐个检查依赖的项目，如果项目项目下有类似 android studio 项目下的文件， 全部删除（避免麻烦）
     
      例如：build目录 .idea目录 gradle相关的文件

## 3. 新建一个 eclipse android lib 项目， 将需要迁移的项目本身和所有的依赖项目 中`重复使用的` jar包 复制到新的项目中，然后删除原来的jar包

## 4.确保前面3步完成。备份好。

# 第一步，将项目导入成 as 项目，（包括主项目，和库项目，新加的库项目也需要）。

# 第二步，假设 主项目是A  库项目是B，C   新库项目是D

# 第三步，将库项目加入到主项目中，作为module

# 第四步，修改项目内的build.gradle 文件，添加依赖：

  项目A  ： compile project(':B');   compile project(':C');  compile project(':D');

  项目B，C  : compile  project(':D') 


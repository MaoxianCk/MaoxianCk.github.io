---
title: SpringBoot Web 项目开发 (二) 工程结构
date: 2019-06-26 16:51:50.0
updated: 2021-01-08 16:54:14.462
url: https://maoxian.fun/archives/springbootweb项目开发二工程结构
categories: 
- 程序
- SpringBoot
tags: 
- 程序
- Spring
- Web
---

上一篇文章中，我们已经搭建好了springboot web项目的基本架构，但是web开发当然只有这些是不够的，而一个项目当然也需要一个比较好的结构划分，

![img](https://img-maoxian-fun.oss-cn-hangzhou.aliyuncs.com/MxBlogImg/image-9b8189d3e15f07574698cf9232d72c34-bb380d-1610093408.png?x-oss-process=style/mxcompress)

- main 主要的源代码区
  - java 具体的代码层
    - config 配置，springboot的一些配置类可以放在这里，例如swagger的配置类等
    - error 异常处理，自定义的异常处理类等
    - interceptor 拦截器
    - utils 工具类，放置项目中通用的工具
    - validator 通用的校验工具
    - controller 控制器，在微服务架构中主要是api的定义、参数的校验以及调用service服务层进行业务处理
    - service 业务层，主要的业务逻辑都在这里，数据库dao层的调用、复杂数据的生成处理、密码的加密解密、文件处理等
    - dao 持久化层，数据库处理层，只有该层可以控制数据库
  - resources 资源层
    - mapping 示例项目中是存放mybatis-generater生成的mapping文件
    - static 静态资源
    - templates 模板页面，如thymeleaf等编写的模板页面
    - application.properties springboot的配置文件
    - mybatis-generator.xml mybatis-generator插件的配置文件
- test 测试层，测试代码
- pom.xml Maven项目依赖管理的配置文件
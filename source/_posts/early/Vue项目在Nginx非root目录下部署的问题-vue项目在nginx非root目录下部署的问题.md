---
title: Vue项目在Nginx非root目录下部署的问题
date: 2020-10-21 16:51:59.0
updated: 2021-01-08 17:07:55.884
url: https://maoxian.fun/archives/vue项目在nginx非root目录下部署的问题
categories: 
- 程序
- Web
tags: 
- 程序
- 代码
- Web
---

## 错误排查

最近在部署Vue项目时，出现如下错误：

![img](https://img-maoxian-fun.oss-cn-hangzhou.aliyuncs.com/MxBlogImg/009501e2311bca14f237f5d30a9f8ac2-378b44-1610095800.png?x-oss-process=style/mxcompress)

一开始以为是常见的无限路由导致的爆栈这类基础问题，但是考虑到在本地调试时一切正常，并且在本次版本更新前生产环境也正常运行。于是直接被整懵，一度怀疑是更新了依赖包版本导致的问题。在尝试了调整路由配置、依赖包版本回退、项目回退均无果后，换了台服务器进行部署测试，结果正常运行。

比较后发现，在项目部署的原服务器上，还运行着一个用户端项目，路径如下：

```nginx
/      用户端
/admin 管理员端
```

尝试将管理员端项目部署在原用户端根路径上时，发现又正常，于是推测是项目部署在Nginx的非root即 ‘/’ 根路径上出了问题。

## 解决方案

修改管理员端项目的配置文件

该项目使用的是vue-cli3，故修改vue.config.js文件

```nginx
module.exports = {
  // publicPath: './',   // 注释原配置
  publicPath: '/admin/', // 修改为要配置的路径，注意两个斜杠
  其他配置...
}
```

重新build项目，检查打包后的dist文件夹中的index.html文件的资源引用路径是否正确（publicPath打头的路径）

![img](https://img-maoxian-fun.oss-cn-hangzhou.aliyuncs.com/MxBlogImg/f48c4c3cab9bf32d52e1043e5341c724-54d547-1610095811.png?x-oss-process=style/mxcompress)

修改nginx配置文件，nginx.conf

```nginx
# 用户端路径，使用root配置
location / {
    root   html/dist;
    try_files $uri $uri/ @router;
    index  index.html index.htm;
}
# 管理员端配置，使用alias配置虚拟路径，匹配路径时使用rewrites重写定位
location ^~/admin {
    alias html/admin/;
    try_files $uri $uri/ @rewrites;
}
# 将管理员端路径重写定位
location @rewrites{
    rewrite ^/(admin)/(.+)$ /$1/index.html last;
}
# 后端代理
location /local {
    proxy_pass  http://localhost:8080/;
}
```

执行 nginx -t 检查配置文件

执行 nginx -s reload 重载配置文件

完事。
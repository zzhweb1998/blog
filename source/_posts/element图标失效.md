---
title: element图标失效
date: 2020-09-17 23:37:59
tags: ["element-ui"]
top_img: /img/element图标失效/1.PNG
cover: /img/element图标失效/1.PNG
categories: 练习笔记
---
## vue打包后引起element-ui字体图标失效原因有两种：
1. 打包后引用的路径有问题
vue中引用element-ui后，引用时间模块在打包后不显示图标可能时打包后引用的路径有误
![](/img/element图标失效/2.PNG)
在build文件夹中找到utils.js，在这个js文件修改
修改前：
if (options.extract) {
  return ExtractTextPlugin.extract({
    use: loaders,
    fallback: 'vue-style-loader',
  })
在里面加入
修改后：
if (options.extract) {
  return ExtractTextPlugin.extract({
    use: loaders,
    fallback: 'vue-style-loader',
    // element-ui图标不显示问题
    publicPath: '../../'
  })

2. 打包进后端运行时拦截
如果浏览器报提示：
Failed to decode downloaded font: /font/element-icons.woff
OTS parsing error: incorrect file size in WOFF header
说明使用 maven 的 resource 插件开启 filtering 功能后，会破坏有二进制内容的文件。
解决方案：修改pom.xml
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <excludes>
                <exclude>**/*.ttf</exclude>
                <exclude>**/*.woff</exclude>
            </excludes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>false</filtering>
            <includes>
                <include>**/*.ttf</include>
                <include>**/*.woff</include>
            </includes>
        </resource>
    </resources>
</build>
```
## 总结
  在本次运维旧项目中遇到单个页面出现element图标不显示问题，一开始以为时第一种
情况，但后面发现路径其它的都没发现有问题，后来查看提示语发现可能是后端引起的，修改
后端后也确实解决了问题，但由于这就旧项目是以vue-cli2创建的，面又引入一个自定义vue项
目，自定义vue项目是直接引用element-ui包的，打包后自定义vue的页面才出现这种情况，而
vue-cli2构建的项目引入的element-ui就没问题，尚未搞懂什么情况，猜想是vue-cli2脚手架在
打包时做了处理。

## 参考链接：
https://blog.csdn.net/weixin_30326741/article/details/95160787
https://www.jianshu.com/p/e1fc82756afc



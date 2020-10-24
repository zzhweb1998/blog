---
title: 安装ffi-napi
date: 2020-10-24 16:48:33
tags: ["koa2","ffi-napi"]
categories: 配置教程
copyright: false
---

## 安装ffi-napi
注意：如果访问的dll文件是32位的，node需要安装32位的版本确认好需要node的版本，安装好node后进行一下操作：以管理员身份运行控制台输入
1、安装所需程序
```
	npm install --global --production windows-build-tools（此命令为一键安装）
	解释：一键安装以下程序
	1、python(v2.7 ，3.x不支持);
	2、visual C++ Build Tools,或者 （vs2015以上（包含15))
	3、.net framework 4.5.1
	安装成功后把python的.exe文件添加到环境变量中：C:\Users\zhouz\.windows-build-tools\python27\python.exe
```
2、全局配置node-gyp
```
	npm install -g node-gyp
```

安装以上步骤后，在node项目下安装ffi-napi:
```
	npm i ffi-napi
```

安装完ffi-napi，在项目引入调用dll：
```
const ffi = require('ffi-napi');
const path = require('path');

const ffipath = path.join(__dirname, '../dll/DllInjectHelperDll.dll')
const demo = new ffi.Library(ffipath, {
    'InjectDll': ['int', ['int', 'string']]
});
let data = demo.InjectDll(13808,'D:/privateCode/koa2/koa2_demo/dll/FunctionHook32.dll')
```


ffi-napi错误原因：
Error: Win32 error 193 （原因是Node.js是64位而dll是32位的）
Error: Win32 error 126 （没有这个目录或者这个目录下没有要找的dll文件，还可能dll还依赖了
其他的一些dll, 但是无法找到这个dll）
Error: Win32 error 127 （找到了dll文件, 但是在dll中找不到你声名的函数.这通常是由于函数名
字错误, 或者是返回值类型/参数的个数及类型不一致导致的）

参考链接：
https://www.jianshu.com/p/5af3ad2b0856
https://www.cnblogs.com/silenzio/p/11606389.html
https://www.cnblogs.com/wangyuxue/p/11218113.html

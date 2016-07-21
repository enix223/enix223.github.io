---
layout: blog
categories: blog
published: false
title: Nodejs开发实践
tags: ''
---
## Nodejs开发技巧

1. 监控项目文件改变，并重启server    

		npm install -g supervisor
	
    	$ supervisor index.js
    
2. 使用supervisor并开启node debug模式

		$ supervisor -- --debug index.js
    
3. nodejs 调试工具: node-inspector

		$ npm install -g node-inspector
	
    	# 打开node-inspector, 之后就可以通过chrome浏览器调试代码
		$ node-inspector &

## Mongoose实践

1. 定义schema后，指定对应的collection名字

		// 指定mongodb的collecition名字为'userinfo'
        var UserInfo = new Schema({
          username : String,
          password : String 
        }, { collection: 'userinfo' });

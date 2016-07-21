---
layout: blog
categories: blog
published: false
title: Nodejs best practise
tags: ''
---
### 使用技巧

1. 监控项目文件改变，并重启server    

		npm install -g supervisor
	
    	$ supervisor index.js
    
2. 使用supervisor并开启node debug模式

		$ supervisor -x node-debug index.js
    
3. nodejs 调试工具: node-inspector

		$ npm install -g node-inspector
	
    	# 打开node-inspector, 之后就可以通过chrome浏览器调试代码
		$ node-inspector &

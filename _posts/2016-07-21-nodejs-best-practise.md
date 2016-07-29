---
layout: blog
categories: blog
published: true
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
        
## Nodejs 开发实践

1. 配置route的时候，避免重复输入url的前缀(prefix)

		/**
         * 实现公共route前缀，减少重复输入次数
         *
         * /api/v1/user 
         * /api/v1/post
         *
         */
		var router = express.Router();
        router.use('/user', user);
        router.user('/post', post);

        app.use('/api/v1', router);

## Mongoose实践

1. 定义schema后，指定对应的collection名字

		// 指定mongodb的collecition名字为'userinfo'
        var UserInfo = new Schema({
          username : String,
          password : String 
        }, { collection: 'userinfo' });
        
2. mongoose model默认添加virtual属性至json中

        var schemaOptions = {
	        toObject: {
    		    virtuals: true
        	}
        	,toJSON: {
        		virtuals: true
        	}
        };

3. 根据document中的sub-document查找，并只返回匹配的sub-document

		/*
         Category: {categoryName: 'ABC', books:[{bookName: 'BBQ'}, {bookName: 'CCD'}]}
         现在需要按books.bookName查找
         */
         CategoryModel.find({"books.bookName": 'BBQ'}, function(err, result){});
         
         // 以上语句，默认返回整个document，也就是仍然包括CCD的book，如果需要只选择 BBQ的book，可以加上如下语句：
         CategoryModel.find(
         	{"books.bookName": 'BBQ'},
            {books: {'$elemMatch': {bookName: 'BBQ'}}}
            function(err, result){}
         );
         
         // 或者使用Positional operator `$`
         CategoryModel.find(
         	{"books.bookName": 'BBQ'},
            {'books.$': 1}
            function(err, result){}
         );
         
4. select语句排除某些field (如果某个document有很多field，而我们只需要滤掉其中几个，可以使用如下语法)

		CategoryModel.find(
        	{}, // Finding criteria
            {'books': 0}, // 过滤掉books这个field，也就是结果中不会包含books这个field
            function(err, result)
        );


## MongoDB实践

1. 删除document的一个field
		
        //删除catagory field
    	db.catalog.update({}, {$unset:{catagory: 1}});

2. 向document添加field

		// 添加category, slug FIELD
		db.catalog.update({}, {$set:{category: 'cheetsheet', slug: 'vi'}});
         
        

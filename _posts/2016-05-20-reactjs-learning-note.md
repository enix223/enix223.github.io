---
layout: blog
categories: blog
published: false
title: ReactJs 学习笔记1
tags: ""
---
## Get started

1. 创建可重用的元素

        var Component = React.createClass({
            render: function(){
                return (<div className="component"> Hello world </div>);
            }
        });

2. 渲染元素

		ReactDOM.render(<Component></Component>, document.getElementById("parent"));


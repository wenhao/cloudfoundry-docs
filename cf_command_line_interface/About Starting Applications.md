##关于启动应用

这篇文章假设你使用cf命令行工具(v6)为前提。

```cf push```使用以下三种资源之一来启动你的应用：

1. ```-c```选项，例如：

	```
	$ cf push my-app -c "node my-app.js"
	
	```
	
2. 在部署清单里面添加```command```属性，例如：

	```
	command: node my-app.j
	
	```
	
3. 针对不同类型的应用，buildpack提供对应的启动命令行。

```cf```的执行原理解释如下：

###cf push如何决定使用哪一个默认启动命令
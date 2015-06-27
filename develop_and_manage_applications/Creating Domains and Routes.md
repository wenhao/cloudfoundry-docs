##创建域名和路由

*此文章假设你使用的cf命令行工具(v6)*

此主题描述如何在Cloud Foundry创建和使用域名和路由。
>注意：这里谈到的域名和Cloud Foundry不一样。

在Cloud Foundry中，你可以根据这篇文章创建域名。Cloud Foundry路由通过这些域名转发请求到应用程序。你也可以在Cloud Foundry中使用你自定义的域名。

域名与组织相关联并不绑定到具体的应用程序。域名可以共享也可以私有独立使用。共享域名可以在不同的组织上注册，然后私有域名或者自有域名只能在某个组织上注册。如果你不指定其他域名，Cloud Foundry会为你的应用程序分配一个共享的域名。

路由是可以在网上访问当应用程序的路径。在Cloud Foundry，每个路由都绑定到一个或者多个应用程序。路由URL是由域名和一个可选的主机名前缀组成。例如，“myappname”为主机名称，“example.com”为域名，路由URL就是“myappname.example.com”。在此“example.com”可以为自定义或者自己已有的域名。

###使用根域名
根域名或者无前缀域名例如“example.com”是没有主机名称的域名。要想使用根域名访问部署的应用程序，你必须设置自定义记录类型或者子域名重定向。

下面章节说明如何使用ALIAS或者ANAME自定义记录类型或者子域名重定向映射根域名到应用程序。

####为DNS配置ALIAS或者ANAME记录

如果你的DNS提供商支持ALIAS或者ANAME记录，在DNS提供商配置你的根域名。

ALIAS和ANAME记录配置模式如下：

**记录**  | **名称** | **目标** | **说明**
------------- | ------------- | ------------- | -------------
ALIAS或者ANAME | 空或者@  | foo.example.com  | 根据你的DNS提供商的文档决定使用空还是@

如果你的DNS提供商不支持ANAEME和ALIAS记录，使用子域名重定向，就是熟知的域名转发，添加根域名。如果你使用这种方式，注意访问根域名的SLL请求会出错，是因为SSL认证只能用于子域名。

配置根域名重定向到“www”子域名，设置CNAME记录配置“www”子域名映射为目标程序URL。下表举例说明此配置。

**记录**  | **名称** | **目标** | **说明**
------------- | ------------- | ------------- | -------------
URL或转发 | example.org  | www.example.com  | 这种方式会发出```301 permanent redirect```到你配置的子域名。
CNAME | www  | foo.cfapps.io  | 

####配置应用程序使用根域名

下面章节包含说明如何为你的应用程序路由创建并使用域名。由于路由是没有主机名称的根域名，域名和路由在此处表现为“example.org”。

1. 在Cloud Foundry创建域名并于组织关联起来。用以下命令在“test-org”组织里创建“example.org”域名：
	
	```
	$ cf create-domain test-org example.org
	```
2. 映射此路由到你的应用程序。此命令依赖你是否使用共享域名。如果你的路由使用的不是共享域名，使用```cf map-route```命令如下：
	
	```
	$ cf map-route myapp example.org
	```
	
	如果路由使用的是共享路由，在使用```cf map-route```命令时要加```-n HOSTNAME```参数，HOSTNAME必须是唯一的。这条命令会做两件事：
	
	* 合并主机名称“app”和现有的路由“myapp.example.org”，生成新的路由“app.myapp.example.org”
	* 映射新的路由到名为“myapp”的应用程序
	
	```
	$ cf map-route myapp example.org -n app
	```

###创建父域名


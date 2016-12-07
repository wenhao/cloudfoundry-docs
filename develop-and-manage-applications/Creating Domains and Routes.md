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

域名的不同部分，“example.com”是子域名“myapp”的父级域名。

####在DNS提供商配置CNAME记录

参见[在DNS提供商配置CNAME记录](./Creating Domains and Routes.md#在DNS提供商配置CNAME记录)。

####使用私有或共享父级域名配置你的应用程序

父级域名必须满足以下需求：

* **私有父级域名**：创建的私有的域名必须是私有子域名的父级域名。
* **共享父级域名**：创建的共享域名可以是共享或者私有子域名的父级域名。

###在组织之间共享私有域名

在组织管理器里面，你可以配置某个私有域名用于多个组织。你必须有组织管理权限。选择属于你的某个组织。

从Cloud Controller：

* ```PUT /v2/organizations/:GUID/private_domains/:PRIVATE_DOMAIN_GUID```关联私有域名到此组织。你不能```put```到你自己的组织。
* ```DELETE /v2/organizations/:GUID/private_domains/:PRIVATE_DOMAIN_GUID```与此组织解关联私有域名。你不能删除自己的组织。
* ```GET /v2/private_domains/:GUID/shared_organizations```列出属于此私有域名的所有共享组织。

###使用私有或共享子域名

在Cloud Foundry中，域名存在多个部分如在“myapp.example.com”包含子域名“myapp”。在这种情况下，“example.com”是“myapp”的父级域名。在域名“test.myapp.example.org”中，“test”是子域名而父级域名是“myapp.example.org”

####在DNS提供商配置CNAME记录

使用CNAME记录配置域名。

CNAME记录规则如下所示：

**记录**  | **名称** | **目标** | **说明**
------------- | ------------- | ------------- | -------------
CNAME | test  | foo.example.com.  | 参看你所使用DNS提供商的文档决定是否需要以```.```结尾。
Wildcard CNAME | sample  | *.example.com.  | 你可以使用通配符配置你的CNAME记录来映射你所有的子域名到父级域名。每一个单独的子域名配置优先于通配符配置。

####使用私有或者共享子域名配置应用程序

你可以映射子域名到已有的域名来创建新域名。每一个添加的子域名都是其父级域名的子域名。这些子域名必须满足以下条件：

* **私有子域名**：
	* 你自能映射属于同组织的私有子域名到私有父级域名。
	* 你可以映射私有域名到一个共享的父级域名。
* **共享子域名**：
	* 你只能映射某个共享子域名到某个共享父级域名。
	* 你不能映射某个共享子域名到某个私有父级域名。

给应用程序使用子域名、创建域名与映射路由如下：

1. 在Cloud Foundry创建域名并于组织关联。下列命令是在“test-org”组织里面定义域名“myapp.example.org”：
	
	```
	$ cf create-domain test-org myapp.example.org
	```
2. 为应用程序映射路由并添加可选参数主机名称，例子如下：
	
	```
	$ cf map-route myapp myapp.example.org -n test
	```

###管理你的域名

####列出某个组织的所有域名

你可以使用```cf domains```查看某个组织的所有可用域名。

在下面的例子当中，有两个可用的域名：一个系统范围默认的域名“example.com”和自定义的“example.org”域名。

```
$ cf domains
Getting domains in org my-org as user@example.org... OK

name           status
example.com    shared
example.org    owned
```

####分配域名和主机

你可以使用命令行或者部署清单文件分配域名或者主机。

#####使用命令行分配

当你执行```cf push```时，你可以选择性的为应用程序设置域名和主机。

* **域名**：使用```-d```参数指定某个组织的域名。
* **主机**：使用```-n```参数主机字符串。

Cloud Foundry为应用程序创建的路由为：```myapp.example.org```。

```
$ cf push myapp -d example.org -n myapp
```
#####使用部署清单分配

当你创建或者编辑你应用程序的部署清单文件时，你可以指定```host```和```domain```属性来分配应用程序的主机名称和域名。更多的信息，参见[应用程序部署清单](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)。

####删除域名

你可以使用```cf delete-domain```命令在Cloud Foundry上删除域名。

```
$ cf delete-domain example.org
```

###管理你的路由

####列出所有路由

你可以执行```cf routes```命令列出当前空间的所有路由。注意主机名称有域名分组。

例如：

```
$ cf routes
Getting routes as user@example.org ...

host                     domain          apps
myapp                    example.org     myapp1
myapp2
1test                    example.com     1test
sinatra-hello-world      example.com     sinatra-hello-world
sinatra-to-do            example.com     sinatra-to-do
```

####创建路由

使用```cf create-route```命令创建路由并于之后需要用到的空间关联。你可以提供可选参数```-n HOSTNAME```在同一域名下为每一个路由指定一个唯一的主机名称。

例如， 以下命令在“development”空间内创建“myapp.example.org”路由：

```
$ cf create-route development example.org -n myapp
```

如果你是管理员，你可以使用通配符如```*```在指定主机名称，对于主机名称，符号```*```可以匹配你域名下所有的URL。
>注意：在命令行中```*```必须使用双引号。

例如，以下命令在空间“development”中创建使用通配符的路由“*.example.org”。

```
$ cf create-route development example.org -n "*"
```

####分配或改变路由

使用```cf map-route```命令为某个应用程序分配或改变路由。主机名称是可选的。如果路由不存在，运行此命令则会先创建路由然后再映射到应用程序。

例如， 以下命令映射路由“myapp.example.org”到“myapp”应用程序。

```
$ cf map-route myapp example.org -n myapp
```
使用通配符映射路由会变成一个备选方案如果用户输入的访问地址不能匹配其他所有路由配置。例如，如果用户输入“myap.example.org”试图访问绑定到“myapp.example.org”的应用程序，用户访问的应用程序绑定的是“*.example.org”。

以下命令映射路由“*.example.org”到应用程序“myfallbackapp”:
```
$ cf map-route myfallbackapp example.org -n *
```

>注意：你可以在同一空间中映射某个路由到多个应用程序。了解更多信息请参见[蓝绿部署](http://docs.cloudfoundry.org/devguide/deploy-apps/blue-green.html)。

当你映射一个正在运行的应用程序时，新的路由规则只有在应用程序重启之后才会生效。

####删除路由

你可以使用```cf unmap-route```命令删除一个程序的路由。解映射之后，该路由还可以在此空间内继续使用。

```
$ cf unmap-route myapp example.org -n myapp
```

你可以使用```cf delete-route```命令从某个空间中删除某个路由。

```
$ cf delete-route example.org -n myapp
```

注意以上命令用来指定主机名称的参数```-n```是可选的。
##开始使用cf CLI
此指南会对使用cf命令行工具(v5)用户介绍新版本cf命令行工具(v6)有哪些新特性。

对比老版本v5，新版本cf命令行工具(v6)使用更为简单，运行速度更快并且更加强大。有以下不同：

* 新版本cf命令行工具使用Go语言完全重写，比起之前的Ruby版性能有很大提升。
* 对大多数主流操作系统提供本地化安装包。
* 命令行更加的通俗易懂：必选参数没有任何标志；可选参数都具有标志。
* 命令行关键字更短，但更容易理解并且大多数都有单字符别名。
* cf命令行工具(v5)版本中很多命令都需要读取manifests，而在cf命令行工具(v6)中只有push命令才会读取manifests。
* cf命令行工具(v5)大量使用了交互式提示命令，而在cf命令行工具(v6)中，只有在用户登录和创建user-provided服务的时候才会用到交互式提示命令(在创建user-provided时，你也可以选择不使用交互式提示)。

在cf命令行工具(v6)使用指南中，对于一些老版本没有的或者改进过的功能，我们会着重介绍如何使用它们。

>注意：如果你在cf命令行工具中执行某些命令行不成功，很可能那些命令行只有Cloud Foundry管理员才有权限执行。如果你不是Cloud Foundry管理员，你会看到如下错误消息：
>**error code: 10003, message: You are not authorized to perform the requested action**

###安装
cf命令行工具(v6)的安装只需要简单的执行安装包即可，不再要求你需要先安装Ruby开发环境。你可以直接使用新的可执行文件或者使用本地安装包。安装详情请参见[安装cf命令行工具](./Installing the cf Command Line Interface.md)。

我们建议你时刻关注我们的[工具](https://console.run.pivotal.io/tools)何时发布最新版本，你可以下载任意你想要更新的版本的可执行文件或者本地安装包。

###本地化
cf命令行工具会根据你设置的语言自动翻译所有的输出。
>注意：只有cf命令行工具产生的输出才有国际化特性。

运行``` cf config --locale 语言标志```设置语言。默认的语言为英语``` en_US ```。

例如：

```
cf config --locale pt_BR
```

cf现已支持8种不同的语言：

* 俄语：``` de_DE ```
* 英语：``` en_US ```
* 西班牙语：``` es_ES ```
* 法语：``` fr_FR ```
* 意大利语：``` it_IT ```
* 日语：``` ja_JA ```
* 葡萄牙语(巴西)：``` pt_BR ```
* 简体中文：``` zh_Hans ```

###登陆
在cf(v6)新版中，``` login ```命令做了些扩展。在登陆的时候除了用户名和密码之外，你还可以追加目标接口地址，组织和空间。

如果不指定任何参数，cf命令行工具会给出提示：

* **API endpoint(接口地址)**：[这是你的Cloud Foundry服务器实例的Cloud Controller地址](http://docs.cloudfoundry.org/running/cf-api-endpoint.html)。
* **Username(用户名)**：登陆用户名。
* **Password(密码)**：登陆密码。
* **Org(组织)**：应用需要部署到得组织。
* **Space(空间)**：部署组织中的空间名称。

如果你只有一个组织并且组织里只存在一个空间的话，你就可以省略这两个参数，``` cf login ```会默认选择你唯一的组织和空间。用法：

```
cf login [-a API_URL] [-u USERNAME] [-p PASSWORD] [-o ORG] [-s SPACE]
```

为了方便，你可以写一段脚本并预先设置好自己的API地址，依照个人喜好你也可以使用一些没有交互提示的命令如```cf api ```，``` cf auth ```和``` cf target ```命令。

一旦登陆成功，cf命令行工具(v6)会保存一个叫```config.json```文件，此文件包含了你的API地址，组织名称，空间名称及登陆访问令牌。当你改变你的设置时，```config.json```会跟着改变。

默认情况下，```config.json```保存在```~/.cf```目录下。通过设置环境变量```CF_HOME```，你可以蒋此文件保存到任意位置。

###部署
在cf命令行工具(v6)中，```push```更为快捷简单。

* ```APP```，所部署应用的名称，这是你唯一需要提供的参数而且不需要添加额外的标志。如果你的部署清单文件中已经包含了应用的名称，就可以直接忽略此参数。
* 大多数的命令行参数都有单个字符的别名。例如，参数主机名称或子域名可以用```-n```代替```--host```。
* 冗余的交互模式不再存在，更加简化。
* 创建清单不再繁琐。详情请参见[部署应用清单](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)。
* 在创建服务的时候不再需要push或者在清单中注明。如果想了解更多关于如何创建服务请参见[用户提供服务](http://docs.cloudfoundry.org/devguide/installcf/whats-new-v6.html#user-provided)。
* ```-m```内存大小参数需要注明单位：```M```，```MG```，```G```或者```GB```，大小写皆可。

cf命令行工具(v6)添加了一些新的参数：

* ```-t```(超时时间)允许设置项目启动时间，最大180秒。
* ```--no-manifest```强制忽略已存在的部署清单。
* ```--no-hostname```在没有主机名称的情况下，可以用域名指定一个路由。
* ```--no-route```不适用当前路由而使用自定义路由。在部署产品应用时，可以使用此参数。

用法：

```
cf push APP [-b URL] [-c COMMAND] [-d DOMAIN] [-i NUM_INSTANCES] [-m MEMORY] /
[-n HOST] [-p PATH] [-s STACK] [--no-hostname] [--no-route] [--no-start]

```

可选参数有：

* ```-b``` ——自定义buildpack地址，例如[https://github.com/heroku/heroku-buildpack-play.git](https://github.com/heroku/heroku-buildpack-play.git)或者 [https://github.com/heroku/heroku-buildpack-play.git#stable](https://github.com/heroku/heroku-buildpack-play.git#stable)选择```stable```分支。
* ```-c``` ——启动应用。
* ```-d``` ——域名，例如：```example.com```。
* ```-f``` ——替代```--manifest```。
* ```-i``` ——启动实例的数量。
* ```-m``` ——内存的大小，例如：256，1G，1024M等等。
* ```-n``` ——主机名称，例如：```my-subdomain```。
* ```-p``` ——应用文件路径。
* ```-s``` ——使用栈。
* ```-t``` ——启动超时时间，单位秒。
* ```--no-hostname``` ——应用的根域名(新)。
* ```--no-manifest``` ——忽略已存在的部署清单。
* ```--no-route``` ——不为当前应用分配路由(新)。
* ```--no-start``` ——部署之后不启动应用。

>注意：```--no-route```选项会移除已经部署应用的路由配置。

###用户自定义服务实例
cf命令行工具(v6)提供了一些新的命令行来创建和更新用户自定义服务实例。你有三种方式来运行这些命令：交互式的，非交互式的，或者如[RFC 6587](http://tools.ietf.org/html/rfc6587)所提到的通过使用第三方日志管理系统。当你使用第三方日志系统，cf会根据[RFC 5424](http://tools.ietf.org/html/rfc5424)规范自动发送格式化后的日志信息。

创建完成之后，可使用```cf bind-service```命令把此服务实例与现有的应用绑定，解绑定可以使用```cf unbind-service```，重命名此服务实例使用```cf rename-service```，删除服务实例```cf delete-service```。

####创建用户自定义服务命令
```create-user-provided-service```别名是```cups```.

交互式的方式创建服务实例，使用```-p```选项，不同的服务之间使用```,```逗号隔开。执行此命令之后，cf命令行工具会依次提示你如何创建每一个服务。
```
cf cups SERVICE_INSTANCE -p "host, port, dbname, username, password"
```

非交互式的方式创建服务实例，在使用```-p```选项的时候，参数以**JSON**的形式提供。
```
cf cups SERVICE_INSTANCE -p '{"username":"admin","password":"pa55woRD"}'
```

创建服务实例时集成第三方日志管理系统，使用```-l SYSLOG_DRAIN_URL```选项。
```
cf cups SERVICE_INSTANCE -l syslog://example.com
```

####更新用户自定义服务命令
```update-user-provided-service```别名是```uups```。你可以使用```cf update-user-provided-service```更新服务实例中的一个或多个属性。不支持的属性则不会更新。

非交互式的方式更新服务实例，在使用```-p```选项的时候，参数以**JSON**的形式提供。
```
cf uups SERVICE_INSTANCE -p '{"username":"USERNAME","password":"PASSWORD"}'
```

更新服务实例时集成第三方日志管理系统，使用```-l SYSLOG_DRAIN_URL```选项。
```
cf uups SERVICE_INSTANCE -l syslog://example.com
```

###域名、路由、组织与空间

域名、路由、组织与空间之间的关系，在cf命令行工具(v6)中得以简化。

* 每一个域名都对应一个组织。
* 路由任然从属于空间和应用。

以前，路由的URL由```HOSTNAME.DOMAIN```组成，如果你不提供主机名称(也就是子域名)，路由的URL就默认由```APPNAME.DOMAIN```组成。

cf命令行工具(v6)分开管理私有域名和共享域名，并且只有管理员才能管理共享域名。

对于域名，cf命令行工具(v6)同时支持新旧版本的```cf_release```。

>注意：```map-domain```和```unmap-domain```在新版中不在支持。

用于管理域名的命令如下：

* ```cf create-domain``` —— 创建域名。
* ```cf delete-domain``` —— 删除域名。
* ```cf create-shared-domain``` —— 共享域名，仅限管理员。
* ```cf delete-shared-domain``` —— 删除共享域名，仅限管理员。

用户管理路由的命令如下：

* ```cf create-route``` —— 创建路由。
* ```cf map-route``` —— 映射路由。如果应用之前没有映射路由，则先创建再映射。
* ```cf unmap-route``` —— 解映射路由。
* ```cf delete-route``` —— 删除路由。

##### 映射路由

1. 使用```cf create-domain```先在某个组织中创建一个原本不存在或者没有被共享的域名
2. 使用```cf map-route```映射此域名到某个应用，再使用```-n HOSTNAME```选项为每个路由指定一个唯一的主机名称。

>注意：你可以在同一个空间里面映射某个路由到多个应用。想了解更多请参见[蓝绿部署](http://docs.cloudfoundry.org/devguide/deploy-apps/blue-green.html)。

###用户与角色

关于用户的命令：

* ```cf org-users``` —— 以角色分组，列出组织中所有的用户。
* ```cf space-users``` —— 以角色分组，列出空间中所有的用户。

关于用户角色的命令(仅限管理员)：

* ```cf set-org-role``` —— 给用户分配特定的组织角色。可用的角色有：```OrgManager```，```BillingManager```和```OrgAuditor```。
* ```cf unset-org-role``` —— 删除用户的组织角色。
* ```cf set-space-role``` —— 给用户分配特定的空间角色。可用的角色有：
```SpaceDeveloper```和```SpaceAuditor```。
* ```cf unset-space-role``` 删除用户的空间角色。

###统一的标准帮助你更高效的工作

在cf命令行工具中，我们立志于帮助运维和开发人员创建更为简单高效的命令。

cf命令行工具中的命令更简单有意义：

* 必须参数没有任何标志。
* 可选参数都具有标志。
* 必须参数不止一个时，不能颠倒顺序。
* 多个可选参数可以以任意顺序排列。

例如：```cf create-service```，它需要三个参数而且必须按照```SERVICE```、```PLAN```和```SERVICE_INSTANCE```的顺序排列。再者，```cf push```命令需要一个必选参数和多个可选参数，对于可选参数可以以任何顺序排列。

命令行的名字尽量简单一看即懂，例如：

* ```cf map-route```可以用```cf map```代替。
* ```cf marketplace```可以用```cf m```代替。

开发人员常用到的```cf curl```命令在cf命令行工具(v6)中又很大改进。

* ```curl```自动使用你登陆之后的身份验证。
* ```curl```使用方法被尽可能的简化为类似于UNXI的模式。

####新的别名和命令行更为规范

cf命令行工具(v6)中添加的别名及命令行使其更加的简单易懂。

cf命令行工具(v6)使用单字符别名。例如：

* ```cf p```是```cf push```的别名。
* ```cf t```是```cf target```的别名。

cf命令行工具(v6)提供更为流畅的线性命令行规范：

* 用户参数全大写，例如：```cf push APP```。
* 可选参数包含标识符和中括号，例如：
```
cf create-route SPACE DOMAIN [-n HOSTNAME]
```

执行```cf help```可以查看所有的cf命令行及使用帮助。执行```cf <command-name> -h```查看某个特定命令行的使用方法(包含别名)，例如：

```
 $ cf p -h
    NAME:
       push - Push a new app or sync changes to an existing app

    ALIAS:
       p

    USAGE:
       Push a single app (with or without a manifest):
       cf push APP [-b BUILDPACK_NAME] [-c COMMAND] [-d DOMAIN] [-f MANIFEST_PATH]
       [-i NUM_INSTANCES] [-m MEMORY] [-n HOST] [-p PATH] [-s STACK] [-t TIMEOUT]
       [--no-hostname] [--no-manifest] [--no-route] [--no-start]

       Push multiple apps with a manifest:
       cf push [-f MANIFEST_PATH]

    OPTIONS:
       -b             Custom buildpack by name (e.g. my-buildpack) or GIT URL
                      (e.g. https://github.com/heroku/heroku-buildpack-play.git)
       -c             Startup command, set to null to reset to default start command
       -d             Domain (e.g. example.com)
       -f             Path to manifest
       -i             Number of instances
       -m             Memory limit (e.g. 256M, 1024M, 1G)
       -n             Hostname (e.g. my-subdomain)
       -p             Path of app directory or zip file
       -s             Stack to use
       -t             Start timeout in seconds
       --no-hostname    Map the root domain to this app
       --no-manifest    Ignore manifest file
       --no-route        Do not map a route to this app
       --no-start        Do not start an app after pushing

```
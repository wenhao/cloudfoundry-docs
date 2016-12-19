<!--
##Cloud Foundry Components
Page last updated: December 9, 2016
-->
##Cloud Foundry组件

最后更新: 2016-12-9

<!--
Cloud Foundry components include a self-service application execution engine, an automation engine for application deployment and lifecycle management, and a scriptable command line interface (CLI), as well as integration with development tools to ease deployment processes. Cloud Foundry has an open architecture that includes a buildpack mechanism for adding frameworks, an application services interface, and a cloud provider interface.
-->
Cloud Foundry组件包括一个自我服务的应用程序运行引擎，一个自动化的应用程序部署引擎和生命周期管服务，还有一个脚本化的命令行接口(CLI)，同时也提供了一些开发工具来简化整个部署过程。Cloud Foundry整个系统拥有一个开发的架构，包括能够自定义添加框架的buildpack机制，开放的应用服务接口及云服务提供商接口。

<!--
Refer to the descriptions below for more information about Cloud Foundry components. Some descriptions include links to more detailed documentation.
-->
更多关于Cloud Foundry组件信息可以查看下图。某些描述信息还链接到更为详细的文档。

![Cloud Foundry架构](../images/cloud-foundry-concepts/cf_architecture_block.png)

<!--
###Routing
-->
###路由

<!--
Router
-->
####路由器

<!--
The router routes incoming traffic to the appropriate component, either a Cloud Controller component or a hosted application running on a Diego Cell.
-->

[路由]分发请求到特定的组件，要么是控制器，要么是一个托管应用程序的Diego单元。

<!--
The router periodically queries the Diego Bulletin Board System (BBS) to determine which cells and containers each application currently runs on. Using this information, the router recomputes new routing tables based on the IP addresses of each cell virtual machine (VM) and the host-side port numbers for the cell’s containers.
-->
路由定期的查询Diego电子布告栏系统来决策哪个单元或者容器的应用程序

<!--
###Authentication
-->
###认证

<!--
####OAuth2 Server (UAA) and Login Server
-->

<!--
The OAuth2 server (the UAA) and Login Server work together to provide identity management.
-->
OAuth2([UAA])和登陆服务器共同管理认证服务。

###Cloud Controller

[Cloud Controller](http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html)负责管理应用程序的生命周期。当开发人员部署应用程序到Cloud Foundry后，应用程序就被Cloud Controller接管。Cloud Foundry会存储应用程序字节码文件，为应用程序元数据创建跟踪记录，然后分配DEA节点预处理和运行此应用程序。Cloud Foundry也维护组织、空间、服务、服务实例、用户角色等信息。

###HM9000

HM9000有四个主要的职责：

* 监控并获取应用程序的状态(例如：运行、停止和崩溃等), 版本号和实例数量。HM9000会根据运行在DEA上的应用程序的心跳测试和`droplet.exited`消息状态来更新应用程序的实际状态。
* 定义应用程序的预期状态，版本号和实例数量。HM9000可以从Cloud Controller数据库里查询应用程序的预期状态。
* 对比实际状态和预期状态，调整应用实例数量。举例来说，如果某个应用程序只有几个实例的运行状态是所期望的，HM9000就会通知Cloud Controller创建并启动对应数量的实例。
* 通知Cloud Controller处理任何状态错误的应用程序。

更多关于HM9000架构的详细信息参见[HM9000 readme](https://github.com/cloudfoundry/hm9000)。

###应用执行单元(DEA)

[Droplet 执行代理](http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html)管理应用程序实例，跟踪任何已启动应用实例并广播应用程序各种状态信息。

应用程序实例运行在[Warden](http://docs.cloudfoundry.org/concepts/architecture/warden.html)容器内。集装箱式的架构保证应用程序实例之间相对独立，远离任何共享资源和阻碍其他实例干扰。

###大数据存储

大数据存储包括：

* 应用程序代码
* Buildpacks
* Droplets

###服务代理

应用程序通常依赖某些[服务](http://docs.cloudfoundry.org/services/)例如，数据库或者第三方SaaS提供商。当某个开发人员创建并绑定某个服务到应用程序后，服务代理就会负责创建服务所需要的服务实例。

###消息总线

Cloud Foundry使用[NATS](http://docs.cloudfoundry.org/concepts/architecture/messaging-nats.html)，一个轻量级的消息订阅和分发系统，用于组件之间的通信。

###日志与统计

度量收集器会收集各个组件的度量信息。管理员可以通过这些信息来监控Cloud Foundry的实例。

应用程序的日志聚集器会收集并发送应用程序的日志给开发人员。

[路由]: http://docs.cloudfoundry.org/concepts/architecture/router.html
[UAA]: http://docs.cloudfoundry.org/concepts/architecture/uaa.html

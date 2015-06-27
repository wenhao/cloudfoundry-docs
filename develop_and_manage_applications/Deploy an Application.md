##部署应用

*此文件假设你使用的cf命令行工具(v6)*

>注意：对于采用不同语言及框架开发的应用程序，请参见更为详细完整的部署文档[buildpacks]()，例如[部署Ruby on Rails应用程序](http://docs.cloudfoundry.org/buildpacks/ruby/gsg-ror.html)指南。

###部署流程概述

通过使用cf命令行工具执行```push```命令部署你的应用程序到Cloud Foundry。详细内容参见[安装cf命令行工具](../cf_command_line_interface/Installing the cf Command Line Interface.md)。从执行```push```命令之后到应用程序启动之前，Cloud Foundry会执行下列任务：

* 上传并存储应用程序文件。
* 校验并存储应用程序元数据。
* 为应用程序创建“droplet”(Cloud Foundry最小可执行单元)。
* 选择最适合的“微执行代理”(droplet execution agent, DEA)执行“droplet”。
* 启动应用程序。

应用程序需要用到的一些服务，例如，数据库、消息服务或者邮箱服务，只有在你部署了应用之后，才能够绑定这些服务到你的应用程序。详情参见[服务概述](http://docs.cloudfoundry.org/devguide/services/)主题。

###步骤1：部署准备

在你部署应用程序到Cloud Foundry之前，确保：

* 应用程序对云服务友好。Cloud Foundry对文件存储、HTTP会话和端口有要求，部署之前对应用程序做相应的修改。
* 上传应用程序所有的资源文件。例如，数据库驱动。
* 多余的文件或者工件最好排除。你应该尽可能的排除不需要上传的文件，特别对于大型应项目。
* 创建应用程序服务所需要的所有服务实例。
* Cloud Foundry实例支持你将要部署的应用程序类型或者外部可用的buildpack来处理你的应用程序。

关于准备部署你的应用程序，请参见：

* [在云服务上设计与运行应用注意事项](./Considerations for Designing and Running an Application in the Cloud.md)
* [Buildpacks]()

###步骤2：明确你的认证信息和目标地址

在部署你的应用程序到Cloud Foundry之前，你需要知道：

* 你的Cloud Foundry实例的API地址。换言之目标URL，参见[the URL of the Cloud Controller in your Cloud Foundry instance](http://docs.cloudfoundry.org/running/cf-api-endpoint.html)。
* 你的Cloud Foundry实例用户名和密码。
* 应用程序部署的组织和空间。Cloud Foundry工作区由不同组织构成，而组织则由不同的空间组成。Cloud Foundry的用户，可以任意访问一个或多个不同的组织和空间。

###步骤3：(可选)配置域名

Cloud Foundry通过路由分发请求到应用程序，应用程序URL由主机名词和域名组成。

* 应用程序的名词是默认的应用程序主机名称。
* 每个部署到空间的应用程序都有一个默认域名。你可以不使用默认的域名，在部署应用程序的时候可以使用自定义的域名，并且在组织中注册此域名映射到此应用程序空间。

更多关于域名，参见[Create Domains and Routes](http://docs.cloudfoundry.org/devguide/deploy-apps/domains-routes.html)。

###步骤4：确定部署选项

部署之前，你需要设置一下选项：

* **名称**：应用程序名词只能由字母和数字组成，不包括空格。
* **服务实例**：一般而言，运行的实例越多，应用程序停机时间就越短。如果你的应用程序任然在开发过程中，单个实例更容易做故障排除。对于任何应用程序产品环境，我们建议至少两个服务实例。
* **内存大小**：每个服务实例都有最大内存限制，如果应用程序运行时达到了此限制，Cloud Foundry就是重启此服务实例。
	>注意：首先，如果超过了最大内存限制，Cloud Foundry会立即重启服务实例。如果在一段时间内，应用程序不断地超过最大内存限制，Cloud Foundry将推迟重启此服务实例。
	
* **启动命令**：此命令用于启动应用程序的服务实例。不同的应用程序框架使用的启动命令是不同的。
* **子域名和域名**：路由是由子域名和域名组成，而且唯一。你可以选择自定义或者使用Cloud Foundry给定的默认值。
* **服务**：应用程序可以绑定服务例如数据库、消息服务和键值存储系统。应用程序部署到应用空间。只有当应用程序服务实例存在于应用空间内的情况下，应用程序才能绑定服务。

####设置部署选项


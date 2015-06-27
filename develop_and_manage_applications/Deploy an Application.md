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


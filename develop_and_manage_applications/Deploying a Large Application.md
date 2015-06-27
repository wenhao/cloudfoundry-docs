##部署大型应用

此主题描述在Cloud Foundry中部署的大小在750 MB和1 GB之间应用程序配置的限制和建议。

###部署的限制和注意事项

Cloud Foundry最大支持1 GB的应用程序。

部署过程包括上传、预处理和启动应用程序。你的应用程序必须在管理员预先设置的指定的时间内成功的执行每一个步骤。每一个阶段默认时间如下：

* 上传：15分钟
* 预处理：15分钟
* 启动：60秒

>注意：管理员可以修改这些值。在部署之前与管理员确定每一个阶段限制的时间长短。

在Cloud Foundry上部署大型应用，确保：

* 你的网络连接速度必须允许在15分钟内上传应用程序。Pivotal建议最低速度为874 KB/s。
* 为没一个应用程序的服务实例分配足够的运行内存。你可以在执行```cf push```命令是使用```-m```参数或者在```manifest.yml```中设置内存大小。
* 如果你使用部署清单文件，```manifest.yml```，确保给应用设置合适的值，例如内存大小、启动时间限制和磁盘空间大小。更多关于使用部署清单文件，参见[使用部署清单部署应用](./Deploying with Application Manifests.md)主题。
* 只部署应用程序需要的文件。为了达到此目标，只部署应用程序目录下的文件并且使用```.cfignore```文件移除不需要的文件[可排除的文件类型](./Considerations for Designing and Running an Application in the Cloud.md#Ignore Unnecessary Files When Pushing)。
* 如果需要你可以在部署清单文件中重置预处理、启动和超时的时间。
	* ```CF_STAGING_TIMEOUT```：控制在应用程序打包上传之后到成功预处理之间最大的等待时间，单位分钟。
	* ```CF_STARTUP_TIMEOUT```：控制应用程序启动的最大等待时间，单位分钟。
	* ```cf push -t TIMEOUT```：控制应用程序启动的最大等待时间，使用此设置，cf会忽略在部署清单中的```CF_STARTUP_TIMEOUT```环境变量配置，单位秒。

更多关于使用cf命令行工具部署应用程序，参见开始使用cf命令行工具中的[部署](../f_command_line_interface/etting Started with the cf CLI.md#部署)主题。
>注意：有时即便cf命令行工具报出```App failed```错误消息，你的应用程序可能已经启动成功，这是因为你在命令行设置的超时时间可能与管理员在部署清单设置的值不一样。举个例子，管理员设置应用程序启动时间限制为180秒。你在部署清单中设置的时间则为120秒，同时你在命令行中使用的超时时间为60秒然后开始部署。在这种情况下，即便命令行报错，你的应用程序可能已部署成功因为没有超过管理员设置的最大时间。执行```cf apps APP_NAME```查看应用程序的状态。

###默认配置及限制汇总表

下表是在Cloud Foundry部署大型应用所用到的一些默认值。

| **设置**  | **说明** |
| ------------- | ------------- |
| App Package Size  | Maximum: 1 GB  |
| Authorization Token Grace Period  | Default: 20 minutes, minimum  |
| ```CF_STAGING_TIMEOUT```  | CLI environment variable, Default: 15 minutes  |
| ```CF_STARTUP_TIMEOUT```  | CLI environment variable, Default: 5 minutes  |
| ```cf push -t TIMEOUT```  | App start timeout maximum, Default: 60 seconds  |
| Disk Space Allocation  | Default: 1024 MB  |
| Internet Connection Speed  | Recommended Minimum: 874 KB/s  |

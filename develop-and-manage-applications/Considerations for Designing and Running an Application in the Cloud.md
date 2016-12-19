##在云服务上设计与运行应用注意事项

###针对云设计应用

如果应用程序的设计遵循以下几条规则，通常应用程序不用做任何修改就可以在Cloud Foundry上运行。遵循这些规则，使应用程序更适合云服务，也同样是应用程序适合其他云平台。

下面这些规则都是针对云平台设计现代应用程序的最佳实践。更多详细信息请阅读针对云服务的优秀应用设计，参见[优秀应用程序的12条关键因素](http://www.12factor.net/)

####避免使用本地文件系统

运行在Cloud Foundry上的应用程序应该避免写入任何信息到本地文件系统。理由如下：

* **本地文件系统存储生命周期短暂。**当一个应用程序崩溃或者停止时，云平台会收回之前应用程序启动时分配给服务器的资源也包括应用于本地磁盘所有资源。当此服务器重新启动时，此应用程序会启用一个新的磁盘空间。经管你的应用程序能够在运行时写入本地文件，但是这些文件在应用程序重启后会消失不见。
* **应用程序的不同服务器不能同时共享某个本地系统文件。**每一个应用程序实例都运行在独立的容器里面。所以某个服务实例写入的文件对于此应用程序的其他服务实例来说是不可见的。如果这些文件是临时文件，就不会有任何问题。但是，在应用程序重启的时候，你的应用程序需要从本地文件里面读取数据并做持久化的话或者此应用程序的不同服务实例之间有共享这些文件数据，本地系统文件就不应该使用。在这种情况下，我们建议使用公共服务诸如共享的数据库或者大数据存储系统来解决。

例如，你可以使用Cloud Foundry上服务如文档数据库MongoDB或者关系型数据库MySQL或者Postgres。另外一种选择是使用云存储提供商提供的服务诸如[Amazon S3](http://aws.amazon.com/s3/)，[Google Coud Storage](https://cloud.google.com/products/cloud-storage)，[Dropbox](https://www.dropbox.com/developers)或者[Box](http://developers.box.com/)。如果你的应用程序服务实例之间有信息交流，可以考虑使用缓冲技术例如Redis或者基于消息传递机制的RabbitMQ。

####不保存和复制HTTP会话

Cloud Foundry支持会话关联或者称为粘性会话，前提是应用程序使用接收到的请求cookie中包含jsessionid。如果应用程序含有多个服务实例运行在Cloud Foundry上，某个用户的所有请求始终会发送到同一个应用程序服务器实例上。这样特定的用户会话信息就会存储在应用程序的容器或者框架里面。

会话信息必须保证在应用程序崩溃或者停止之后还能够使用，或者可以被此应用程序的其他服务实例共享使用，必须存储在Cloud Foundry服务中。有一些开源项目可以支持在数据服务器中共享会话信息。

####端口限制

运行在Cloud Foundry上的应用程序通过配置好的URL获取请求。HTTP请求发送到端口80或者443上。此外，Cloud Foundry会使用一个TCP/WebSocket接口。默认情况下，```cf-release```部署清单会分配4443端口给TCP/WebSocket通道。

如果你的均衡负载服务器占用了此端口，必须为TCP/WebSocket选择其他未被占用的端口，为了访问应用程序的日志，你需要按照以下步骤并运行命令```cf logs APP_NAME```：

* 在部署清单文件里面，设置```logger_endpoint```的子属性```port```.
* 配置HAProxy或者负载均衡服务器监听此端口。

###部署的时候忽略不需要的文件

默认情况下，当你部署你的应用程序的时候，应用程序项目下得所有文件都会上传到Cloud Foundry服务实例上，除了版本控制器和配置文件如：

* ```.cfignore```
* ```_darcs```
* ```.DS_Store```
* ```.git```
* ```.gitignore```
* ```.hg```
* ```/manifest.yml```
* ```.svn```

如果应用程序目录下包含其他文件(如```temp```或者```log```)，或者在构建或运行过程中都不需要的某个子目录，最佳实践是使用```.cfignore```配置文件(```.cfignore```类似于git的```.gitignore```文件，他可以定义不需要给git跟踪的文件及目录)。特别对于一些大型应用，如果上传了一些不需要的文件肯定会减低整个应用程序部署过程的效率。

在```.cfignore```中注明你想排除的文件类型或者目录，保存此文件在项目的根目录。例如，下面几句话排除了“tmp”和“log”两个目录。

```
temp/
log/
```

根据应用程序使用框架的不同，需要排除的文件类型也各不一样。针对不同的框架，github提供了很多```.gitignore```模板可以直接使用，参见[https://github.com/github/gitignore](https://github.com/github/gitignore)，非常好用。

###运行多个服务实例以提高可用性

当一个DEA升级完成后，优雅的关闭运行在上面的应用程序，然后再另外一个DEA上重新启动。为了避免在Cloud Foundry升级过程中应用程序不可访问的风险，你应当运行多个应用程序的实例。

###使用Buildpacks

一个buildpack是由一系列检测和配置脚本组成，它为应用程序提供运行所有需要的框架和运行时。当你部署应用程序时，Cloud Foundry会把buildpack安装在运行应用程序的微执行代理(Droplet Execution Agent, DEA)上.

想了解更多参见[Buildpacks](http://docs.cloudfoundry.org/buildpacks/)主题。
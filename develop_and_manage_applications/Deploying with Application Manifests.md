##使用部署清单部署应用

*此文章假设你使用的cf命令行工具(v6)*

应用程序部署清单告知```cf push```应该如何配置应用程序。包括需要创建的实例数量，需要分配的内存大小以及需要使用的所有服务。

部署清单文件能够帮助你做自动化的部署，是部署过程更为简单，特别是一次性部署多个应用程序的时候。

###cf push如何找到部署清单文件

默认情况下，```cf push```命令在部署应用程序的时候会在当前工作目录下寻找```manifest.xml```文件。也可以使用```-f```选项指定部署清单文件的路径跟文件名。

在使用```cf push```时，如果不提供```-f```选项，那么你必须在当前工作目录文件夹下提供一个名为```manifest.xml```的文件。

```
$ cf push
Using manifest file /path_to_working_directory/manifest.yml
```

使用```-f```选项，跟路径但不包含文件名时，此路径目录下必须存在名为```manifest.xml```的文件。

```
$ cf push -f ./some_directory/some_other_directory/
Using manifest file /path_to_working_directory/some_directory/some_other_directory/manifest.yml
```

如果部署清单文件名不是```manifest.xml```，在使用```-f```选项时，必须同时指定路径和 文件名。

```
$ cf push -f ./some_directory/some_other_directory/alternate_manifest.yml
Using manifest file /path_to_working_directory/some_directory/some_other_directory/alternate_manifest.yml
```

###部署清单例子

你可以在部署应用程序的时候不适用部署清单。使用部署清单的好处主要有保持每次部署的一致性和重用。如果你想应用程序的部署在不同云平台上兼容的话，部署清单文件就特别有用。

部署清单文件使用Yaml格式。下面的部署清单文件简要说明了Yaml的书写规范：

* 部署清单文件以三个中划线开始。
* 第一个```applications```配置后面跟随冒号。
* 应用程序的名称```name```以一个中划线和空格开始。
* 其他属性以两个空格开始与```name```对齐。

部署清单最小的配置只需要应用程序的名称```name```就够了。删除掉以上例子的```memory```和```host```配置只剩下```name```，它依然是一个合法的部署清单文件。

把```manifest```文件放在与应用程序文件的同一个目录下面，然后再执行```cf push```。如果你不适用```-p```参数指定应用程序路径的话，```cf push```会试图在当前文件夹下面寻找```manifest.xml```文件。

###cf push时总是提供应用程序名称

命令```cf push```需要应用程序名称，可以在部署清单文件或者命令行里面提供。

如[./Deploying with Application Manifests.md#cf push如何找到部署清单文件]上所述，命令```cf push```只有在文件```manifest.xml```存在于当前工作目录文件夹下才能工作。

如果你不适用部署清单文件，最简单的部署命令如下：

```
cf push my-app
```

> 注意：当在命令行提供应用程序的时候，```cf push```会使用此应用程序名称而不是部署清单文件里面的应用程序名称。如果部署清单文件里面配置了多个应用程序，在部署的时候你可以在命令行指定要部署的某个应用程序名称来部署单个应用程序，```cf```命令只会部署你指定的那个应用程序而不会部署其他的。在做测试的时候这些功能很有用。

###cf push如何找到应用程序

默认情况下，```cf push```会递归的部署当前工作目录文件夹下的所有内容。或者，你可以通过部署清单文件或者命令行选项指定一个要部署文件的路径。

* 如果路径指向某个目录，```cf push```就会递归的部署指定目录文件夹下的所有内容而不是当前目录。
* 如果路径指向某个文件，```cf push```就只会部署路径指定的文件。

>注意：如果你想部署多个文件但是又不是某个目录下的所有文件，你可以使用```.cfignore```文件配置在```cf push```执行时需要排除的文件。

###部署清单文件、命令行选项及最近一次部署默认值的优先级

当你第一次部署某个应用程序的时候，如果你没有使用部署清单文件或者```cf push```命令行选项的话，Cloud Foundry会给所有的属性设置默认值。

* 例如，在不提供部署清单文件情况下，执行```cf push my-app```部署应用程序到某个实例，在这种情况下，服务实例的数量为一，内存大小为1G。

多次部署过程中，那些属性值也可以改变。

* 例如，```cf scale```命令可以改变服务实例的数量。

任何时候，服务器上这些属性的值都是由默认值，部署清单中的值，```cf push```命令行中使用选项的值以及某些命令如```cf scale```修改后的值的组合。没有一个合适的名字来描述这些值的状态。你可以认为他们最近改变的值。

命令```cf push```在设置属性值时，遵循以下规则：

* 部署清单配置的值会覆盖掉最近改变的值以及任何默认的值。
* 命令行选项的值会覆盖掉部署清单配置的值。

一般来说，你可以认为部署清单文件就是```cf push```命令的另外一种输入配置方式，合并命令行选项以及最近改变的值。

###可选属性

此处讲解释如何在部署清单文件里面使用某些可选参数配置应用程序。任何一个可选属性都可以以命令行选项的方式提供。命令行选项会覆盖掉在部署清单文件中对应的属性值。

####buildpack属性

如果你的应用程序需要自定义的buildpack，你可以使用```buildpack```属性在制定其名称或者URL：

```
---
  ...
  buildpack: buildpack_URL
```

>注意：命令```cf buildpacks```列出所有你可以在部署清单文件或者命令行选项中可以使用的buildpack。

使用命令行选项覆盖buildpack的属性是```-b```。	
####命令行属性

某些开发语言或者框架需要使用自定义的命令在启动应用程序。是否需要提供自定义的启动参数，请参考[buildpack](http://docs.cloudfoundry.org/buildpacks/)文档。

你也可以在部署清单文件或者命令行选项中提供自定义的启动命令。

在应用程序的部署清单文件中添加特定的启动命令，在```command: START-COMMAND```上添加，如下：

```
---
  ...
  command: bundle exec rake VERBOSE=true
```

对于命令行选项，使用```-c```指定自定义的启动命令，如下：

```
$ cf push my-app -c "bundle exec rake VERBOSE=true"
```

>注意：在使用```cf push```命令时，如果```-c```选项值为‘null’，则会使用buildpack的启动命令。更多信息参考[关于启动应用](../cf_command_line_interface/About Starting Applications.md)

####磁盘配额(disk_quota)属性

使用```disk_quota```属性指定需要给应用实例分配的磁盘大小。需要同时给定单位：```M```，```MB```，```G```或者```GB```，大小写都可以。

```
---
  ...
  disk_quota: 1024M
```

在命令行选项里面使用```-k```覆盖相应的配置。

####域名(domain)属性

每一个使用```cf push```部署应用程序到特定Cloud Foundry服务实例，每一个Cloud Foundry服务实例可能都有一个有管理员配置的一个共享的域名。如果你不修改此默认域名，Cloud Foundry路由就会使用此域名把请求分发到你的应用程序。

你可以使用```domain```属性自定义你想要的域名而不使用默认域名。

```
---
  ...
  domain: unique-example.com
```

在命令行选项里面使用```-d```覆盖相应的配置。

####多个域名(domains)属性

使用```domains```属性指定多个域名配置。如果你同时使用```domain```和```domains```属性，Cloud Foundry会使用这两种配置的域名来做路由分发。

```
---
  ...
  domains:
  - domain-example1.com
  - domain-example2.org
```

命令行选项里面使用```-d```覆盖相应的配置。

####服务实例(instances)属性

使用```instances```属性指定在部署的时候需要创建的应用服务实例的数量：

```
---
  ...
  instances: 2
```

为了提供更好的容错机制，我们建议你至少为一个应用程序创建两个服务实例。

命令行选项里面使用```-i```覆盖相应的配置。

####内存(memory)属性

使用```memory```属性指定需要给应用实例分配的内存大小。需要同时给定单位：M，MB，G或者GB，大小写都可以。

```
---
  ...
  memory: 1024M
```

默认的内存大小限制为1G. 如果你的应用实例不需要1G大小的内存空间，你可以定义一个更小的配额空间。

命令行选项里面使用```-m```覆盖相应的配置。

####主机(host)属性

使用```host```提供主机名称或者子域名属性，提供主机名称可以保证路由的唯一性。如果你不提供主机名称，应用程序的URL则有```APP-NAME.DOMAIN```组成。

```
---
  ...
  host: my-app
```

命令行选项里面使用```-n```覆盖相应的配置。

####多个主机(hosts)属性

使用```hosts```提供多个主机名称或者子域名配置。对于每一个主机名称都会为应用生成唯一的路由。```hosts```可以结合```host```使用。如果你同时定义它们，Cloud Foundry会同时使用它们生成对应路由。

```
---
  ...
  hosts:
  - app_host1
  - app_host2
```

命令行选项里面使用```-n```覆盖相应的配置。

####无主机名(no-hostname)属性

默认情况下，如果你不提供主机名称，应用程序的URL则由```APP-NAME.DOMAIN```组成。如果你想覆盖并使用跟域名映射到应用程序，你可以设置```no-hostname```值为```true```。

```
---
  ...
  no-hostname: true
```

命令行选项里面使用```--no-hostname```覆盖相应的配置。


####随机路由(random-route)属性

使用```random-raoute```属性创建应用的URL包括应用程序名称和随机的字符。使用这个属性来避免在部署同一个应用程序的时候URL冲突。

命令行选项里面使用```--random-route```覆盖相应的配置。

```
---
  ...
  random-route: true
```

####路径(path)属性

使用```path```属性告知Cloud Foundry去哪里找应用程序。如果你的应用程序在你执行```cf push```命令工作目录之下，就不需要此配置。

```
---
  ...
  path: path_to_application_bits
```

命令行选项里面使用```-p```覆盖相应的配置。

####超时(timeout)属性

使用```timeout```属性指定应用程序启动需要的最大时间，最大180秒。默认值为60秒。例如：

```
---
  ...
  timeout: 80
```

如果你的应用程序在设置的超时时间之内没能启动，Cloud Foundry则为不在尝试启动此应用程序。在部署大型应用时，你可以延长超时的时间。

命令行选项里面使用```-t```覆盖相应的配置。

####无路由(no-route)属性

默认情况下，```cf push```会为每个应用程序分配路由。某些应用程序只需要在后台处理一些数据，并不需要分配路由。

如果你不想给应用程序分配路由，你可以设置```no-route```属性为```true```。

```
---
  ...
  no-route: true
```

命令行选项里面使用```--no-route```覆盖相应的配置。

如果你发现某个应用程序具有本不应该有的路由：

1. 使用```cf unmap-route```命令删除分配的路由。
2. 设置部署清单文件属性```no-route: true```或者指定命令行选项```--no-route```，然后重新部署应用。

更多信息，参见[使用部署清单部署多个应用程序](./Deploying with Application Manifests.md/#使用部署清单部署多个应用程序)

####环境变量

配置```env```项，包括env头和一个或多个环境变量键值对。

例如：

```
---
  ...
  env:
    RAILS_ENV: production
    RACK_ENV: production
```

命令```cf push```部署应用程序到服务器容器里。这些变量是属于容器的环境变量。

当应用程序运行时，Cloud Foundry允许你改变这些环境变量。

* 查看所有的变量：```cf env my-app```
* 设置某个变量：```cf set-env my-variable_name my-variable_value```
* 重置某个变量：`cf unset-env my-variable_name my-variable_value`

在部署清单文件里定义的环境变量在以下情况起作用：

* 当你第一次部署应用程序时，Cloud Foundry会读取部署清单文件里面的环境变量，然后添加这些环境变量到要部署应用的容器。
* 当你暂停或者重启应用程序时，所有的环境变量都以存储。

####服务

应用程序可以绑定服务例如数据库，消息队列和键值存储器。

应用程序部署到程序空间。应用程序部署之后可以绑定任何存在于应用空间的服务。

配置`services`项，包括services头和一个或多个服务名称。

无论谁创建这些服务并选择这些服务实例名字。这些名字传达逻辑信息，例如`backend_queue`，名字就表明它的用途，如`mysql_5.x`也是如此，例子如下：

```
---
  ...
  services:
   - instance_ABC
   - instance_XYZ
```

绑定的服务信息都存储在`VCAP_SERVICES`环境变量里。参见[绑定服务](http://docs.cloudfoundry.org/devguide/services/bind-service.html)。

###使用部署清单部署多个应用程序

在执行`cf push`命令时，可以在部署清单文件里面定义多个需要部署的应用程序。在你这样做时，你要留意在部署清单文件中定义的应用程序目录结构和路径。

假设你要部署两个应用程序分别叫`spark`和`flame`，然后你想Cloud Foundry先创建并启动`spark`。你只需要把`spark`定义在前面即可。

在这种情况下，你有两个应用程序需要部署。假设他们分别是`spark.rb`位于spark目录和`flame.rb`位于flame目录。在上级目录，fireplace目录包含spark和flame目录还有`manifest.xml`部署清单文件。为了找到部署清单文件，你希望在frieplace目录运行`cf`命令。

现在如果你改变了目录结构和部署清单文件的路径，默认情况`cf push`不能在当前工作目录下找到你的应用程序。如果确保`cf push`在部署的时候找到你想部署的应用程序路径。

解决方案是在使用`cf push`时为每个应用程序添加路径配置。假设在`fireplace`目录下运行`cf push`。

对于`spark`：

```
---
  ...
  path: ./spark/
```

对于`flame`：

```
---
  ...
  path: ./flame/
```

部署 清单文件包括这两个应用程序的配置。

```
---
# 此部署清单部署两个应用程序
# 应用程序分别在flame和spark目录下
# flame和spark在fireplace目录下
# cf push应当自fireplace目录下运行
applications:
- name: spark
  memory: 1G
  instances: 2
  host: flint-99
  domain: example.com
  path: ./spark/
  services:
  - mysql-flint-99
- name: flame
  memory: 1G
  instances: 2
  host: burnin-77
  domain: example.com
  path: ./flame/
  services:
  - redis-burnin-77
```

在部署清单文件使用多个应用程序配置时遵守以下规则：

* 应用程序名称具有自描述性。
* 如果某个程序是另外某个程序的后台服务的时候，也就是说它不需要路由，这时候可以使用`no-route`配置。
* 应用程序名称不能是`cf push`。
* 任何命令行选项不能是`cf push`。

这里有两个特殊情况：

* 如果你的部署清单文件名不是`manifest.xml`或者不在当前工作目录下，则使用`-f`命令行选项。
* 如果你只想部署在部署清单文件中的定义的某一个应用程序，只需要在执行部署命令时提供应用程序名称即可`cf push my-app`。

###避免重复

在部署清单文件里面多个应用程序可以共享所有通用的服务配置，你可以看到某些内容是重复的。然而部署清单文件任然可以使用，重复可能会导致排版错误的增加，进而导致应用程序部署失败。

解决此问题办法是把所有重复并只需要出现一次的内部移出来并放在`applications`的上面。被提出来的内容可以提所有应用程序使用。注意在应用程序里面定义的所有一样的配置会覆盖掉上面的配置。

这样的话部署清单文件就变得更加的短小易读和更容易维护。

注意下面的部署清单文件有多少内容可以提出来。

```
---
  ...
# all applications use these settings and services
domain: example.com
memory: 1G
instances: 1
services:
- clockwork-mysql
applications:
- name: springtock
  host: tock09876
  path: ./spring-music/build/libs/spring-music.war
- name: springtick
  host: tick09875
  path: ./spring-music/build/libs/spring-music.war
```

在下面的章节我们会继续讨论如果在部署清单文件里面共享配置。

###继承部署清单文件


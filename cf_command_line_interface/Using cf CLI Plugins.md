##使用cf命令行工具的插件

Cloud Foundry命令行工具版本v.6.7以上的版本都支持插件。这些插件可以帮助开发人员在cf命令行工具中添加自定义的命令行。你可以同时使用来自Cloud Foundry developers或者任何第三方开发的插件。你可以在[Cloud Foundry Community CLI Plugin page](http://plugins.cloudfoundry.org/ui/)查看当前社区所支持的所有插件。在[Cloud Foundry CLI Plugin Repository(CLIPR) on GitHub](https://github.com/cloudfoundry-incubator/cli-plugin-repo)上查阅有关如何提交自己的插件。

cf命令行工具通过二进制文件名识别插件，插件名称由其开发人员定义，命令行由此插件提供。二进制文件名只用作插件安装，插件名称或者命令行用来做其他操作。

>注意：	cf命令行工具对插件名称和命令行大小写敏感，对二进制文件名不受此限制。

###前置条件

cf命令行工具v.6.7以上的版本才能使用插件。参见[安装cf命令行工具](./Installing the cf Command Line Interface.md)的如何下载、安装及卸载主题。

###设置插件目录

默认情况下，cf插件的安装位置位于```$HOME/.cf/plugins```，如果想要改变插件的根目录路径```$HOME```，只需要设置```CF_PLUGIN_HOME```环境变量。cf命令行工具会在```CF_PLUGIN_HOME```路径后面添加```.cf/plugins```。

例如， 如果你把```CF_PLUGIN_HOME```设置为```/my-folder```，cf命令行工具存储插件的位置就变成为```/my-folder/.cf/plugins```。

###插件安装

1. 从可信的地方下载插件的二进制文件或者源代码。
	>注意：cf命令行工具只支持用Go语言编写的二进制文件。如果你下载的是源代码 ，你必须先编译生成二进制文件。
	
2. 执行```cf install-plugin BINARY_FILENAME```安装你需要的插件，把**BINARY——FILENAME**换成你二进制文件的全路径。
	>注意：你不能安装已经具有相同插件名称或者相同命令行名称的插件。如果需要，你必须先删除已有的插件。
	
	>注意：cf命令行工具无法安装任何与cf内置命令行名称或者别名冲突的插件。例如，如果你试图安装某个具有```cf push```命令的第三方插件，安装会失败。
	
###运行插件命令行

参考运行```cf help PLUGIN或者PLUGIN COMMANDS```的内容来管理和运行插件的命令行、

1. 执行```cf plugins```列出已安装的所有插件的命令行。
2. 执行```cf PLUGIN_COMMAND```，**PLUGIN_COMMAND**为特定插件的命令行名称。

###卸载插件

使用**插件名称**卸载插件而不是二进制文件名。

1. 执行```cf plugins```查看所有已安装的插件。
2. 执行```cf uninstall-plugin PLUGIN_NAME```删除特定的插件。

####添加插件仓库

执行```cf add-plugin-repo REPO_NAME URL```添加插件仓库。

例如：

```
$ cf add-plugin-repo CF-Community http://plugins.cloudfoundry.org
OK
http://plugins.cloudfoundry.org/list added as 'CF-Community'

```

####列出所有可用的插件仓库

执行```cf list-plugin-repos```查看所有可用的插件仓库。

例如：

```
$ cf list-plugin-repos
OK


Repo Name      Url

CF-Community   http://plugins.cloudfoundry.org 

```

###按照仓库分类列出所有的插件

执行```cf repo-plugins```查看所有仓库的所有插件。

###故障排除

cf命令行工具提供以下错误消息帮助你理解并进一步解决在安装或者使用插件过程中遇到的问题，第三方插件的错误消息由其自行提供。

####拒绝访问

如果你收到```permission denied```错误消息，说明你没有运行此插件的权限。你必须拥有对插件的二进制文件读和执行的权限。

####插件命令行冲突

插件的名称和命令行必须唯一。如果你试图安装与其他插件相冲突的插件cf命令行工具会报错。

如果发生冲突，必须先卸载已有的插件，然后再进行安装新插件。

如果待安装的插件命令行与原生cf命令行工具相冲突，此插件不可安装。
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
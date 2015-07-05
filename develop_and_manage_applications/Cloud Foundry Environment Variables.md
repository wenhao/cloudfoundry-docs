##Cloud Foundry环境变量

*此文章假设你使用的cf命令行工具(v6)*

环境变量是用于Cloud Foundry运行时与已部署的应用程序通信。此文章将讲述Droplet Execution Agents(DEAs)和buildpacks如何使用环境变量设置应用程序。

更多关于为特定的某个应用程序设置环境变量，参见[环境变量](./Deploying with Application Manifests.md/#环境变量)

###查看环境变量

使用`cf env`命令在Cloud Foundry中查看应用程序多有的环境变量。`cf env`会显示一下环境变量：

* 环境变量`VCAP_SERVICES`属于容器。
* 使用`cf set-env`命令设置用户自定义变量。

```
$ cf env my-app
  Getting env variables for app my-app in org my-org / space my-space as admin...
  OK
  
  System-Provided:
  
  
  {
   "VCAP_APPLICATION": {
    "application_name": "my-app",
    "application_uris": [
     "my-app.10.244.0.34.xip.io"
    ],
    "application_version": "fb8fbcc6-8d58-479e-bcc7-3b4ce5a7f0ca",
    "limits": {
    "disk": 1024,
    "fds": 16384,
    "mem": 256
    },
    "name": "my-app",
    "space_id": "06450c72-4669-4dc6-8096-45f9777db68a",
    "space_name": "my-space",
    "uris": [
     "my-app.10.244.0.34.xip.io"
    ],
    "users": null,
    "version": "fb8fbcc6-8d58-479e-bcc7-3b4ce5a7f0ca"
    }
  }
  
  User-Provided:
  MY_DRAIN: http://drain.example.com
  MY_ENV_VARIABLE: 100
```

###应用程序的环境变量

此子章节讲述在Cloud Foundry中应用程序容器可以使用的环境变量。

你可以通过编程的方式访问这些环境变量，也包括buildpack定义的所有环境变量。参见buildpack的文档[Java]()、[Node.js]()和[Ruby]().

####主目录(HOME)

应用程序部署的根目录。

```
HOME=/home/vcap/app
```

内存限制(MEMORY_LIMIT)

应用程序实例可以使用的最大内存大小。你可以通过应用程序部署清单文件或者命令行选项指定其他值。内存大小受限于空间和组织配额。

如果实例运行内存大与此值，应用程序讲自动重启。如果多次启动，此实例将被删除。

```
MEMORY_LIMIT=512m
```

####端口号(PORT)


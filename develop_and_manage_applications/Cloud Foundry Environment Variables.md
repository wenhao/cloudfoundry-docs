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

此端口是用于DEA和应用程序通信。在应用程序预处理时，DEA会创建此端口。应用程序实例也是用此端口号。

```
PORT=61857
```

####当前工作目录(PWD)

显示当前工作目录，它是buildpack和应用程序运行的目录。

```
PWD=/home/vcap
```

####临时目录(TMPDIR)

临时目录和预处理文件存储的目录。

```
TMPDIR=/home/vcap/tem
```

####用户(USER)

DEA运行的用户。

```
USER=vcap
```

####DEA的IP地址(VCAP_APP_HOST)

DEA的IP地址。

```
VCAP_APP_HOST=0.0.0.0
```

####应用程序信息(VCAP_APPLICATION)

此变量包含部署了得应用程序所有信息。返回JSON格式的数据。下表展示所有的属性。

属性 | 描述
---------- | ----------
`application_users, users`|
`instance_id` | 应用程序唯一标示符
`instance_index` | 程序实例的索引值。你可以使用[CF_INSTANCE_INDEX]()查看此值。
`application_version, version` | 部署应用程序版本的唯一标示符，每次应用程序部署，此值都会更新。
`application_name, name` | 部署后应用程序的名称。
`application_urls` | 应用程序的URL。
`started_at, start` | 应用程序最近启动的时间。
`started_at_timestamp` | 应用程序最近启动的时间戳。
`host` | 应用程序实例的IP地址。
`port` | 应用程序实例的端口地址。你可以通过[端口](./Cloud Foundry Environment Variables.md/#端口(PORT))环境变量获取。
`limits` | 应用程序实例内存、磁盘和文件数量的限制。在应用程序部署时，会使用在命令行选项或者部署清单文件定义的内存和磁盘大小限制。文件数量是由管理员定义。
`state_timestamp` | 应用程序获取某个状态的时间戳。

例如：

```
VCAP_APPLICATION={"instance_id":"451f045fd16427bb99c895a2649b7b2a",
"instance_index":0,"host":"0.0.0.0","port":61857,"started_at":"2013-08-12
00:05:29 +0000","started_at_timestamp":1376265929,"start":"2013-08-12 00:05:29
+0000","state_timestamp":1376265929,"limits":{"mem":512,"disk":1024,"fds":16384}
,"application_version":"c1063c1c-40b9-434e-a797-db240b587d32","application_name"
:"styx-james","application_uris":["styx-james.a1-app.cf-app.com"],"version":"c10
63c1c-40b9-434e-a797-db240b587d32","name":"styx-james","uris":["styx-james.a1-ap
p.cf-app.com"],"users":null}
```

####弃用的端口变量(VCAP_APP_PORT)

已弃用，使用上面的[端口](./Cloud Foundry Environment Variables.md/#端口(PORT))环境变量。

####服务(VCAP_SERVICES)



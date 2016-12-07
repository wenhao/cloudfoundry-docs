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

绑定服务实例到应用程序之后，应用程序重启时，Cloud Foundry会把服务连接信息写入`VCAP_SERVICES`环境变量里面。

返回的JSON文档里面包括绑定到每个应用程序的一个或者多个服务信息。服务对象的子对象是绑定到应用程序的服务实例。下表详细描述了绑定服务的所有属性。

The key for each service in the JSON document is the same as the value of the “label” attribute.

在JSON文档里面描述的每一个服务的键和“label”属性的值是一样的。

属性 | 描述
-----|----
`name`|用户创建实例服务的名字。
`label`|**v1 broker API** 服务名称和服务版本号(如果没有版本号属性，字符串“n/a”就会被用到)，使用中划线链接，例如：“cleardb-n/a”。**v2 broker API** 服务名称。
`tags`|服务可以由一组字符串来鉴别。
`plain`|服务创建之后选择的计划。
`credentials`|以JSON对象的形式包含了一系列访问服务实例使用到的一些认证信息。

查看Cloud Foundry中VCAP_SERVICES变量所有的值，参见[查看环境变量](./Cloud Foundry Environment Variables.md/#查看环境变量)。

下列例子展示了在[Pivotal服务]()商店绑定了多个服务之后，以[版本v1格式]()显示VCAP_SERVICES变量的值。

```
VCAP_SERVICES=
{
  "elephantsql-n/a": [
    {
      "name": "elephantsql-c6c60",
      "label": "elephantsql-n/a",
      "tags": [
        "postgres",
        "postgresql",
        "relational"
      ],
      "plan": "turtle",
      "credentials": {
        "uri": "postgres://seilbmbd:PHxTPJSbkcDakfK4cYwXHiIX9Q8p5Bxn@babar.elephantsql.com:5432/seilbmbd"
      }
    }
  ],
  "sendgrid-n/a": [
    {
      "name": "mysendgrid",
      "label": "sendgrid-n/a",
      "tags": [
        "smtp"
      ],
      "plan": "free",
      "credentials": {
        "hostname": "smtp.sendgrid.net",
        "username": "QvsXMbJ3rK",
        "password": "HCHMOYluTv"
      }
    }
  ]
}
```

以[版本v2格式](http://docs.cloudfoundry.org/services/api.html)展示同一个服务如下：

```

VCAP_SERVICES=
{
  "elephantsql": [
    {
      "name": "elephantsql-c6c60",
      "label": "elephantsql",
      "tags": [
        "postgres",
        "postgresql",
        "relational"
      ],
      "plan": "turtle",
      "credentials": {
        "uri": "postgres://seilbmbd:PHxTPJSbkcDakfK4cYwXHiIX9Q8p5Bxn@babar.elephantsql.com:5432/seilbmbd"
      }
    }
  ],
  "sendgrid": [
    {
      "name": "mysendgrid",
      "label": "sendgrid",
      "tags": [
        "smtp"
      ],
      "plan": "free",
      "credentials": {
        "hostname": "smtp.sendgrid.net",
        "username": "QvsXMbJ3rK",
        "password": "HCHMOYluTv"
      }
    }
  ]
}
```
####应用程序实例变量

每一个应用程序的实例都可以访问诸多以`CF_INSTANCE`开头的变量。这些变量提供某个应用程序实例的一些基本信息，包含DEA的IP等地址。

#####`CF_INSTANCE_ADDR`

`CF_INSTANCE_IP`和`CF_INSTANCE_PORT`格式如下：

```
particular-DEA-IP:particular-app-instance-port
```

```
CF_INSTANCE_ADDR=1.2.3.4:5678
```

#####`CF_INSTANCE_INDEX`

应用程序的索引值。

```
CF_INSTANCE_INDEX=0
```

#####`CF_INSTANCE_IP`

运行应用程序实例容器DEA的外部访问地址。

```
CF_INSTANCE_IP=1.2.3.4
```

#####`CF_INSTANCE_PORT`

应用程序实例的端口号。

```
CF_INSTANCE_PORT=5678
```

#####`CF_INSTANCE_PORTS`

分配给应用程序实例的外和内部端口号。

```
CF_INSTANCE_PORTS=[{external:5678,internal:5678}]
```

####环境变量组

环境变量组是系统级变量，管理人员可以用它分别定义应用程序运行或预处理时需要使用的不同环境变量。

一个环境变量组有单个键值对组成，会在应用程序运行或者预处理时加入到程序的容器里。这些值包含如HTTP代理等信息。环境变量组的值是大小写敏感的。

当我们创建环境变量组时，需要考虑以下几点：

* 只有Cloud Foundry管理员可以设置环境变量组的值。
* 所有验证通过的用户都可以访问应用程序的所有环境变量。
* 所有环境变量更改都会在管理员重启或者重新预处理应用程序之后起效。
* 任何用户自定义的环境变量会优先于环境变量组。

下表列出了一些环境变量组。

命令行|描述
-----|-----
`running-environment-variable-group`, `revg`|查询运行时环境变量组的内容。
`staging-environment-variable-group`, `sevg`|查询预处理时环境变量组的内容。
`set-staging-environment-variable-group`, `ssevg`|以JSON格式传参数创建预处理环境变量组。
`set-running-environment-variable-group`, `srevg`|以JSON格式传参数创建运行时环境变量组。

下面的例子解释如何获取环境变量：

```
$ cf revg
  Retrieving the contents of the running environment variable group as sampledeveloper@pivotal.io...
  OK
  Variable Name   Assigned Value
  HTTP Proxy      87.226.68.130


$ cf sevg
  Retrieving the contents of the staging environment variable group as sampledeveloper@pivotal.io...
  OK
  Variable Name   Assigned Value
  HTTP Proxy      27.145.145.105
  EXAMPLE-GROUP   2001



$ cf apps
  Getting apps in org SAMPLE-ORG-NAME / space dev as sampledeveloper@pivotal.io...
  OK



name                                               requested state   instances   memory   disk   urls
  APP-NAME                                         started           1/1         256M     1G     APP-NAME.cf-app.com, test-foo.com



$ cf env APP-NAME
  Getting env variables for app APP-NAME in org SAMPLE-ORG-NAME / space dev as sampledeveloper@pivotal.io...
  OK



System-Provided:



{
   "VCAP_APPLICATION": {
    "application_name": "APP-NAME",
    "application_uris": [
     "APP-NAME.sample-app.com",
     "test-foo.com"
    ],
    "application_version": "7d0d64be-7f6f-406a-9d21-504643147d63",
    "limits": {
     "disk": 1024,
     "fds": 16384,
     "mem": 256
    },
    "name": "APP-NAME",
    "space_id": "37189599-2407-9946-865e-8ebd0e2df89a",
    "space_name": "dev",
    "uris": [
     "APP-NAME.sample-app.com",
     "test-foo.com"
    ],
    "users": null,
    "version": "7d0d64be-7f6f-406a-9d21-504643147d63"
   }
  }



Running Environment Variable Groups:
  HTTP Proxy: 87.226.68.130



Staging Environment Variable Groups:
  EXAMPLE-GROUP: 2001
  HTTP Proxy: 27.145.145.105
  
```

下面的例子解释如何设置环境变量：

```
$ cf ssevg '{"test":"87.226.68.130","test2":"27.145.145.105"}'
  Setting the contents of the staging environment variable group as admin...
  OK
  $ cf sevg
  Retrieving the contents of the staging environment variable group as admin...
  OK
  Variable Name   Assigned Value
  test            87.226.68.130
  test2           27.145.145.105


$ cf srevg '{"test3":"2001","test4":"2010"}'
  Setting the contents of the running environment variable group as admin...
  OK
  $ cf revg
  Retrieving the contents of the running environment variable group as admin...
  OK
  Variable Name   Assigned Value
  test3           2001
  test4           2010
```

##识别Cloud Foundry服务的API地址

API地址或者目标地址为Cloud Foundry实例中Cloud Controller的URL地址。通过质询管理员或者命令行工具可以找到。

###通过命令行

通过命令行，可以使用```cf api```命令可以查看API地址。

例如：

```
$ cf api
API endpoint: https://api.example.com (API version: 2.2.0)
```
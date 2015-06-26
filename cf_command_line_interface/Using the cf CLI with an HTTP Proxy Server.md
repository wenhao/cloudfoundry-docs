##cf命令行工具使用HTTP代理

如果在运行cf的主机和Cloud Foundry API端口之间，你的网络设置有HTTP代理，你必须设置你的```HTTP_PROXY```为你的代理服务器的主机名称或者IP地址。

环境变量```HTTP_PROXY```是设置你代理服务器的主机名称或者IP地址。

```HTTP_PROXY```是一个标准的环境变量。与其他环境变量一样，设置的方法取决于特定的操作系统。

###HTTP_PROXY的格式

HTTP_PROXY是用代理服务器的主机名称或者IP地址写成URL的格式：

``` 
http_proxy=http://proxy.example.com
```

如果代理服务器需要用户名及密码登陆，必须包含登陆信息：

```
http_proxy=http://username:password@proxy.example.com
```

如果代理服务器用到80以外的端口，必须包含端口信息：

```
http_proxy=http://username:password@proxy.example.com:8080
```

###Mac OS或Linux设置代理

在不同的shell中使用不同的方式设置HTTP_PROXY环境变量的值。例如，在bash中使用```export```命令。

例如：

```
$ export HTTP_PROXY=http://my.proxyserver.com:8080
```

永久的改变此变量的值，只需要把此代码加到特定的文件里面即可。例如，在bash中，找到**.bash_profile**或者**.bashrc**文件，加入如下代码：

```
http_proxy=http://username:password@hostname:port;
export $HTTP_PROXY
```

###Windows设置代理

1. 点击任务栏左下角**开始**按钮，右键单击**我的电脑**并选择**属性**。

2. 在弹出来的控制面板窗口左侧，选择**高级系统设置**。

3. 在弹出来的系统属性窗口上，选择**高级**标签，然后点击**环境变量**设置。

4. 在弹出来的环境变量窗口上，找到**用户变量设置**，然后点击**创建**。

5. 在弹出来的窗口，输入新加入环境变量的名字**HTTP_PROXY**并设置其值。

6. 最后单击**确定**保存，设置生效。

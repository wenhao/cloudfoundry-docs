##如何打包应用程序

![应用程序预处理时序图](../images/app_push_flow_diagram.png)

1. 在命令行里，开发人员进入应用程序的目录并使用cf命令行工具执行`cf push`命令。
2. 此命令会通知Cloud Controller为此应用程序创建记录信息。
3. Cloud Controller保存应用程序的元数据信息(例如：应用程序名称，用户定义的实例数量和buildpack)。
4. 此命令会上传应用程序文件。
5. Cloud Controller在大数据存储中存储应用程序的字节文件。
6. 此命令会发起应用程序启动指令。
7. 由于此应用程序未被打包过，Cloud Controller就会在DEA池里面选择一个合适的DEA实例来打包应用程序。DEA会根据buildpack来打包应用程序。
8. DEA打包过程中会输出所有的打包信息以便开发人员做故障排除。
9. 应用程序打包好之后会成为一个“droplet”，存储在大数据存储之中。结果会被缓存下来以供下次直接使用。
10. DEA打包通知Cloud Controller打包完成。
11. Cloud Controller会从DEA池中选择一个或者多个来运行已经打包好的应用程序。
12. 运行着的DEA会时刻通知Cloud Controller应用程序的状态。

##安装cf命令行工具

安装cf命令行工具，下载[https://github.com/cloudfoundry/cli/releases](https://github.com/cloudfoundry/cli/releases)对应的安装包并依照安装向导完成安装。

###卸载cf(v5)
如果你之前安装过老版本的cf(v5)，你必须先卸载此Ruby gem，然后再安装新版本的cf(v6).

卸载cf，只需要执行``` gem uninstall cf ```

>注意：确保你的Ruby环境管理已经执行此变化，关闭后再重新打开你的终端。

###Windows安装
在Windows上安装cf：

1. 解压zip文件。
2. 双击可执行文件``` cf.exe ```
3. 当弹出确认对话框时，选择**Install**，然后关闭窗口。

###Mac OSX和Linux安装
1. 打开``` .pkg ```文件。
2. 出现安装向导后，选择**Continue**。
3. 选择要安装的目录并选择**Continue**。
4. 当弹出确认对话框时，选择**Install**。

###下一步
验证安装是否正确，打开你的终端并执行``` cf ```命令。如果你安装正确，屏幕上就会出现cf的使用说明。

想了解更多关于如何使用cf命令行工具(verions 6), 请参照[Getting Started with cf CLI v6](./Getting Started with the cf CLI.md)。
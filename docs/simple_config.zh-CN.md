# 简单配置

请考虑使用[结构化配置](advanced_config.md)。

这个文件是关于之前通过不同文件的配置方法，使用 `target_packages` 和 `injected_libraries` 并且不支持所有
功能。

`/data/local/tmp/re.zyg.fri/target_packages` 
是一个简单的文本文件，包含您想要将 frida 注入其中的所有包名称。

每行接受一个包名称，例如
````
adb shell 'su -c "echo com.example.package > /data/local/tmp/re.zyg.fri/target_packages"'
````

**启动延迟**

有时您可能想要延迟小工具的注入。
一些应用可能会在启动时运行检查，延迟注入可以帮助避免这些情况。

`/data/local/tmp/re.zyg.fri/target_packages` 
接受以毫秒（ms）为单位的启动延迟。
您可以设置该参数，并与 package_name 之间用英文逗号分隔。

举例：
````
adb shell 'su -c "echo com.example.package,20000 > /data/local/tmp/re.zyg.fri/target_packages"'
````
会在 20 秒后注入小工具。

您可以在 ZygiskFrida（本模块） 日志 “adb logcat -S ZygiskFrida” 中看到注入的 10 秒倒计时。
如果您想通过应用程序交互来确定注入时间，这对于您会有所帮助。

**小工具版本和配置**

本模块捆绑的小工具（Gadget）位于 “/data/local/tmp/re.zyg.fri/libgadget.so” 。
您可以按照 [Gadget 官方文档](https://frida.re/docs/gadget/) 
并添加其他该位置中的小工具配置和脚本。

如果您想使用与捆绑版本*不同*的小工具版本，
您可以简单地将 `libgadget.so` 替换为您自己的 frida 小工具。

**加载任意库**

该模块还允许您将任意 so 库加载到进程中。
这可以让您为小工具加载额外的帮助程序库或
*启用*可能需要将库加载到应用程序进程中的任何其他用例。

为此，您可以添加文件 “ /data/local/tmp/re.zyg.fri/injected_libraries ”。
该文件应包含库的文件路径。
库按照文件中*指定*的顺序加载。

首先加载 libhelperexample.so，然后加载捆绑的 frida-gadget 的示例文件内容：

````
/data/local/tmp/re.zyg.fri/libhelperexample.so
/data/local/tmp/re.zyg.fri/libgadget.so
````

确保库位于应用程序可访问的位置
并且文件权限设置正确。

如果您希望启动 frida 小工具，则需要在以下位置*显式*指定捆绑的 frida-gadget
`/data/local/tmp/re.zyg.fri/libgadget.so`.
您还可以选择以这种方式指定您自己的小工具或完全忽略该小工具。

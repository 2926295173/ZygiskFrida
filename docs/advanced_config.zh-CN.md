# 高级配置

之前各种文件的配置方法,请参见[简单配置](simple_config.zh-CN.md)，目前仍有效。

但此处配置方法是**首选**的方法，未来将支持更多功能。

这两种配置均受支持，并且高级配置*优先*，以防应用程序同时出现在这两种配置中。

## 配置文件

该模块通过位于 “/data/local/tmp/re.zyg.fri/config.json” 的 json 配置进行配置。
首先，您可以复制示例配置

````shell
adb shell 'su -c cp /data/local/tmp/re.zyg.fri/config.json.example /data/local/tmp/re.zyg.fri/config.json'
````

配置示例
```json
{
    "targets": [
        {
            "app_name" : "com.example.package",
            "enabled": true,
            "start_up_delay_ms": 0,
            "injected_libraries": [
                {
                    "path": "/data/local/tmp/re.zyg.fri/libgadget.so"
                }
            ],
            "child_gating": {
                "enabled": false,
                "mode": "freeze",
                "injected_libraries" : [
                    {
                        "path": "/data/local/tmp/re.zyg.fri/libgadget-child.so"
                    }
                ]
            }
        }
    ]
}
````

该配置包含一系列目标。
一个目标包含一个应用程序（你想要注入的程序）的配置。

如果没有按预期工作，请检查 “ adb logcat -s ZygiskFrida ” 日志以查看是否记录了错误。

## 目标配置

### app_name
您想要注入 frida 的应用程序的包 ID （bundle id）。

### enabled
如果设置为 false，则模块将忽略此配置。
如果您想在维护配置的同时*暂时*禁用目标，这非常有用。

### start_up_delay_ms
注入延迟时间（以毫秒为单位）。

有时您可能想要延迟小工具的注入。 
一些应用可能会在启动时运行检查，延迟注入可以帮助避免这些情况。

### injected_libraries
这些是将要被注入到进程中的库（.so）。 
此处指定库的将按照数组的*顺序*加载。

该模块包含一个捆绑的 frida 小工具，位于 “ /data/local/tmp/re.zyg.fri/libgadget.so ”。
`libgadget.so` 默认架构始终是您设备的架构。

为了方便起见，该模块还在 “ /data/local/tmp/re.zyg.fri/libgadget32.so ” 安装了一个小工具，
用于注入32位的应用程序（但该模块仅支持64位的设备）。

您可以根据官方[Gadget 文档](https://frida.re/docs/gadget/)
调整小工具配置

如果您想使用不同的 frida 版本或替代版本，
您可以替换它以及您自己的小工具的路径。

使用此功能，您还可以将任意库与小工具一起注入，
但是如果删除小工具则不注入小工具。 
请确保您在此处提供的库具有正确的*文件权限*设置并且可由应用程序本身访问。

该模块将在安装时，在完整的“ re.zyg.fri ”目录中设置文件权限。
如果你怀疑存在文件权限问题，一个简单的检查方法是将您的库放在 `re.zyg.fri` 目录中
并*再次*安装该模块（无需卸载）。


## （Child gating） 子进程门控配置 (实验性)
这是一个实验性功能，有很多注意事项！ 请务必仔细阅读。

该模块能够拦截进程内的 fork/vfork 来检测子进程。 
因为应用程序可能会派生一个子进程来运行检查，如果没有子进程门控（Child gating），您将无法拦截这些检查。

通过将 `enabled` 设置为 true 来启用此功能，
您可以配置如何处理与这些子进程。

目前子进程门控（Child gating）的运作方式有 3 种。 
您可以通过以下方式确定，
将模式设置为“ freeze ”、“ kill ”或 “ inject ”。

使用任何子门控模式都可能导致即使强制关闭也无法正确关闭应用程序的问题。
这可能会导致重新启动应用程序时出现问题。 
手动终止该应用程序可以解决此问题。

````shell
adb shell 'su -c Kill -9 $(pidof com.example.package)'
````

### 冻结模式(freeze)
子进程不会从 fork 中返回。
这意味着没有代码会在子进程中运行，但进程本身保持活动状态。

### kill模式
子进程一旦被fork就会被杀死。 
没有代码会在子进程中运行。

### 注入模式（inject）
此模式会将 “ injected_libraries ” 注入到与目标配置类似的子进程中。
注入后，子进程将恢复其正常的代码流。
如果子进程只是进行快速检查并退出，您可能无法以**交互方式**连接到小工具。

请注意，当子进程被派生（fork）时，它已经包含父进程加载的所有库。
但是 ，由于只有一个线程从 fork 返回，因此加载的 frida gadget 线程 **不在** 子进程中。

重载相同的捆绑小工具将无法启动。 为此，您必须加载该小工具的副本。
您无法再次将相同的文件加载到进程中，因为符号链接将不起作用，所以它必须是副本。
例如：

````shell
adb shell 'su -c cp /data/local/tmp/re.zyg.fri/libgadget.so /data/local/tmp/re.zyg.fri/libgadget-child.so'
````

默认配置的 gadget 会因为与父进程中的 gadget 发生端口**冲突**而无法启动。
因此，对于子进程，您必须将小工具配置为使用**不同**的端口。

在 `/data/local/tmp/re.zyg.fri/libgadget-child.config.so` 创建一个像这样的小工具配置。
请参阅[小工具官方文档](https://frida.re/docs/gadget/) 以供参考。
````
{
  "interaction": {
    "type": "listen",
    "address": "127.0.0.1",
    "port": 27043,
    "on_port_conflict": "pick-next",
    "on_load": "wait"
  }
}
````

请注意 “ on_port_conflict: pick-next ” ，这个选项在父进程派生**很多**子进程时十分重要。

由于这是一个非默认端口小工具，
您可以通过 “ adb logcat -s Frida ” 来查看哪些端口子进程小工具开始了。

然后你可以连接它，例如
````shell
adb forward tcp:27043 tcp:27043
frida -H 127.0.0.1:27043 -n Gadget
````

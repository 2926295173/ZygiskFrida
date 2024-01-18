# ZygiskFrida - 一款 Magisk 的模块

> [Frida](https://frida.re) 是一个面向开发人员、逆向工程师和安全研究人员的动态检测工具包

> [Zygisk](https://github.com/topjohnwu/Magisk) Magisk 的一部分允许您在每个 Android 应用程序的进程中运行代码。


## 介绍

[ZygiskFrida](README.md) 是一个 zygisk 模块，允许您在 Android 应用程序中注入 frida 小工具
更隐蔽的方式。

- 该小工具未嵌入 APK 本身。 因此 APK 完整性/签名检查仍然会通过。
- 该进程没有像 frida-server 那样会被跟踪。 这避免了基于 ptrace 的检测。
- 控制小工具（frida gadget）的注入时间。
- 允许您将多个任意库加载到进程中。

这个存储库还提供了 [Riru](https://github.com/RikkaApps/Riru) 风格，以防您仍然
将 riru 与较旧的 Magisk 版本而不是 zygisk 一起使用。

## 如何使用该模块

### 快速开始
- 从[发布页面](https://github.com/lico-n/ZygiskFrida/releases)下载最新版本
   如果您使用 riru 而不是 zygisk，请选择 riru-release。 否则选择普通版本。
- 将 ZygiskFrida.zip 文件传输到您的设备并通过 Magisk 安装。
- 安装后重新启动
- 创建配置文件并将包名称调整为您的目标应用程序（替换命令中的“{your.target.application}”）
- 
````shell
adb shell 'su -c cp /data/local/tmp/re.zyg.fri/config.json.example /data/local/tmp/re.zyg.fri/config.json'
adb shell 'su -c sed -i s/com.example.package/{your.target.application}/ /data/local/tmp/re.zyg.fri/config.json'
````

- 启动您的应用程序。 它将在启动时暂停，以便您附加

````shell
frida -U -N {your.target.application}
或者
frida -U -n Gadget
````

【该模块*自带frida服务器*，且使用了默认端口，如您运行其他frida服务器，可能造成端口冲突。】

我们假设您没有运行任何其他 frida 服务器（例如没有使用 MagiskFrida）。
您仍然可以与 frida-server 一起运行它，但您必须配置该小工具
使用不同的端口。

### 配置

简单配置[简单配置](docs/simple_config.zh-CN.md)
该模块还支持添加*启动延迟*，可以延迟小工具的注入
避免在启动时运行检查、加载任意库和子门控。

请查看[配置指南](docs/advanced_config.md)。

## 如何构建

- 检查项目
- 运行`./gradlew:module:assembleRelease`
- 构建 Magisk 模块应该位于“out”目录中。

您还可以使用 `./gradlew :module:flashAndRebootZygiskRelease` 直接构建模块并将其安装到您的设备上

## 注意事项

- 对于模拟器，这将在本机领域启动该小工具。 这意味着您将能够拦截 Java，但不能拦截本机函数。

## 感谢

- 灵感来自 https://github.com/Perfare/Zygisk-Il2CppDumper
- https://github.com/hexhacking/xDL

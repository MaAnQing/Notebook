#Android 的蓝牙简介


本文主要是翻译了 Google 的官方[文档](http://source.android.com/devices/bluetooth.html)。

##简介
Android 提供默认的蓝牙协议栈是 BlueDroid，分为两层：蓝牙嵌入式系统（BTE）和蓝牙应用层（BTA），BTE 层主要实现蓝牙的核心功能，BTA 层则主要负责和 Anroid 框架通信。

Android 4.2 之前，Google 一直应用的是 Linux 官方蓝牙协议栈，即知名老牌开源 
项目 BlueZ。BlueZ 实际上是由高通公司在 2001 年 5 月基于 GPL 协议 release 的一个开源项目，该项目仅 release 一个月后就被 Linux 之父 Linux Torvalds 纳入了 Linux 内核，并做为 Linux 2.4.6 内核的官方蓝牙协议栈。随着 Android 设备的流行，BlueZ 也得到了极大的完善和扩展。例如 Android 4.1 中 BlueZ 的版本升级为 4.93，它支持蓝牙核心规范4.0，并实现了绝大部分的 Profiles。但是从 Android 4.2 即 Jelly Bean 开始，Google 便在 Android 源码中推出了它和博通公司一起开发的 BlueDroid 以替代 BlueZ。虽然因为时间及成熟度的原因，大部分手机厂商在 Android 4.2 中仍继续应用 BlueZ。但是 BlueZ 的创始者，高通公司也将在基于其芯片的 Android 参考设计中去除 BlueZ，并仅支持 BlueDroid。BlueZ 的未来如何笔者姑且不论。不过，能让高通改弦易辙，BlueDroid 自有其合理之处。相比 BlueZ，BlueDroid 最值得称道的地方就是其框架结构变得更为简洁和清晰。另外，借助 HAL（Hardware Abstraction Layer，硬件抽象层），BlueDroid 终于不再和 dbus 有任何瓜葛。([引用](http://code.qtuba.com/article-54306.html))

##架构
蓝牙系统服务通过 JNI 来与蓝牙协议栈交互，通过 Binder IPC 来与应用层交互。系统服务给开发者提供各种蓝牙Profile的访问。Android 中的蓝牙总体架构如下图所示： 
![title](http://github.com/MaAnQing/Notebook/res/images/ape_fwk_bluetooth.png)

* 应用框架层 Application framework 在应用框架层是具体的蓝牙相关应用的代码，这里使用 [andorid.bluetooth](http://developer.android.com/reference/android/bluetooth/package-summary.html) 相关的 API 来和蓝牙硬件交互。其内部的实现，是通过 Binder IPC 机制来调用 Bluetooth 进程的；

* 蓝牙系统服务 Bluetooth system service 蓝牙系统服务位于 `packages/apps/Bluetooth`，被作为一个 Android 的系统 App。它在 Android 框架层实现了蓝牙的 Service 和 Profile，并通过 JNI 调用到 HAL 层。编译的到系统中就是 /system/app/Bluetooth.apk；

* JNI 与 [andorid.bluetooth](http://developer.android.com/reference/android/bluetooth/package-summary.html) 对应的 JNI 代码位于 `packages/apps/Bluetooth/jni`， JNI 层代码调用到 HAL 层，并当发生某些蓝牙动作（例如发现了设备）的时候，接受来自 HAL 层的回调。

* 硬件抽闲层 HAL 硬件抽闲层（Hardware Abstraction Layer）定义了 [andorid.bluetooth](http://developer.android.com/reference/android/bluetooth/package-summary.html) API 和蓝牙进程需要用到的标准接口，你必须实现这些接口来确保你的蓝牙硬件正常工作。蓝牙 HAL 相关的头文件位于 `hardware/libhardware/include/hardware/bluetooth.h` 和 `hardware/libhardware/include/hardware/bt_*.h`；

* 蓝牙协议栈 Bluetooth Stack Android 提供的默认蓝牙协议栈位于 `system/bt`。这个协议栈实现了通用的蓝牙 HAL 接口，并且可以通过扩展（Extentions）和配置（Configuration）来自定义。

* 供应商扩展 Vendor extentions 供应商也可以通过创建 libbt-vendor 并指定这些模块，来添加自己定义扩展和 HCI 层跟踪。

各层之间的交互实例可以参考这篇转载文章，很抱歉原文链接也没有找到。

##HAL 的实现
蓝牙的 HAL 相关文件在 `hardware/libhardware/include/hardware/`，包括但不局限于以下的文件：

* `bluetooth.h`：这里包含蓝牙硬件的接口定义；

* `bt_gatt.h`, `bt_gatt_client.h`, 和 `bt_gatt_server.h`：这里包含 GATT 

* `profile` 相关的接口定义；

* `bt_hf.h`：HFP profile 相关接口的定义；

* `bt_hh.h`：HID host profile 相关接口的定义；

* `bt_hl.h`：HDP profile 相关接口的定义；

* `bt_mce.h`：MAP profile 相关接口的定义；

* `bt_pan.h`：PAN profile 相关接口的定义；

* `bt_rc.h`：AVRCP profile 相关接口的定义；

* `bt_sock.h`：RFCOMM profile 相关接口的定义；

值得注意的是，Android 中蓝牙并不局限于 HAL 中暴露的这些特性和 Profile。BlueDroid 是是蓝牙协议栈的默认是实现，它实现了默认的 HAL 以及额外的自定义特性。BlueDroid 的代码在 external/bluetooth/bluedroid 目录。

##自定义蓝牙协议栈
如果使用了默认的 BlueDroid 协议栈，但是想加入一些自定义功能，可以这样做：

* 自定义蓝牙 Profile -- 如果你想要添加一个在 Android 的　HAL 中未定义的蓝牙 Profile，你必须提供一个额外的 SDK 供开发者下载，并且确保　API 在蓝牙系统进程的应用中可用（`packages/apps/Bluetooth`），并添加 Profile　的实现到　BlueDroid　中（`external/bluetooth/bluedroid`）；

* 自定义供应商扩展和配置修改　--　你可以通过创建　`libbt-vendor` 模块来添加例如额外的　AT　命令或者设备特定的配置。可以参考 `vendor/broadcom/libbt-vendor` 目录的例子；
* Host Controller Interface (HCI)　-- 你可以通过添加　`libbt-hci`　模块来提供你自己的　HCI　接口（主要用来做调试跟踪）。参考 `external/bluetooth/hci` 目录的例子。
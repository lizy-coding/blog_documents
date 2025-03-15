# 武林争霸——Flutter 小试嵌入式显示领域

2025.2.25 更新

*   [Flutter on Embedded Devices](https://flutter.dev/multi-platform/embedded) 官网最新版声称支持嵌入式设备开发的环境
*   [Supported platforms | Flutter 中文文档 - Flutter 中文开发者网站 - Flutter](https://docs.flutter.cn/reference/supported-platforms) 生成对于 3.29.0 版本支持 Debian 10 以上的 arm64 平台
*   [What’s new in Flutter 3.29. Enhancing Performance and Fidelity… | by Kevin Chisholm | Flutter | Feb, 2025 | Medium](https://medium.com/flutter/whats-new-in-flutter-3-29-f90c380c2317) 对于 3.29.0 的更新说明

| Target platform |  Target architectures   |                      Supported versions                      |    CI-tested versions     |      Unsupported versions       |
| --------------- | :---------------------: | :----------------------------------------------------------: | :-----------------------: | :-----------------------------: |
| Android SDK     |    x64, Arm32, Arm64    |                           21 to 35                           |         21 to 35          |         20 and earlier          |
| iOS             |          Arm64          |                           12 to 18                           |            17             |         11 and earlier          |
| macOS           |       x64, Arm64        |                Mojave (10.14) to Sequoia (15)                | Ventura (13), Sonoma (14) | High Sierra (10.13) and earlier |
| Windows         |       x64, Arm64        |                            10, 11                            |            10             |          8 and earlier          |
| Debian (Linux)  |       x64, Arm64        |                          10, 11, 12                          |          11, 12           |          9 and earlier          |
| Ubuntu (Linux)  |       x64, Arm64        |                    20.04 LTS to 24.04 LTS                    |   20.04 LTS, 22.04 LTS    |    23.10 and earlier non-LTS    |
| Chrome (Web)    | JavaScript, WebAssembly | [Latest 2](https://chromereleases.googleblog.com/search/label/Stable updates) |         119, 125          |         95 and earlier          |
| Firefox (Web)   |       JavaScript        | [Latest 2](https://www.mozilla.org/en-US/firefox/releases/)  |            132            |         98 and earlier          |
| Safari (Web)    |       JavaScript        |                        15.6 and newer                        |           15.6            |        15.5 and earlier         |
| Edge (Web)      | JavaScript, WebAssembly | [Latest 2](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-relnote-stable-channel) |         119, 125          |         95 and earlier          |

拥抱变化吧，相关：Flutter Debain Ubuntu Linux macOS Windows Web iOS Android

#### Flutter 与嵌入式传统开发方案对比

| 对比维度     | Flutter                                                      | 嵌入式传统开发方案                                           |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 跨平台兼容性 | 使用相同代码库可在不同嵌入式平台运行，如手机、平板、电脑、物联网终端等2 | 通常需为每个平台单独开发，开发和维护成本高                   |
| 性能表现     | 渲染引擎 Skia 直接绘制 UI，能实现流畅动画和响应式界面，但在与原生模块频繁通信、启动时加载引擎和代码等场景存在一定性能开销1 | 可针对特定平台和硬件进行深度优化，能更有效地利用硬件资源，在一些性能密集型操作上可能更具优势 |
| UI 设计      | 有丰富的 UI 组件和动画效果，UI 组件在不同平台外观和行为一致，设计灵活美观 | 受限于不同平台原生组件的风格和特性，要实现统一美观的 UI 需大量适配工作 |
| 开发效率     | 热重载功能可即时看到代码更改效果，开发和调试速度快，有统一的编程语言和开发模式 | 开发过程相对繁琐，修改代码后需重新编译和部署，开发不同平台应用需掌握多种语言和工具 |
| 学习成本     | 需学习 Flutter 框架和 Dart 语言，对不熟悉的开发者有一定学习成本 | 涉及多种编程语言、平台相关知识和硬件交互知识，学习内容广泛而复杂 |
| 应用包体     | 包含引擎和框架代码，包体通常比原生开发的应用大1              | 根据具体应用功能和所使用的原生组件等因素而定，一般相对较小   |
| 第三方库     | 有不断增长的插件生态系统，但某些特定功能的库可能不如传统嵌入式开发平台成熟 | 针对不同的嵌入式平台和应用领域，有大量成熟的第三方库和工具可供使用 |
| 平台特定功能 | 实现某些平台特定功能可能需编写平台特定代码或使用插件         | 可直接调用平台原生 API，实现特定功能相对容易                 |

### 初探 flutter-elinux

#### elinux 是什么

Linux 是一个开源的、免费的操作系统内核，而 elinux 是在 Linux 内核的基础上定制构建的一个特定版本。它与传统的桌面或服务器版 Linux 发行版（如 Ubuntu、Debian、CentOS 等）有一些不同之处。

在嵌入式系统中，通常需要更加精简和定制的操作系统，以满足嵌入式设备的资源限制和实时性需求。因此，elinux 可能会剔除一些在桌面或服务器版 Linux 中常见的组件，同时可能会针对特定的硬件平台进行优化。

elinux 和传统的桌面或服务器版 Linux 有一些共同点，比如它们都是基于 Linux 内核，都支持 Linux 的标准工具和命令行接口，也都支持标准的 Linux 库和应用程序。但由于定制和优化的缘故，elinux 在某些方面可能会有一些差异，特别是在支持的硬件平台和库的选择上。

#### flutter-elinux 是什么

[GitHub - sony/flutter-elinux: Flutter tools for embedded Linux (eLinux)](https://github.com/sony/flutter-elinux)

`flutter-elinux` 是用于在嵌入式 Linux 系统上构建和运行 Flutter 应用程序的工具。它是 Flutter 在嵌入式 Linux 平台上的特定支持工具，将 Flutter 应用程序移植到嵌入式设备中运行，使其在更多平台上运行

![flutter-elinux](https://github.com/sony/flutter-elinux/raw/main/doc/images/overview.png)

图示中的主要内容如下：

1.  Flutter Engine：这是 Flutter 的核心引擎，负责处理 Flutter 应用程序的 UI 渲染和交互。
2.  Flutter Embedder：这是嵌入器，用于将 Flutter 引擎嵌入到主机应用程序中。在 **flutter-elinux** 中，Flutter 应用程序需要通过嵌入器与 Flutter 引擎进行交互。
3.  Flutter Application：这是 Flutter 应用程序本身，由 Dart 代码和 Flutter 引擎组成。Flutter 应用程序通过嵌入器与 Flutter 引擎进行通信。
4.  Wayland 和 DRM：这是 Linux 系统上的图形后端，用于处理图形渲染和显示。在 **flutter-elinux** 中，Wayland 和 DRM 被用作图形后端。
5.  Linux Input：这代表 Linux 系统上的输入设备，如触摸屏、键盘和鼠标。Flutter 应用程序可以通过 Flutter 引擎和嵌入器接收来自输入设备的输入。
6.  OpenGL ES 和 EGL：这是用于处理图形渲染的接口。Flutter 引擎通过 OpenGL ES 和 EGL 与图形硬件进行通信，完成图形渲染。

架构图的含义是展示了在嵌入式 Linux 环境中，`flutter-elinux` 是如何将 Flutter 引擎嵌入到应用程序中，并与底层系统的图形硬件和输入设备进行交互的。通过嵌入器和 EGL-Wayland 接口，Flutter 应用程序可以在嵌入式 Linux 系统上实现图形渲染和用户交互。

如何将 `elinux` 系统加入到 Flutter 引擎中：

在 `flutter-elinux` 中，已经提供了对 `elinux` 系统的支持，你可以按照以下步骤将 `elinux` 系统加入到 Flutter 引擎中：

1.  获取 `flutter-elinux` 项目：首先，你需要获取 `flutter-elinux` 的代码，可以从 GitHub 上克隆或下载这个项目。
2.  构建 Flutter 引擎：接下来，根据 `flutter-elinux` 的文档中提供的指引，构建 Flutter 引擎，并确保支持 `elinux` 平台。
3.  构建 Flutter Embedder：在构建 Flutter 引擎的过程中，会生成 Flutter Embedder 的库文件，这是用于将 Flutter 引擎嵌入到主机应用程序中的关键组件。
4.  创建 Flutter 应用程序：使用 Flutter SDK 创建你的 Flutter 应用程序，编写 Dart 代码，设计 UI 界面等。
5.  集成 Flutter 引擎和嵌入器：将构建好的 Flutter 引擎和嵌入器集成到你的 `elinux` 系统中的主机应用程序中。
6.  设置图形后端：根据 `flutter-elinux` 的文档中的指引，将图形后端设置为 Wayland 和 DRM，这样 Flutter 应用程序就能在 `elinux` 系统上进行图形渲染。
7.  运行 Flutter 应用程序：完成以上步骤后，你就可以在 `elinux` 系统上运行你的 Flutter 应用程序了。

## 验证

设备信息

rk3568
SO：Debain 10 aarch64
GPU驱动：x11

## 环境配置

### 网卡配置

通过串口模式调试设备将它设置为 IP 自动获取
auto eth0
allow-hotplug eth0
iface eth0 inet dhcp

systemctl restart networking.service

### 代理配置，在 profile 添加 IP port

```shell
vim /etc/profile
```

跟进你的代理在配置文件中添加配置

```profile
http\_proxy=<http://IP:port>
https\_proxy=<https://IP:port>
export http\_proxy
export https\_proxy
```

配置后刷新文件，并指定 git 的全局代理

```shell
source /etc/profile
git config --global http.proxy <http://IP:port>
git config --global --get http.proxy
```

## demo 创建

环境：
Flutter version: 3.10.5\
Flutter-elinux version:3.10.5\
Cmake version: 3.26.4
Git version:2.33

**Action**

*   以下用到绝对路径的地方使用 \~/ 代替
*   不同 GPU 后端图形化库不一致，要求的 so 库不一样，使用 flutter-elinux build 指定参数不一样
*   Flutter 与 Flutter-elinux 环境版本应一致 验证环境 3.10.5
*   cmake 版本大于 3.1.3
*   git 版本 支持 restore 命令
*   flutter-elinux 编译需要的资源都要从 github 获取，因此需要科学上网，参照资料给板子配置代理，使用 flutter 国内镜像会引起访问受限，
*   有提出可自行下载资源配置的方案，但可行性未知 [Artifacts download failure #191](https://github.com/sony/flutter-elinux/issues/191)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec85a4a19a9e4cf4a2ff3b5c695921c7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1136&h=817&s=130984&e=png&b=1c1c1c)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4704b4a36bc24a9ea6ff71eae10bf44b~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=981\&h=534\&s=75997\&e=png\&b=1c1c1c)

### 1、创建 elinux demo

```shell
flutter create elinux_sample
```

### 2、打开 elinux/CMakeLists.txt 添加配置

```shell
cd ~/elinux_sample/elinux  
vim CMakeLists.txt
```

添加

```cmake
set(TARGET_ARCHITECTURE "arm64")
set(CMAKE_CXX_COMPILER g++-aarch64-linux-gnuindex)
```

### 3、编辑 elinux/runder/CMakeLists.txt

```shell
cd elinux_sample/elinux/runder
vim CMakeLists.txt
```

替换原始文件或在原始基础上添加新配置

**注：** 不同硬件环境使用 so 文件不同，需要结合硬件进行选用

```
cmake\_minimum\_required(VERSION 3.15)
project(runner LANGUAGES CXX)

if(FLUTTER\_TARGET\_BACKEND\_TYPE MATCHES "gbm")
add\_definitions(-DFLUTTER\_TARGET\_BACKEND\_GBM)
endif()

set(FLUTTER\_LIB \${CMAKE\_CURRENT\_SOURCE\_DIR}/flutter\_lib/)

add\_executable(${BINARY_NAME} "${FLUTTER\_MANAGED\_DIR}/generated\_plugin\_registrant.cc")
apply\_standard\_settings(\${BINARY\_NAME})

target\_link\_libraries(${BINARY_NAME} PRIVATE flutter)
target_link_libraries(${BINARY\_NAME} PRIVATE flutter\_wrapper\_app)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/libffi.so)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/libwayland-cursor.so)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/libxkbcommon.so.0)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/libmali-bifrost-g52-g2p0-x11.so)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/libdrm.so.2)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/libwayland-server.so.0)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/libc.so.6)
target\_link\_libraries(${BINARY_NAME} PRIVATE ${FLUTTER\_LIB}/ld-linux-aarch64.so.1)

```

### 4、备份需要的 so 库

创建 elinux\_sample/elinux/flutter\_lib 路径后从用户盘拷贝待使用的 so 文件，如果没有的话根据硬件型号进行下载

```shell
sudo cp /usr/lib/aarch64-linux-gnu/libffi.so ~/elinux_sample/elinux/flutter_lib
sudo cp /usr/lib/aarch64-linux-gnu/libwayland-cursor.so ~/elinux_sample/elinux/flutter_lib
sudo cp /usr/lib/aarch64-linux-gnu/libxkbcommon.so.0 ~/elinux_sample/elinux/flutter_lib
sudo cp /usr/lib/aarch64-linux-gnu/libmali-bifrost-g52-g2p0-x11.so ~/elinux_sample/elinux/flutter_lib
sudo cp /usr/lib/aarch64-linux-gnu/libdrm.so.2 ~/elinux_sample/elinux/flutter_lib
sudo cp /usr/lib/aarch64-linux-gnu/libwayland-server.so.0 ~/elinux_sample/elinux/flutter_lib

ld-linux-aarch64.so.1  libc.so.6  

sudo cp /lib/ld-linux-aarch64.so.1 ~/elinux_sample/elinux/flutter_lib

sudo cp /lib/aarch64-linux-gnu/libc.so.6  ~/elinux_sample/elinux/flutter_lib


~/elinux_sample/elinux/flutter_lib$ ls
ld-linux-aarch64.so.1  libc.so.6  libdrm.so.2  libffi.so  libmali-bifrost-g52-g2p0-x11.so  libwayland-cursor.so  libwayland-server.so.0  libxkbcommon.so.0
```

### 5、build

系统跟路径是 /dev/root 的话

```shell
flutter-elinux build elinux --target-arch=arm64 --target-sysroot=/ --target-backend-type=x11 --verbose
```

*   `build`: 指定要执行的操作是构建应用程序。
*   `elinux`: 指定目标平台为 eLinux。
*   `--target-arch=arm64`: 指定目标架构为 arm64，即 ARM 64位架构。
*   `--target-sysroot=/`: 指定目标系统根路径，这里是 /dev/root。
*   `--target-backend-type=gbm`: 指定目标后端类型为 gbm，即使用 GBM 后端。
*   `--debug`: 指定以调试模式构建应用程序。

请确保将 `target-sysroot=` 参数赋值为实际的 Debian 10 aarch64 系统根路径。

### 6、运行

进入到项目中 bundle 绝对路径

```shell
cd ~/elinux_sample/build/elinux/arm64/release/bundle
./[app_name] --bundle=~/elinux_sample/build/elinux/arm64/release/bundle
```

整体说明：

*   对于 arm64 Flutter 官方暂未支持，可由 [sony/flutter-elinux: Flutter tools for embedded Linux (eLinux)](https://github.com/sony/flutter-elinux) 提供的开源 Flutter-Elinux 进行环境编译

*   可以使用交叉编译在 x86 进行编程后在 arm 平台使用，但对于使用人员及环境要求比较严格

*   直接在板子进行资源部署编译话需要注意物理内存占用，

![image-20250315141700988](D:\desk\code\blog_documents\image\image-20250315141700988.png)
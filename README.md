# OBS Studio 云台摄像机控制插件（PTZ Controls）· 简体中文版

[![Push](https://github.com/glikely/obs-ptz/actions/workflows/push.yaml/badge.svg)](https://github.com/glikely/obs-ptz/actions/workflows/push.yaml)
[![Crowdin](https://badges.crowdin.net/obs-ptz/localized.svg)](https://crowdin.com/project/obs-ptz)

> 本仓库（`obs-ptz-zh`）是 [glikely/obs-ptz](https://github.com/glikely/obs-ptz) 的**简体中文汉化分支**。
> 插件本身是成熟的上游项目，本分支只做中文化、不动功能逻辑，方便中文用户直接上手。
> 英文原版说明见 [README.en.md](README.en.md)。

这是一个用于从 OBS Studio 直接控制 PTZ（云台）摄像机的插件。

插件会在 OBS 主窗口中增加一个控制停靠面板，可以就地控制摄像机的水平转动（Pan）、
俯仰（Tilt）与变焦（Zoom）。它还会跟踪当前的活动场景，自动选中要控制的那台摄像机；
也可以添加「PTZ 动作」源，在场景切换时触发摄像机动作，实现自动化。

![PTZ 控制面板截图](/docs/ptz-controls-screenshot.png?raw=true "OBS Studio PTZ 控制面板")

![PTZ 设备设置截图](/docs/ptz-settings-screenshot.png?raw=true "OBS Studio PTZ 设备设置")

功能一览：

- 调整摄像机的水平、俯仰、变焦与对焦
- 手动对焦 / 自动对焦切换
- 给摄像机控制功能分配热键
- 用摇杆（游戏手柄）控制摄像机位置
- 保存与调用摄像机预设位
- 在 OBS 中同时控制多台摄像机
- 根据活动场景自动选中相应摄像机
- 控制摄像机电源
- 调整摄像机白平衡
- 支持多种摄像机控制协议，包括：
  - VISCA（RS232、RS422、UDP 与 TCP）
  - Pelco-P
  - Pelco-D
  - ONVIF（实验性）
  - USB 摄像机（UVC）

## 汉化说明

本分支相对上游的改动**只有中文化**，涉及以下文件：

| 文件 | 改动 |
| --- | --- |
| `data/locale/zh-CN.ini` | 简体中文语言包：在上游 AI 草稿基础上**逐条校订**——统一术语（Pan=平移 / Tilt=俯仰 / Zoom=变焦）、标点全部改为中文全角、修正误译 |
| `data/locale/en-US.ini` | 补齐上游缺失的词条（英文原文），使新增词条在其它语言下也能正常回退 |
| `src/ptz-controls.cpp` | 摇杆轴映射的 9 个选项名改用命名空间 key（原先这 8 个选项**没有语言包条目**，中文界面下会显示英文） |
| `src/ptz-controls.ui` | 面板右键菜单的 6 个菜单项文本改用语言包 key（原先硬编码英文：属性 / 添加 / 移除 / 上移 / 下移 / 重命名） |
| `src/ptz.c` | 插件模块名与描述改用命名空间 key，可在 OBS 插件列表中显示中文 |
| `README.md` | 项目介绍中文化（英文原版保留为 `README.en.md`） |

### 已知限制

- **Windows 安装程序界面仍为英文**：安装器用的是 Inno Setup 自带的英文语言文件（`compiler:Default.isl`），
  而官方编译环境里没有简体中文 `.isl`，强行加入会让打包失败。安装过程只有两三步，不影响使用。
- 除简体中文外，其余 29 种语言**未做改动**；若某个新词条没有对应语言的译文，OBS 会自动回退显示英文。
- 本分支不参与上游的 Crowdin 翻译流程。若日后启用仓库内的 `.github/workflows/crowdin.yml`，
  该工作流会以 PR 形式拉取上游译文，可能覆盖这里的汉化内容——纯汉化用途建议保持该工作流关闭。

### 跟随上游更新

```bash
git remote add upstream https://github.com/glikely/obs-ptz.git   # 首次
git fetch upstream
git merge upstream/main        # 上游若改动了语言包 key，按提示重新校订受影响的词条
```

## 相关链接

- [OBS 官方资源页](https://obsproject.com/forum/resources/ptz-controls.1284/)
- [上游项目 PTZ Controls on GitHub](https://github.com/glikely/obs-ptz)
- [上游翻译工程 PTZ Controls on Crowdin](https://crowdin.com/project/obs-ptz)

# 使用指南

## 安装

到上游的 Releases 页面下载对应平台的安装包。官方目前提供 Windows（x64、arm64）、
macOS（Universal）与 Ubuntu Linux 26.04（x86_64、aarch64）的二进制包。
下载后直接安装即可。插件需要 **OBS Studio 31 或更高版本**。

> Ubuntu 24.04 自带的 `obs-studio` 软件包是 30 版，版本过低；请改从 obsproject PPA 安装
> OBS Studio（该 PPA 只提供 x86_64 构建）。

如果你的平台或发行版不在上述范围内，需要参考下文「从源码构建」。

[OBS PTZ Releases](https://github.com/glikely/obs-ptz/releases)

## 配置

在 OBS 菜单栏的 `停靠窗口` 中选择 `PTZ 控制`，控制面板就会出现。
你可以把它拖到 OBS 主窗口的任意一侧停靠，也可以让它保持浮动。

初次使用时没有任何已配置的摄像机。点击面板底部的齿轮图标，或在 `工具` 菜单中选择
`PTZ 控制`，会打开 PTZ 设置窗口。

设置窗口有三个选项卡：`常规`、`摄像机` 与 `关于`。
`常规` 选项卡里是对所有摄像机都生效的设置；`摄像机` 选项卡用来增删摄像机、调整单台摄像机的设置；
`关于` 选项卡显示插件信息与当前版本。

### 添加摄像机

在本插件中，摄像机是与 OBS 的视频源（来源）关联的。
添加摄像机前，先在 OBS 的 `来源` 面板里为这路摄像机画面添加一个源。
画面正常显示后，再到 PTZ 控制设置窗口中（`工具` → `PTZ 设备`）为这个源添加 PTZ 控制。

在 `摄像机` 选项卡中点击底部工具栏的 `+` 按钮添加设备，
会展开一份可用的摄像机控制连接方式列表，选择你的摄像机所用的控制协议。
列表中会出现一条新条目：点选它，右侧就会显示该摄像机的设置。
你需要填写摄像机连接信息——控制用的网络地址，或串口。
点击 `应用` 按钮连接摄像机。
最后用 `源` 下拉框把摄像机与某个 OBS 源关联起来，
这样在 OBS 中预览或节目场景切换时，插件就能自动选中正确的摄像机。

### 移除摄像机

在设置窗口中选中要移除的摄像机，点击工具栏中的 `-` 按钮即可。
其设置与预设位会作为备份保留，之后可通过 `添加` 恢复。

## 控制摄像机

摄像机通过控制面板上的箭头按钮操作。
要调整某台摄像机，需先在 PTZ 面板左下角的摄像机列表里选中它。
默认情况下（设置中开启了 `自动选择活动摄像机`），当前场景变化时插件会自动选中正确的摄像机。
随后点击控制按钮即可调整摄像机机位：

- 箭头按钮：水平转动 / 俯仰
- 放大镜按钮：变焦拉近 / 推远
- 小 / 大按钮：调整对焦
- `AF` 按钮：开 / 关自动对焦，也可触发一次「一键」对焦

预设位列在面板右侧，可以直接在面板中保存、调用与重命名。
要保存预设位，右键点击该预设位并选择 `保存预设`——摄像机当前画面的缩略图会随预设一起保存并显示在列表中。
重命名同理，右键选择 `重命名预设`；选择 `清空预设` 则把名称恢复为默认值。
双击即可调用该预设位。

### 摇杆控制

在设置窗口的 `常规` 选项卡中勾选 `摇杆控制`，即可启用摇杆（游戏手柄）控制。
已连接的手柄会显示在列表框中，点选你要用来控制摄像机的那一个。
摇杆的轴可以映射为水平、俯仰、变焦或对焦；手柄按键则可映射为任意 OBS 热键动作。

## 高级功能

### 直播中锁定摄像机移动

在演播室（Studio）模式下，通常是在预览场景里把机位调好，再切换给节目场景。
节目场景上要避免手动动机位——那会在直播中显得很突兀。

PTZ Controls 默认会阻止对节目场景中可见摄像机的手动操作。
要启用此功能，请勾选设置窗口 `关于` 选项卡中的 `在演播室模式下锁定实时 PTZ 移动`。
启用后，只要摄像机出现在节目场景中，其水平、俯仰、变焦与预设位控制都会被禁用，从而杜绝直播中的误操作。
确需临时移动时，可以点击工具栏上的锁形图标临时解除锁定。

### 调试数据

插件可以输出大量调试数据，包括与摄像机之间收发的全部协议报文。
调试日志写入 OBS 主日志，默认关闭。
要启用，请在设备设置中勾选 `将协议跟踪写入 OBS 日志文件`，并以 `--verbose` 命令行参数启动 OBS。

# 从源码构建

本项目的构建脚手架来自
[OBS Plugin Template](https://github.com/obsproject/obs-plugintemplate) 仓库。
构建本插件请参考该模板的说明文档
[README.md](docs/plugin-template-README.md)。

## Linux 快速开始

只要装好了 OBS Studio 与全部构建依赖，在任何 Linux 发行版上构建都很简单，
具体安装方式请查阅你所使用发行版的文档。之后用 cmake 构建即可：

```
$ cmake -B build
$ cmake --build build --config RelWithDebInfo
$ cmake --install build --config RelWithDebInfo
```

在 Ubuntu 26.04 上，也可以直接使用 GitHub Actions 的 CI 脚本构建，
它会顺带下载并安装全部构建依赖：

```
$ ./.github/scripts/build-ubuntu
$ sudo cp -r release/RelWithDebInfo/* /usr/
```

## macOS 快速开始

最省事的办法是使用 CMake 预设。以下命令会完成配置、构建，并把插件安装到你的 OBS 插件目录：

```
$ cmake -B build_macos --preset macos
$ cmake --build build_macos --config RelWithDebInfo
$ cmake --install build_macos --config RelWithDebInfo
```

## Windows 快速开始

Windows 下最省事的是使用 GitHub Actions 的构建脚本。
先按上面链接的 obs-plugintemplate 文档装好 Visual Studio 与 CMake，
然后打开 `x64 Native Tools Command Prompt for VS 2022`，执行：

```
c:\> pwsh
PS > cd path/to/obs-ptz
PS > $env:ci=1
PS > .github/scripts/Build-Windows.ps1
```

# 参与贡献

欢迎贡献！你可以通过 GitHub Pull Request 提交改动，详见项目的 Pull Request 页面。
https://github.com/glikely/obs-ptz/pulls

我们也需要把插件翻译成更多语言，欢迎到 Crowdin 项目页参与：
[PTZ Controls on Crowdin](https://crowdin.com/project/obs-ptz)

更多细节见 [CONTRIBUTING.md](CONTRIBUTING.md)。

# 致谢

感谢所有为本项目做出贡献的人——无论是提交问题、答疑，还是贡献代码。
所有代码与文档贡献者都列在 [AUTHORS](AUTHORS) 中。

同样感谢 OBS 项目开发者，以及他们维护的
[OBS Plugin Template](https://github.com/obsproject/obs-plugintemplate) 仓库——本插件大量借鉴了该项目。

摇杆支持使用了 [QJoystick 库](https://github.com/alex-spataru/QJoysticks)。

最后，感谢所有为本项目所依赖的自由与开源软件做出贡献的人，包括 OBS Studio、Qt、
Simple DirectMedia Layer（SDL）、Linux，以及不计其数的库与工具。

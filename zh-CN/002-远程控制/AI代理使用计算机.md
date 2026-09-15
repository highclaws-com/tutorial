# AI 代理使用计算机

**用途**：使 AI 代理真正能够使用一台图形化计算机，包括列举窗口、识别控件、点击鼠标与输入键盘，而不是仅面对一块画布推测坐标。
**读者**：需要为代理赋予计算机操作能力的执行者。文中命令均可直接复制执行。
**适用范围**：第 1 至第 10 节以 Linux X11 为例，这是唯一经过实际验证的路径，验证所用发行版为 Arch Linux。macOS 与 Windows 的差异集中收录于第 11 节，该节**未经实际验证**，内容依据官方文档归纳。

## 0. 占位符

| 占位符 | 含义 |
|---|---|
| `<TARGET_HOST>` | 被操作的那台机器。本文不记录主机名，文中命令均在它上面执行 |
| `<TARGET_USER>` | 该机器上登录图形会话的账号，桌面操作发生在它的会话之内 |
| `<DRIVER>` | 驱动可执行文件，安装完成后的命令名即为 `cua-driver` |
| `<DAEMON_SOCKET>` | daemon 的本地端点，其路径随平台而异，见第 2 节 |

---

## 1. 这套方案解决什么问题

远程桌面工具提供的是一块**画布**：代理得到的是像素，只能依靠推测坐标、依靠视觉模型辨认按钮。计算机操作驱动提供的则是**结构化的窗口语义**：窗口清单、控件树、每个控件的角色与名称、可执行的动作，以及受控的鼠标与键盘输入。

二者并非替代关系。画布面向使用者，结构化操作面向代理。同一台机器可以同时部署两者：人通过浏览器观察桌面，代理通过驱动执行操作，彼此互不干扰。

本文介绍 [Cua Driver](https://github.com/trycua/cua)，它来自 `trycua/cua` 仓库，支持 macOS、Windows 与 Linux，对外提供三种接入方式：命令行、stdio MCP，以及各语言的 SDK。

## 2. 运行结构

整套方案由一个常驻的 daemon 与若干客户端组成。daemon 必须运行在**图形会话内部**，因为它需要该会话的显示服务与无障碍总线。

| 角色 | 对应的命令 | 要点 |
|---|---|---|
| daemon | `<DRIVER> serve` | 在 `<TARGET_HOST>` 上以 `<TARGET_USER>` 的身份运行，是握有桌面权限的进程；元素索引缓存、录制状态与光标状态都保存在它之内 |
| 一次性调用 | `<DRIVER> call <工具名> '<JSON>'` | 连接 daemon，输出一次结果后即退出；daemon 不在时**直接失败**，不会改为在本地擅自执行 |
| MCP 服务 | `<DRIVER> mcp` | stdio 传输。在 Linux 与 Windows 上单独运行时它会自建运行时，若要接入已有的 daemon，必须显式传入 `--socket` |
| 有限命令 | `list-tools`、`describe`、`doctor`、`status` | 不创建运行时，因而可以在没有桌面的环境中执行 |

daemon 只监听一个属于同一用户的、权限为 `0600` 的本地端点，本文记作 `<DAEMON_SOCKET>`：在 Linux 上位于 `~/.cache/cua-driver/cua-driver.sock`，在 macOS 上位于 `~/Library/Caches/` 下的对应路径，在 Windows 上则是一条命名管道。pid 文件与端点位于同一目录。该服务本身**不开放任何 TCP 端口**。

端点即凭证：能够连接它的一方，即可操作那块桌面。它仅供同一用户访问，也不应被转发到任何公网地址。

这一设计的意义在于把「负责与调用方通信的进程」与「握有桌面权限的进程」相互分离。Windows 上如此设计，是因为从 SSH 进入的进程会落在没有桌面的服务会话之中；macOS 上如此设计，是因为隐私授权绑定的是应用身份，而非可执行文件的路径。

权限模式在 daemon 启动时即被固定，运行期间无法更改：

| 模式 | 适用情形 |
|---|---|
| `standard` | 默认。日常自动化使用，不打断使用者；仅个别越界行为需要显式授权 |
| `bounded` | 无人值守的代理或网关，只允许一份经过审阅的工具与应用清单，清单之外一律拒绝 |
| `unrestricted` | 机器可随时丢弃或完全可信，且愿意显式承担风险 |

## 3. 安装

有两条路径，按机器自身的管理习惯选择。以下两小节均以 Arch Linux 为例给出实测命令，其他发行版更换包管理器即可。

### 3.1 使用发行版软件包

Arch 上有一个 AUR 包 `cua-driver-bin`，它把主体文件安装到 `/usr/lib/cua-driver/`，在 `/usr/bin/cua-driver` 建立符号链接，并自带一个 systemd 用户单元：

```bash
git clone https://aur.archlinux.org/cua-driver-bin.git
cd cua-driver-bin && makepkg -si
```

### 3.2 使用官方安装脚本

```bash
curl -fsSL https://cua.ai/driver/install.sh | bash -s -- --no-modify-path
```

两条路径的取舍如下：

| | 发行版软件包 | 官方安装脚本 |
|---|---|---|
| 安装位置 | `/usr/bin/cua-driver`，任何 shell 都能直接调用 | `~/.cua-driver/packages/`，并在 `~/.local/bin/cua-driver` 建立符号链接 |
| 开机自启 | 包内自带 systemd 用户单元 | 无，需要自行编写 |
| 卸载方式 | 包管理器一条命令 | 官方 `uninstall.sh`，默认保留遥测身份 |
| 所需权限 | 安装时需要提权 | 全程无需提权 |
| 版本 | 可能落后一到两个小版本 | 与上游同步 |

### 3.3 两处需要注意的地方

**PATH 的追加。** 官方安装器只在「当前 shell 的 `PATH` 中不含 `~/.local/bin`」时才执行追加，而从 SSH 进入的非登录 shell 通常恰好不含该项，于是它会向 `~/.bashrc` 追加一段重复的 PATH 行。安装时应加上 `--no-modify-path`，或设置 `CUA_DRIVER_RS_NO_MODIFY_PATH=1`，不应让安装脚本改动使用者的 shell 配置。

**遥测默认开启。** 安装器会发送一次安装事件，daemon 首次启动时还会生成安装标识并记录注册。若希望全程没有数据回传，应在安装时设置 `CUA_DRIVER_RS_TELEMETRY_ENABLED=false`，并在启动 daemon 之前先执行：

```bash
<DRIVER> telemetry disable
<DRIVER> telemetry reset-id    # 若标识已经生成，用它抹除
```

### 3.4 安装后的验证

```bash
<DRIVER> --version
<DRIVER> doctor
```

`doctor` 会依次检查二进制文件、安装布局、显示服务与无障碍总线。需要注意的是，它探测的是**执行它的那个 shell 的环境**，而非 daemon 的环境，因此从 SSH 中执行时会报 `DISPLAY` 未设置。这并不说明 daemon 存在问题，判断 daemon 的状态应使用 `status`。

## 4. 启动 daemon 的四项原则

以下四项按顺序执行，不可跳过其中任何一项。

**第一，事先征得同意，并避开使用者正在使用机器的时间。** 这一步相当于把一个握有桌面控制权的进程放进他人的工作环境，一旦出错，代价由对方承担。

**第二，关闭代理光标浮层，即以 `<DRIVER> serve --no-overlay` 启动。** 默认的浮层会在屏幕上绘制一个标记代理位置的光标，在合成桌面上它会自行给出警告：`X11 overlay: … save-unders will be served without readback confirmation`。实测中，一次采用默认参数的启动之后，使用者的桌面出现冻屏，停止 daemon 并重启远程画面服务后恢复正常；由于两处改动是同时进行的，**无法单独归因**，但浮层是首要嫌疑，因此此后固定使用 `--no-overlay`。第二次以该参数启动后，同样的操作均正常完成。

**第三，先做只读操作。** 第一步只执行 `status`、`list_windows`、`list_apps` 与按窗口截图，确认桌面响应正常、未产生任何副作用之后，再考虑输入类操作。

**第四，用完立即停止，不作常驻。** 除非明确需要长期使用，否则以临时单元启动，操作结束后用 `<DRIVER> stop` 收束。

从图形会话之外启动 daemon 时，需要显式指定会话环境，不应依赖继承。Linux 上实测可用的形式如下：

```bash
systemd-run --user --unit=cua-driver-pilot \
  --setenv=DISPLAY=:0 \
  --setenv=XAUTHORITY=$HOME/.Xauthority \
  --setenv=DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus \
  <DRIVER> serve --no-overlay
```

临时单元的名称要避开发行版软件包自带的单元名，否则 `systemd-run` 会拒绝执行，并报告 `Unit … was already loaded or has a fragment file`。

若确需常驻，发行版软件包中自带的单元即可直接使用；macOS 与 Windows 采用另一套开机自启机制，见第 11 节。

## 5. 如何获取屏幕内容：整屏截图不可信

**这是实测中最具价值的一条结论。** 在带合成器的桌面上，整屏截图得到的是**不随桌面变化的那一帧画面**，而非使用者所看到的画面。判断依据十分明确：驱动截取的整屏图与独立工具 `import -window root` 截取的图**逐字节完全相同**，并且在使用者打开新窗口前后、间隔数分钟取得的四张图，其**哈希值完全一致**。

因此有三点做法：

- 获取屏幕内容一律采用**按窗口截图**。先用 `list_windows` 取得 `pid` 与 `window_id`，再用 `get_window_state` 配合 `screenshot_out_file` 将 PNG 写入文件。按窗口截图得到的是真实像素，实测能够正确反映目标窗口的内容。
- 截图返回默认携带大量 base64 编码的图像数据，不应使其进入上下文。可用 `screenshot_out_file` 落盘后按需读取，或用 `max_dimension` 压缩为缩略图。
- 优先采用语义方式。`get_window_state` 同时返回无障碍树，树非空时以 `element_index` 执行操作，这种方式**不会移动使用者的真实鼠标**，也不会夺取焦点。

## 6. 如何验证操作是否生效

**驱动自身的成功回执不构成验证。** 工具返回成功，仅说明事件已被投递出去，并不说明应用已经处理该事件。每一个操作都应当找到一条驱动之外的第二证据，且最好来自目标机器本身。

| 证据类型 | 获取方式 | 说明 |
|---|---|---|
| 进程 | `pgrep -af <程序名>` | 证明应用确实已经启动 |
| X 窗口 | `xdotool search --name <标题>`、`xdotool getwindowgeometry --shell <id>` | 证明窗口确实存在，且其几何参数可与驱动回报的数值对照 |
| 磁盘文件 | 让目标应用将内容存盘，再直接读取该文件 | 最为可靠，完全不依赖驱动 |
| 页面状态 | 本地回环地址上的状态接口 | 适用于浏览器类操作 |
| 屏幕截图 | 按窗口截图后自行查看 | 需要视觉通道可用 |

实测中的一次完整对照：驱动 `launch_app` 回报的进程号、窗口号与几何参数，与 `pgrep`、`xdotool` 两个独立来源给出的数值**完全一致**。这样的结果才可判定为验证通过。

此外有两点：操作超时但结果未知时，**应先核对第二证据，再决定是否重试**，不可盲目重放；每一个操作都应当能够追溯到与之对应的证据。

## 7. Linux X11 环境下的实测边界

| 项目 | 结果 |
|---|---|
| 窗口发现 | 可用，能够列出真实的窗口清单、几何参数与层叠顺序 |
| 按窗口截图 | 可用，得到的是真实像素 |
| 无障碍树 | 可用。在一个 GTK 窗口上取得 168 个元素，带有角色、名称与可执行的动作 |
| 语义操作 | 可用，以元素索引执行，不移动使用者的指针 |
| 前台指针与键盘 | 可用，但会真实移动使用者的鼠标，也可能夺取焦点 |
| 整屏截图 | **不可用**，原因见第 5 节 |
| 并发 | 一个 daemon 对应一块桌面。多个客户端可以同时连接，但它们共享同一个屏幕、键盘与指针，操作会互相干扰，需要自行安排次序 |

桌面环境的差异需要在安装之前查明：某些桌面环境下，驱动不会主动开启应用的无障碍广播，依赖驱动开启该桥接的应用可能返回残缺的树。遇到这种情况，不应强行设置 `CUA_DRIVER_RS_A11Y_ADVERTISE_MODE`，退回按窗口截图配合像素坐标即可。Wayland 下的情形另作考虑，官方文档对每一种合成器都给出了独立的支持等级，不可将 X11 的结论直接套用过去。

## 8. 接入其他代理框架

```bash
<DRIVER> mcp-config --client <客户端名>
```

它会直接输出对应客户端的配置片段，支持的客户端包括 Hermes、Claude Code、Codex、Cursor 与 opencode 等。接入已有的 daemon 时**必须显式传入 `--socket <DAEMON_SOCKET>`**，否则在部分平台上，MCP 进程会自行创建一个没有桌面的运行时。

框架自带集成的情况需要单独确认。例如，Hermes 内置的 `computer_use` 工具集，其后端正是调用 `cua-driver mcp`，命令还可以用 `HERMES_CUA_DRIVER_CMD` 覆盖为任意命令，看似能够指向远端；但该版本中的可用性判断写死了「非 macOS 即不可用」，因此在 Linux 目标上直接失效，只能改由 MCP 注册或修改源码。**结论是：应当先查阅框架源码中的可用性判断，而不是仅凭文档。**

## 9. 停止与卸载

以下命令均在 `<TARGET_HOST>` 上执行。

```bash
<DRIVER> stop                      # 令 daemon 优雅退出
```

通过发行版软件包安装的，用包管理器卸载即可，例如在 Arch 上执行 `pacman -R cua-driver-bin`。通过官方脚本安装的：

```bash
curl -fsSL https://cua.ai/driver/uninstall.sh | bash -s -- --purge
rm -rf ~/.cache/cua-driver
```

`--purge` 不可省略。若不加上它，程序会保留遥测身份与偏好设置，下次安装回来仍然是同一个匿名身份。

收尾需要验证四件事：`command -v cua-driver` 没有输出；`~/.cua-driver` 与 `/usr/lib/cua-driver` 均不存在；`systemctl --user list-units --all | grep cua` 没有结果；没有残留进程。若此前承诺过不改动使用者的 shell 配置，还应把安装前后 `~/.bashrc` 的哈希值对照一遍，以数据说明结论。

## 10. 常见故障对照表

| 现象 | 原因 | 处理 |
|---|---|---|
| 提示 `daemon is not running` | daemon 未启动，或已经退出 | 按第 4 节重新启动，先查看 `journalctl --user -u <单元名>` |
| `doctor` 报告 `DISPLAY` 未设置 | 正常现象，它探测的是当前 shell 的环境 | 判断 daemon 应使用 `status`，而非 `doctor` |
| `doctor` 报告无障碍总线不通 | 会话总线的环境变量未传给 daemon | 启动时显式指定 `DBUS_SESSION_BUS_ADDRESS` |
| `Failed to start transient service unit: Unit … already loaded` | 临时单元的名称与软件包自带的单元名冲突 | 更换一个单元名 |
| `list_windows` 返回为空 | daemon 没有桌面权限，或启动了第二个没有桌面的运行时 | 确认 daemon 位于图形会话之内，MCP 一侧传入 `--socket` |
| 整屏截图前后完全一致 | 即第 5 节所述的情形 | 改用按窗口截图 |
| 点击之后没有反应 | 事件已经投递但应用未处理，或有模态窗口遮挡 | 先按第 6 节核对第二证据，再重新截图查找遮挡 |
| 元素索引失效 | 元素索引仅在下一次截图之前有效 | 任何改变状态的操作之后重新截图 |
| 使用者报告桌面无响应 | 见第 4 节第二项 | 立即执行 `<DRIVER> stop`，必要时再重启该机器上的远程画面服务 |
| 安装后找不到 `cua-driver` | 官方脚本将其安装在 `~/.local/bin`，而非登录 shell 的 PATH 中不含该目录 | 以绝对路径调用，或改用发行版软件包 |

## 11. macOS 与 Windows 目标机器

> 本节**未经实际验证**，仅依据官方文档与各平台自身的机制归纳。**本文唯一已确证可用的，是第 1 至第 10 节的 Linux 路径。**

三个平台共用同一套工具面、同一套命令行与 MCP 接口，以及同一套权限模式。差异集中在三个方面：安装位置、权限授权方式，以及由谁来持有桌面。

### 11.1 macOS

要求 macOS 14 或更新的版本，Apple Silicon 与 Intel 均可。官方安装脚本会把 `CuaDriver.app` 放入 `/Applications`，并在 `~/.local/bin/cua-driver` 建立符号链接。该应用包使用固定的签名身份，因此隐私授权可以跨升级保留下来。

macOS 的特殊之处在于**授权绑定的是应用身份，而不是可执行文件的路径**，因此官方只认可三种启动方式：

| 启动方式 | 做法 | 适用情形 |
|---|---|---|
| 独立 daemon | 将授权给予 `CuaDriver.app`，以 `open -n -g -a CuaDriver --args serve` 启动 | 供外部客户端调用，推荐 |
| MCP 进程自持运行时 | `cua-driver mcp --direct` | 由发起调用的进程自行承担授权归属 |
| 嵌于宿主应用 | 由持有授权的应用启动 daemon，再让 MCP 代理连接它的端点 | 应用集成 |

**在应用包之外直接运行 `cua-driver serve` 是不被支持的**，因为它没有稳定的应用身份可供系统归因。

需要两项授权，缺一不可：**辅助功能**负责所有读取控件树、按元素索引点击以及键盘与文本操作；**屏幕录制**负责截图，缺少它时按窗口截图只返回控件树而没有图像。授权的标准流程是先启动 daemon，再执行 `cua-driver permissions grant`，随后在系统设置中打开开关——**弹窗本身只是登记，开关才是授权**，而且改完之后必须让应用完整重启一次才会生效。真实状态可用 `cua-driver permissions status` 查询；daemon 不在时它会报告为未知，而不会以终端自身的授权代替。

官方已覆盖的框架有 Electron、Tauri、AppKit、SwiftUI 与 WKWebView；部分后台滚动与拖拽操作会返回明确的拒绝，而不会给出不实的成功结果。

### 11.2 Windows

要求 Windows 10 或 11，也可以是带交互式桌面的 Windows Server。安装使用 PowerShell：

```powershell
irm https://cua.ai/driver/install.ps1 | iex
cua-driver autostart kick
```

安装器把版本目录放在 `%USERPROFILE%\.cua-driver\packages\releases\`，把可执行文件从 `%LOCALAPPDATA%\Programs\Cua\cua-driver\bin` 暴露出来，并注册一个登录时自动启动的计划任务。

Windows 的特殊之处在于**从 SSH 进入的进程会落在 Session 0**，那是一个没有桌面的服务会话，因此列举窗口、点击、读取控件树这些工具都会返回空结果。校验方式如下：

```powershell
query session                      # 自己账号那一行应当显示 Active 或 Disc
cua-driver doctor                  # 会直接报告 Session 0 这一情形
```

解决办法是让 daemon 运行在 Session 1 或更高的交互会话之内，由那个计划任务负责，SSH 一侧只负责转发协议：

```powershell
cua-driver autostart enable        # 注册计划任务，指定为交互式登录触发
cua-driver autostart kick          # 不必等待下次登录，立即启动
cua-driver status                  # 应当报告 daemon 所在的交互会话
cua-driver call list_apps --socket \\.\pipe\cua-driver
```

在 SSH 一侧启动 MCP 时，同样必须显式指定那条命名管道，否则它会自持一个落在 Session 0 的运行时，随后在毫无提示的情况下看不到任何内容。计划任务在远程桌面断开之后依然有效，因为断开的会话仍是活动的交互会话；它会一直运行到显式注销或重启为止。

官方已覆盖的框架有 Electron、Tauri、WPF、WinUI 3 与 WebView2；部分后台的 Chromium 手势与提权边界尚未得到证明。

### 11.3 这两套系统上必须实测的项目

- macOS 上两项授权在真实使用场景中是否都能一次授予，以及更改授权后的重启是否会打断正在进行的工作
- macOS 上后台滚动与拖拽这类操作的拒绝范围，是否会挡住实际需要完成的操作
- Windows 上从 SSH 一侧调用交互会话中的 daemon 时，注入的输入是否确实落在使用者正在使用的那个窗口
- Windows 上提权边界对目标应用的影响：以管理员身份运行的应用，很可能接收不到非提权进程投递的输入

以上四点都不能依照本文直接下结论。

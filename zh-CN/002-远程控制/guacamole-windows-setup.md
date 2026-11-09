# Guacamole 远程桌面接入 Windows — 从零到通的最小步骤

前提：已开启沙箱的 Guacamole 桌面 sidecar；Windows 已装 OpenSSH 服务，并经 `hc` 反向隧道把 22 端口暴露为沙箱内 `localhost:2222`。下文用 `<SSH_USER>`、`<SSH_PASS>`、`<VNC_PASS>` 等占位符，实际值由你与用户私下约定，**绝不写进公开文档**。

## 一、在 Windows 上（管理员 cmd，一次性）

1. 把 SSH 账号加进管理员组（远程登录才有管理权）：
   `net localgroup Administrators <SSH_USER> /add`
   并允许远程管理通道获得完整管理员令牌：
   `reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f`
2. Windows 家庭版没有 RDP 服务端 → 装 TightVNC 服务器（安装包放本地后）：
   `msiexec /i tvnc.msi /quiet /norestart ADDLOCAL="Server" SERVER_REGISTER_AS_SERVICE=1 SERVER_ADD_FIREWALL_EXCEPTION=1 SET_USEVNCAUTHENTICATION=1 VALUE_OF_VNCAUTHENTICATION=<VNC_PASS>`
3. TightVNC 默认拒绝回环连接（SSH 隧道恰恰是回环进来的），必须打开：
   `reg add "HKLM\SOFTWARE\TightVNC\Server" /v AllowLoopback /t REG_DWORD /d 1 /f`
   `net stop tvnserver && net start tvnserver`
4. 密码规则（每次配置都生成全新 8 位随机口令，见"密码轮换"一节）：
   msi 的密码参数经常不生效（连接时报 "Server is not configured properly"）。**可靠做法**是直接用加密字节写注册表：
   - **长度必须 8 位**（TightVNC 对 7 位会自动"补空格到 8 位"，服务端/客户端补位不一致会坑人——统一用 8 位避开）。
   - 用 vncpasswd.py（github trinitronx/vncpasswd.py）把明文转成 8 字节 REG_BINARY（它使用 d3des + 固定 vnckey；首次使用需 patch py2 遗留：八进制字面量、`0x...L` 后缀、`print` 语句、str-vs-bytes）：
     `enc = do_crypt(<VNC_PASS>).hex()`
   - 写入并重启：
     `reg add "HKLM\SOFTWARE\TightVNC\Server" /v Password /t REG_BINARY /d <enc> /f`
     `reg add "HKLM\SOFTWARE\TightVNC\Server" /v ControlPassword /t REG_BINARY /d <enc> /f`
     `net stop tvnserver && net start tvnserver`
   - **验证**：从沙箱发起一次 VNC 握手（选 security type 2，DES 加密挑战-响应），确认 `AUTH: OK`。不要只看"写入成功"——重启太快或字节算错都会"看似改完"。

## 二、沙箱侧（可自动化）

1. 持久隧道（supervisor 服务）：
   `sshpass -f <pwfile> ssh -N -L 0.0.0.0:<本地端口>:127.0.0.1:5900 -p 2222 <SSH_USER>@localhost`
   ⚠ 必须绑 `0.0.0.0`，原因见"坑"一节。
2. 注册 Guacamole 连接（API `http://guacamole:8080/desktop/api/`，登录 `guacadmin/guacadmin`）：
   protocol=vnc、hostname=<沙箱容器的 Docker 网桥 IP>、port=<本地端口>、password=<VNC_PASS>。
   修改连接用 **PUT /connections/{id}**（PATCH / 分参数 PUT 均 405），body 带完整 parameters。
3. 验证顺序：沙箱脚本跑一遍 VNC 握手+认证（客户端读数据要"来多少读多少"，见坑列表），再用 agent-browser 打开连接截图确认像素；两步都成才算完成。

## 三、结果

用户打开 `SANDBOX_ROOT_URL/desktop/` → 点连接 → 直接看到桌面。VNC 密码自动取自连接的 parameters，用户从来不会被提示手输。

## 四、密码轮换与"新沙箱/新 agent 再次接入同一台电脑怎么重置"

**规则**：每次配置生成一个全新的 **8 位随机**密码（`secrets.choice`，字母+数字），绝不复用旧值；同一台电脑被多次接入时每次换新。同步写两处：Windows 注册表（真值）+ Guacamole 连接 parameters。

**真值来源**：Windows 注册表 `HKLM\SOFTWARE\TightVNC\Server` 的 `Password` / `ControlPassword`（REG_BINARY，d3des+vnckey 加密）。沙箱随时可能被销毁，文档也可能丢——**密码控制权始终在 Windows 自己的注册表里**，新 agent 只要能拿到这台 Windows，就能自己重置。

**新沙箱 + 新 agent 的步骤**：
1. 用户在新沙箱里重新布好 `hc` 反向隧道（沙箱 `localhost:2222` → Windows:22），并把 SSH 账号/密码告诉新 agent。
2. 沙箱验证 `ssh -p 2222 <SSH_USER>@localhost` 可登录。**不要**依赖旧沙箱的任何文件（密钥、supervisor 配置、缓存密码文件）——它们已随旧沙箱消失。
3. 若这台 Windows 没重装过系统：管理员组、LocalAccountTokenFilterPolicy、AllowLoopback、TightVNC 本体都仍有效，跳过第一章 1-3，直接进入密码重置。
4. 生成新 8 位随机密码 → vncpasswd.py 编码 → 写 `Password`/`ControlPassword` → 重启 tvnserver。
5. 沙箱 VNC 握手验证 `AUTH: OK` → 建（或更新）Guacamole 连接 → agent-browser 截图验证 → 交付链接给用户。
6. **永远不要把成品密码明文写进任何公开文档**——写完只保留"如何重置"的步骤本身。

## 五、容易踩的坑

- **guacd 是独立容器**：连接 hostname 写 `127.0.0.1` 会连到它自己 → 报 "server unreachable"。隧道绑 `0.0.0.0`，hostname 用沙箱容器的 Docker 网桥 IP。
- **7 位密码服务端/客户端补位不一致**（服务端补空格、客户端补 `\x00`）→ 统一用 8 位避开。
- **Windows 远程 SSH 令牌被 UAC 降级**：只加管理员组不够，要设 LocalAccountTokenFilterPolicy。
- **验收以真实握手为准**：重启 tvnserver 后等几秒再测；"写入成功"不等于"客户端可用"。

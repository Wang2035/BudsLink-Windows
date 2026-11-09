# BudsLink for Windows

> 给 Redmi / Edifier 蓝牙耳机用的 Windows 桌面控制台：电量、降噪、触摸手势，以及基于头部姿态的鼠标控制。
>
> A Windows desktop companion for Redmi (Xiaomi) and Edifier Bluetooth earbuds — battery, ANC, touch gestures, and head-tracking mouse control.
>
> 当前版本 **0.3.41** · 免安装 zip

> [!IMPORTANT]
> **测试范围**
> - **REDMI Buds 6 Pro**：全功能实测（电量 / 固件 / 认证 / 降噪 / 通透 / 手势表 / 头部追踪）。
> - **REDMI Buds 8 青春版**：实测降噪、通透、触摸控制可用。
> - 其它型号未验证。

---

## 功能

| 模块 | 说明 |
|---|---|
| 电量 / 固件 / 认证状态 | 左耳、右耳、充电盒三环显示 |
| 降噪 / 通透 / 关闭 | 通透含三小模式（标准、人声增强、环境增强），与降噪深度互斥显示 |
| 降噪深度 | 0–20 档滑块 |
| 触摸手势自定义 | 左右耳分栏；写入后 600ms 回读校验，明确提示「已同步 / 未接受 / 没有回传」 |
| 手势映射鼠标 | 单击/双击/长按/滑动映射成鼠标左/右键，走媒体键 `globalShortcut` + `mouse_event` 注入，左右可调换 |
| 头部追踪鼠标 | 椭圆手势区 + 椭圆外方向/速度映射，点头=左键、摇头=右键，`Ctrl+Alt+H` 一键开关 |
| 头控状态行 | 显示「等待耳机数据 / 追踪中 · N 帧 · 横 x° 纵 y°（中心 横 / 纵）」，并给出保活注入状态与未注入原因 |
| 「检查 HID 状态」按钮 | 只读诊断：跑 `hid-doctor.ps1` + `hid-caps.ps1`，不改任何 PnP 状态、不提权 |
| 托盘常驻 + 迷你窗 | 迷你窗含「Redmi 进阶」开关（自适应降噪、双设备连接） |
| 关闭行为 / 开机自启 / 自动重连 | 可选退出或最小化到托盘；`--autostart` 只驻留托盘；每 5 秒静默重连上次设备 |
| 单实例 | 重复启动只唤回已有窗口 |
| 应用图标 | 耳机样式，已写入 exe 资源（任务栏 / 托盘 / 开始菜单一致） |

> 表中所有条目都是 **REDMI Buds 6 Pro** 的实测结果；**REDMI Buds 8 青春版** 已确认降噪 / 通透 / 触摸控制可用，其余条目未在该型号上验证（见「测试范围」）。

---

## 测试范围

| 项目 | 实测 | 说明 |
|---|---|---|
| **REDMI Buds 6 Pro** | ✅ 全功能测试 | 电量、固件、认证、降噪/通透/深度、左右耳手势表、头追鼠标（HID）全部跑通 |
| **REDMI Buds 8 青春版** | ⚠️ 部分测试 | 降噪、通透、触摸控制可用 |
| 其它型号 | ❌ 未验证 | — |

**测试环境**：REDMI Buds 6 Pro（`VID_0094&PID_0004`）· REDMI Buds 8 青春版。

---

## 快速开始

### 用现成包

1. 下载 `BudsLink-Windows-vX.Y.Z.zip`，解压到任意目录（不要放 Program Files，免安装）。
2. 双击 `BudsLink.exe`。
3. 首次使用头控：进入「头部追踪鼠标」页，打开「启用头控」。程序会自动拉起读取器与保活，并在需要时把状态写进 `%APPDATA%\budslink\headtrack.log`；卡住时点 **检查 HID 状态**（只读，不改 PnP）。
4. 手机端在小米耳机 App 里开启 **头部追踪 / 空间音频**，两只耳机都戴上。

### 从源码跑

```powershell
# 需要 Node.js 18+ 与 Electron
npm install
npx electron .
```

本项目没有走 `electron-builder`，打包方式见「构建」。

---

## 系统要求

- Windows 10 1607 及以上
- 蓝牙 4.2+ 适配器（未做适配器兼容性矩阵，遇到问题请附适配器型号）
- **Windows PowerShell 5.1**（系统自带；HeadTrack 桥接脚本依赖 WinRT 投影，`pwsh 7` 跑不了）
- Redmi 耳机需在 Windows 蓝牙设置里配对并连接；Edifier 走 Web Bluetooth，需在支持 Web Bluetooth 的 Chromium 内核里运行

---

## 架构

```
BudsLink.exe                  Electron 壳
resources/app/
├─ main.js                    主进程：窗口/托盘/迷你窗/HTTP 静态服务/Redmi 桥接/
│                             全局快捷键/鼠标注入/头追滤波链/设置持久化/HID 只读诊断 IPC
├─ preload.js                 contextBridge 暴露 IPC 白名单
├─ bridge.ps1                 PowerShell RFCOMM 桥（经典蓝牙 FD2D，JSON 行协议）
├─ renderer/
│  ├─ index.html              主界面（单文件，内联 CSS/JS）
│  ├─ mini.html               托盘迷你控制窗
│  └─ icon.png
└─ headtrack/
   ├─ HidReader.exe           读耳机 IMU 姿态，CSV 输出（约 17.5Hz）
   ├─ KeepAlive.exe           WASAPI 静音注入，避免耳机休眠断流
   └─ *.ps1                   只读诊断（hid-doctor / hid-caps / hid-probe / list-audio / hid-state / hid-arm-test / diag）
```

### 通信

- **Redmi**：`main.js` ⇄ `bridge.ps1` 之间用 stdin/stdout 的 JSON 行，命令有 `list / connect / tx / movemouse / mouse / disconnect / quit`。
- **Edifier**：渲染进程走 Web Bluetooth（BLE GATT）。
- **头追数据**：`HidReader.exe` → stdout CSV → 主进程滤波 → `bridge.ps1 movemouse` → `mouse_event`。

---

## 头部追踪：椭圆模型

以校准后的头部姿态为中心，定义一个**可调宽高的椭圆**（`ellW` = 水平半宽，`ellH` = 垂直半高）：

```
r = sqrt( (yaw / ellW)² + (pitch / ellH)² )

r ≤ 1  椭圆内 → 只做点头 / 摇头识别，光标不动
r > 1  椭圆外 → 只推动光标，方向 = 头部偏离中心的单位向量
```

- 轴映射：`raw[0]` = 左右侧倾（**丢弃**，不参与任何计算）、`raw[1]` = 上下点头、`raw[2]` = 左右摇头。
- 越远越快：`e = r - 1`，`k = 0.2 + accel/100 × 2.0`，`f = min(1, e·k + 0.06)`，`maxSpeed = 620 × sens`。
- 手势一旦起手，光标冻结；但甩出 `r > 1.5` 视为「这是一次转头」，立刻丢弃手势、把光标交还移动。

### 滤波链（按 17.5Hz 样本率，屏幕率只做插值）

```
HidReader(17.5Hz)
  → 跳过热身全零帧，取前 8 帧中值定中心
  → 5 点滑动中值
  → 25ms 快滤波（只给手势判定用，与平滑度解耦）
  → One Euro（导数截止 0.7Hz）
  → 静止锁定（近 6 帧量程 < 0.6° 时截止频率 ×0.22）
  → 限速 500°/s
  → 60Hz 线性插值
  → 椭圆判定 → 速度映射 → 亚像素累加 → SendInput
```

### 参数映射

| 滑块 | 范围 | 映射 |
|---|---|---|
| 灵敏度 `sens` | 0.2–4 | `maxSpeed = 620 × sens` px/s |
| 平滑度 `smooth` | 0–100 | `minCut = 6.0 × 0.042^(s/100)`（6Hz 直通 … 0.25Hz 极顺） |
| 跟随性 `follow` | 0–100 | `beta = 0.002 + f/100 × 0.098` |
| 椭圆半宽 `ellW` | 0.5–40° | 水平手势安全区 |
| 椭圆半高 `ellH` | 0.5–40° | 垂直手势安全区 |
| 出圈加速速率 `accel` | 0–100 | 线性爬升斜率 `k` = 0.2 … 2.2 |
| 点头灵敏度 `nodSens` | 0.3–3 | 起手阈值 `min(4.5/nodSens, 0.9·ellH)` 度 |
| 摇头灵敏度 `shakeSens` | 0.3–3 | 起手阈值 `min(5.0/shakeSens, 0.9·ellW)` 度 |

手势冷却 450 ms，触发点击后冻结光标 160 ms，候选超时 900 ms。

> 出厂值：`sens 1.2 / follow 35 / smooth 50 / ellW 2.5° / ellH 2.5° / accel 60`。头控参数改完 400ms 自动写入 `%APPDATA%\budslink\settings.json`，重启后保留。

---

## 构建

```powershell
# 1) 语法检查
node --check main.js
node --check preload.js

# 2) 版本号两处保持一致
#    package.json 的 version
#    renderer/index.html「关于」里的版本行

# 3) 同步到打包目录（Electron 解包目录）
robocopy . ..\..\BudsLink-Windows\resources\app /MIR

# 4) zip 打包（版本号与文件名一致，完整包约 110–120 MB）
Compress-Archive -Path ..\..\BudsLink-Windows -DestinationPath ..\BudsLink-Windows-vX.Y.Z.zip
```

发布前自查：`main.js` 语法通过、两处版本号一致、zip 体积正常（异常偏小说明漏文件）、能正常启动并连上耳机。

---

## 常见问题

### 界面报「未找到耳机 HID 接口」

按下面顺序定位，每一步都有对应脚本，全部只读，不会改动 PnP 状态：

```powershell
cd resources\app\headtrack
powershell -NoProfile -ExecutionPolicy Bypass -File hid-doctor.ps1     # 四项汇总 + 判定
powershell -NoProfile -ExecutionPolicy Bypass -File hid-caps.ps1       # usage / 报告长度
powershell -NoProfile -ExecutionPolicy Bypass -File hid-arm-test.ps1   # 独立探针：arm + 读报告
```

`hid-doctor.ps1` 的退出码对应五种状态：

| 退出码 | 含义 | 处理 |
|---|---|---|
| 0 | 接口可打开 | 已经不缺接口，无数据是耳机侧不出报告 |
| 10 | 子节点记录在，但 `present=False` | HIDP 通道没建立：在 Windows 蓝牙里对该耳机「断开连接 → 连接」（不删配对） |
| 11 | 子节点记录已缺失 | 只有重新枚举能重建：断开重连，或冷启动 |
| 12 | 子节点在，接口却没注册 | 驱动绑定层，看下面「子节点在、接口未注册」 |
| 13 | 只连上了 Hands-Free 模式 | 把输出设备切到 `REDMI Buds 6 Pro Stereo` |

### 子节点在、接口未注册

Windows 把耳机的 HID 集合当成传感器设备（usage page `0x20`），`problem code 10`，HID 接口不注册，`HidReader` 只能报 `device_not_found`。

观察事实：

- 每次重连 / 重配 / 重启，Windows 都会生成**新的子实例**（`…&4&0000 → &5&0000 → &6&0000`）。
- 强绑只能对 **Present** 的节点生效；ghost 记录上调用 `UpdateDriverForPlugAndPlayDevicesW` 会返回 `True` 但实际不生效。
- 反复 disable/enable 父节点或重绑驱动会把链路打断，子实例号随之递增；这就是「时好时坏」的来源，因此本项目不再做任何自动 toggle。

### 音频在，但没有数据

确认 `hid-doctor.ps1` 里 Stereo 端点为 `ACTIVE`，再看 `%APPDATA%\budslink\headtrack.log` 里 keeper 那几行：`injecting=true` 表示保活静音已经灌进去。此时若 `reports_total=0`，是耳机固件没有推送该 collection 的数据，PC 侧没有可调项。

### 头追不动 / 只在一个方向动

先看是不是轴映射问题：侧倾（roll）必须丢弃，竖直只看 `raw[1]`。再看「椭圆半宽/半高」是不是比平时的头部摆幅还大——那样永远出不了圈。

### 左右摇头比上下点头慢

椭圆半宽调小（例如 `2.0°`）、平滑度压到 `30–40`、出圈加速速率提到 `80–100`。

### 耳机摘下就停

固件行为：两只耳机都戴上才继续上报。播放中断流超过 1.2 秒，程序会立刻停止移动光标，不会按最后角度一直跑。

### SmartScreen 提示未知发布者

没有代码签名。选择「更多信息 → 仍要运行」。

### Redmi 连接报「超时 / RFCOMM 端点为空」

桥接脚本会先尝试 `RfcommDeviceService.ConnectionHostName`，为空时用蓝牙地址重建 HostName。仍失败说明耳机没连上：戴上或开盖取出让 Windows 蓝牙显示「已连接」后重试。

---

## 诊断脚本

| 脚本 | 用途 |
|---|---|
| `headtrack/hid-doctor.ps1` | 四项状态汇总：SetupAPI 里可打开的 HID 接口数、Buds 音频端点（含 ACTIVE/UNPLUGGED）、`BTHENUM{00001124}` 父节点、`HID{00001124}` 子节点。退出码：0 接口可用 / 10 子节点存在但未 present / 11 子节点记录已缺失 / 12 子节点在但无接口 / 13 只连了 Hands-Free 模式 |
| `headtrack/hid-caps.ps1` | 打印该 collection 的 `usagePage/usage`、`inputReportLen/outputReportLen/featureReportLen`、link collection 数 |
| `headtrack/hid-probe.ps1` | 只用 SetupAPI + `HidD_GetAttributes` 枚举 `VID 0x0094 / PID 0x0004` 的可打开接口，与 `HidReader.exe` 判定口径一致 |
| `headtrack/hid-arm-test.ps1` | 独立探针：直接开接口、发 40 字节 arm 报告、带 1.5 秒超时读 6 次，输出 `reports_total` |
| `headtrack/list-audio.ps1` | 列出全部渲染端点（名称 + DeviceState），并跑一次 `hid-probe.ps1` |
| `headtrack/hid-state.ps1` | 现场 PnP 全量转储（只读） |
| `headtrack/restart-parent-once.ps1` | 对 `BTHENUM{00001124}` 父节点做一次 `pnputil /restart-device`（不碰适配器、不碰子节点、不循环） |
| `headtrack/diag.ps1` | PnP 转储（只读） |

日志位置：`%APPDATA%\budslink\headtrack.log`（头控与子进程全量日志）、`C:\ProgramData\BudsLink\adapter-restart.txt`（适配器重启记录）。

---

## 已知限制

1. **型号覆盖有限**：REDMI Buds 6 Pro 全功能通过；REDMI Buds 8 青春版 只确认了降噪 / 通透 / 触摸控制；其它型号未验证，协议字段与手势编码可能不兼容。
2. 头控依赖耳机固件暴露 HID over Bluetooth；部分固件/系统组合下 HID 集合根本不出现，此时 PC 侧没有可调项。
3. HIDP 通道断开或 HID 子节点记录缺失后，只有耳机的断开重连（不删配对）或冷启动能重新枚举出该子设备。
4. 原生 `<select>` 弹层在部分主题下仍是白底（Windows 不继承页面样式）。
5. 打包未工程化，版本号需要手工同步两处。
6. 未签名，SmartScreen 会提示。

---

## 致谢与许可

- Linux 原版 [BudsLink](https://github.com/maniacx/BudsLink) 作者 **maniacx**，本项目是它的 Windows 适配分支。
- 头部追踪的 HID 思路参考 **Wang2035/RedmiBuds6Pro-HeadTrack**（未直接复用其算法，滤波链为本项目自研）。
- 本项目以 **GPL-3.0-or-later** 发布，见 [LICENSE](LICENSE)。
- `headtrack/` 下的 `HidReader.exe`、`KeepAlive.exe` 来自 **Wang2035/RedmiBuds6Pro-HeadTrack**，按其自带的 MIT 许可（见 `headtrack/LICENSE`）分发。

本项目与小米、Redmi、漫步者无隶属关系，属第三方工具；协议解析基于公开抓包，改动设备设置的风险由使用者自担。
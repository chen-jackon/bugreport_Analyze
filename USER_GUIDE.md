# Bugreport Analyzer 使用指南

面向使用者的操作手册。开发者文档在 `docs/PROJECT.md`。

---

## 1. 安装

到 [Releases](https://github.com/chen-jackon/AiCoding/releases) 页下载对应平台的安装包。

| 平台 | 文件 | 说明 |
|---|---|---|
| macOS (Apple Silicon) | `Bugreport Analyzer-x.y.z-arm64.dmg` | 双击挂载，把 app 拖进 Applications |
| macOS (Intel) | `Bugreport Analyzer-x.y.z.dmg` | 同上 |
| Windows | `Bugreport Analyzer Setup x.y.z.exe` | 双击安装 |
| Windows (便携版) | `Bugreport Analyzer x.y.z.exe` | 解压即用，不写注册表 |
| Linux | `Bugreport Analyzer-x.y.z.AppImage` | `chmod +x` 后双击 |
| Linux (Debian/Ubuntu) | `bugreport-analyzer-app_x.y.z_amd64.deb` | `sudo dpkg -i ...deb` |

**macOS 首次启动**：未签名，会被 Gatekeeper 拦。在「系统设置 → 隐私与安全性」里点「仍要打开」即可（只需要一次）。

---

## 2. 第一次使用：分析一个 bugreport

应用打开后是空状态，两种方式开始：

**方式 A — 拖拽（最快）**
直接把 `bugreport-XXXX.zip` 拖到窗口里。会自动开始分析。

**方式 B — 按钮**
点头部的「📊 Analyze」→ 选 zip → 点 Run。

> 也可以拖一个**已经分析过的 `bra-out-*` 目录**进来，会跳过分析直接加载结果。

分析过程会显示一个进度模态框，里面是 `bra` CLI 的实时输出。一般几秒到十几秒。

---

## 3. 主界面

```
┌────────────────────────────────────────────────────────────────┐
│ Bugreport Analyzer    🟢 device-id    📊 📜 ⚙ 🎨 ☀         │  ← 头部
├──────────┬────────────────────────────────────┬───────────────┤
│          │                                    │               │
│ Module   │       Dashboard / FileViewer       │  🔍 Search    │
│ Tree     │                                    │  ⚠ Findings  │
│          │                                    │               │
└──────────┴────────────────────────────────────┴───────────────┘
```

- **左侧 Module Tree**：按模块分组的文件 + finding 列表。
- **中间 Dashboard / FileViewer**：分析结果总览或文件查看器。
- **右侧 Search & Findings**：全局搜索 + 命中规则的 findings。

> 右侧面板可以拖到**左/右/底**三个位置，每个位置的宽/高都会单独记忆。
> 不需要的话可以从「⚙ View」菜单里关掉 Search 或 Findings。

### 头部按钮

| 按钮 | 作用 | 快捷键 |
|---|---|---|
| 📊 Analyze | 重新分析一份 bugreport | Cmd+N |
| 📜 Rules | 编辑/管理 YAML 规则 | Cmd+R |
| ⚙ View | 显隐 Search / Findings 面板 | — |
| 🎨 Highlights | 自定义文件高亮规则（颜色/正则） | Cmd+K |
| ☀/🌙 | 切换深色/浅色 | — |

---

## 4. Dashboard

加载完成后默认显示 Dashboard，包含：

- **Modules 卡片网格**：每个模块一张卡，标题颜色表示健康度（绿=ok / 红=有问题）。点卡片进入模块详情。
- **Tombstones 表格**（红色卡片）：来自 `data/tombstones/*.txt`，列出崩溃的进程/PID/信号/出错的库与符号。点击行直接打开对应的 tombstone 文件。

### 模块详情页

进去后能看到该模块的：
- **Dumpsys sections** — 该模块抓到的 dumpsys 段落，点击直接看原文
- **HAL interfaces** — 涉及的 HAL（来自 lshal）
- **Properties** — 该模块相关的 system property
- **Issues / Hits** — 命中规则的关键日志行，点击跳转到原始日志的对应行
- **Log excerpts** — 按 log_tag 抓取的样例日志

---

## 5. 文件查看器（FileViewer）

点开任何文件（来自 module tree、dashboard、search 结果、finding）会进入 Monaco 编辑器：

- **自动跳转到目标行** + 蓝色高亮
- **Logcat 着色**：自动识别 `MM-DD HH:MM:SS.SSS PID TID L TAG` 格式，按 V/D/I/W/E/F 着色
- **自定义高亮**：默认对 `error` / `fail` / `fatal` / `warn` 标红。点头部 🎨 可以加正则规则
- **Cmd+F 在文件里查找** — Monaco 自带的查找框
- **左上角「← Dashboard」** 返回总览

> 每个文件最大可达几十万行，Monaco 完全能扛住，慢慢滚不卡。

---

## 6. 全局搜索（右侧面板）

| 操作 | 作用 |
|---|---|
| 输入框输入 + Enter / 点 Go | ripgrep 搜索整个 `bra-out` 目录 |
| ↑ / ↓ | 切换历史搜索词（保留 50 条） |
| Esc | 取消历史浏览，恢复正在输入的内容 |
| 🕘 按钮 | 弹出历史下拉，点击挑选；可单条删除或全清 |
| `.*` `Aa` `ab|` 三个按钮 | 切换正则 / 大小写敏感 / 整词匹配 |
| ✕（搜索中） | 取消当前搜索 |
| ⊟ / ⊞ | 折叠/展开所有文件结果 |
| 点结果中某一行 | 自动打开文件并跳转到该行 |

> 搜索结果按文件分组，每个文件最多展示 50 条命中（多的会显示「+N more」）。

---

## 7. Findings（问题列表）

右侧面板下半部分是 Findings。每条 finding 来自一条 YAML 规则的命中：

- 颜色按严重度分（CRITICAL/HIGH/MEDIUM/LOW）
- 点击直接跳到原始日志的对应行
- 标题来自规则的 `summary`，下面是匹配到的具体行

---

## 8. 自定义规则（📜 Rules）

> 这是最强大的功能——通过 YAML 规则告诉分析器「什么算问题」、「这个模块管哪些日志」。
> UI 里有表单可以编辑，但理解 YAML 结构能帮你写更精确的规则。

### 8.1 规则的作用

每次分析 bugreport 时，`bra` 会：

1. 读所有内置规则 + 你的用户规则
2. 按 **module pack** 分组：每个模块抓哪些 dumpsys 段、关心哪些进程/HAL/property/log tag
3. 用每条 **known_issue** 的正则去扫该模块对应的日志，命中就生成一条 finding
4. 用 **common rules** 跨模块扫常见的崩溃/ANR/watchdog 等

最终在 UI 上看到的「Issues / Findings」就是这些规则的命中结果。

### 8.2 在 UI 里编辑

1. 头部点 **📜 Rules** 打开管理器
2. 左侧列表：
   - **Built-in**（带 🔒 图标）：内置规则，只读
   - **Override**：你的覆盖（同名文件会覆盖内置）
   - **Custom**：你新建的，不与任何内置同名
3. 点条目 → 右侧表单 → 改 → **Save**
4. 改完弹出 toast「Rules changed. Re-analyze?」→ 点了就用新规则重跑当前 bugreport
5. **↺ Revert to built-in** 删除覆盖，恢复内置版本

用户规则保存路径：

| 系统 | 路径 |
|---|---|
| macOS | `~/Library/Application Support/bugreport-analyzer-app/rules/` |
| Windows | `%APPDATA%\bugreport-analyzer-app\rules\` |
| Linux | `~/.config/bugreport-analyzer-app/rules/` |

> 你也可以直接把 `.yaml` 文件丢到这个目录，重启 app 就会加载。

### 8.3 两种 schema（管理器自动识别）

打开任何 YAML 文件时，UI 看顶层结构来选表单：
- 顶层是 **对象 + 有 `module:` key** → **Module pack**
- 顶层是 **数组** → **Common rules**

不要在同一个文件里混写两种。

---

### 8.4 Module pack（模块知识包）

定义一个模块的「专家知识」——它管哪些日志、哪些进程、什么算问题。

完整示例（`audio.yaml`）：

```yaml
module: audio                    # 内部 ID（必须，全局唯一；同名会被合并）
display_name: 音频               # UI 上显示的中/英文名

# 该模块需要从 bugreport 抓哪些 dumpsys 段
dumpsys_sections:
  - DUMPSYS::media.audio_flinger
  - DUMPSYS::media.audio_policy
  - DUMPSYS::audio
  - DUMPSYS::media.metrics

# 该模块涉及的进程名（exact match）
processes:
  - audioserver
  - android.hardware.audio.service

# package name（Android app 包名，目前只用作展示）
packages: []

# 该模块关心的 logcat tag（精确匹配 TAG 字段）
log_tags:
  - AudioFlinger
  - AudioPolicy
  - AudioTrack

# 该模块相关的 system property 名（支持 fnmatch 通配符 *）
properties:
  - 'persist.vendor.audio.*'
  - 'audio.*'
  - 'ro.audio.*'

# 抽取的额外文件（路径相对 bugreport 根目录，glob）
files: []

# 涉及的 HAL 接口前缀（lshal 解析时用）
hal_interfaces:
  - android.hardware.audio
  - android.hardware.audio.effect

# 已知问题：扫该模块对应日志，命中就生成 finding
known_issues:
  - id: audio_hal_died               # 唯一 ID
    pattern: 'audio.*HAL.*died|audioserver.*FATAL'  # Python 正则（re.search）
    severity: CRITICAL               # CRITICAL / HIGH / MEDIUM / INFO
    summary: 'AudioHAL/audioserver 异常'   # 出现在 findings 列表里的标题
    context_lines: 40                # finding 详情里附带前后多少行上下文

  - id: audio_track_underrun
    pattern: 'underrun=[1-9][0-9]*|AudioTrack.*starv'
    severity: MEDIUM
    summary: 'AudioTrack underrun/欠载'
    context_lines: 20
```

**字段说明：**

| 字段 | 必填 | 类型 | 作用 |
|---|---|---|---|
| `module` | ✅ | string | 模块 ID，全局唯一；同名 YAML 会合并（用户文件追加内置数组项） |
| `display_name` | ❌ | string | UI 上显示用，省略则用 `module` |
| `dumpsys_sections` | ❌ | string[] | bugreport 里要抓的 dumpsys 段名，前缀 `DUMPSYS::` |
| `processes` | ❌ | string[] | 进程名（精确匹配） |
| `packages` | ❌ | string[] | Android 包名 |
| `properties` | ❌ | string[] | system property 名，支持 `*` 通配（fnmatch） |
| `log_tags` | ❌ | string[] | logcat tag |
| `files` | ❌ | string[] | 额外要抽取的文件路径 |
| `hal_interfaces` | ❌ | string[] | HAL 接口前缀，匹配 `lshal` 输出 |
| `known_issues` | ❌ | KnownIssue[] | 该模块的检测规则 |

**KnownIssue 字段：**

| 字段 | 必填 | 默认值 | 作用 |
|---|---|---|---|
| `id` | ✅ | — | 规则唯一 ID |
| `pattern` | ✅ | — | Python 正则，`re.search` 语义 |
| `severity` | ❌ | `MEDIUM` | `CRITICAL` / `HIGH` / `MEDIUM` / `INFO` |
| `summary` | ❌ | `""` | UI 上显示的标题 |
| `context_lines` | ❌ | `30` | finding 详情附带的前后行数 |

**合并行为**：如果用户文件和内置文件都有 `module: audio`，**两者会合并**——所有数组字段（dumpsys_sections / processes / known_issues 等）都是追加。所以你只想加一条规则不必抄整份内置文件。例如 `~/.config/.../rules/audio.yaml`：

```yaml
module: audio
known_issues:
  - id: my_custom_audio_glitch
    pattern: 'AudioPCM.*glitch.*count=([5-9]|[1-9][0-9]+)'
    severity: HIGH
    summary: '音频 PCM 抖动严重'
    context_lines: 25
```

只这几行，就在内置 audio 模块上加了一条规则。

---

### 8.5 Common rules（跨模块通用规则）

不属于具体模块的规则，比如「任何 SYSTEM LOG 里出现 FATAL EXCEPTION 就报警」。
顶层是数组，每条是一个 `CommonRule`。

完整示例：

```yaml
- id: fatal_exception            # 唯一 ID
  name: Java FATAL EXCEPTION     # 内部名（可选，目前没用到）
  severity: CRITICAL
  module: crash                  # 归到哪个模块（findings 列表里的分组）
  source: SYSTEM LOG             # 在哪个 section 里搜（见下方表格）
  pattern: 'FATAL EXCEPTION'
  context_lines: 40
  summary: 'Java 崩溃 (FATAL EXCEPTION)'

- id: anr_in
  name: ANR
  severity: HIGH
  module: crash
  source: SYSTEM LOG
  pattern: 'ANR in '
  context_lines: 40
  summary: 'ANR 应用无响应'

- id: kernel_panic
  severity: CRITICAL
  module: kernel
  source: KERNEL LOG
  pattern: 'Kernel panic|Unable to handle kernel'
  context_lines: 40
  summary: '内核 panic'
```

**字段说明：**

| 字段 | 必填 | 默认值 | 作用 |
|---|---|---|---|
| `id` | ✅ | — | 规则唯一 ID |
| `name` | ✅ | — | 规则名 |
| `severity` | ✅ | — | `CRITICAL` / `HIGH` / `MEDIUM` / `INFO` |
| `module` | ✅ | — | 归类到哪个模块（用于 findings 分组） |
| `source` | ✅ | — | 要扫的 section 名（见下方常用值） |
| `pattern` | ✅ | — | Python 正则 |
| `summary` | ❌ | `""` | UI 显示标题 |
| `context_lines` | ❌ | `30` | 上下文行数 |

**`source` 常用值：**

| `source` | 对应内容 |
|---|---|
| `SYSTEM LOG` | logcat -b system 全量日志 |
| `MAIN LOG` | logcat -b main |
| `KERNEL LOG` | dmesg / kernel 日志 |
| `EVENT LOG` | logcat -b events |
| `RADIO LOG` | logcat -b radio |
| `TOMBSTONE` | tombstone 文件内容 |
| `DUMPSYS` | dumpsys 全量输出 |

> `source` 是 bugreport 内 section 头部的字符串。如果你不确定，找一份 bugreport 的 raw `.txt`，看顶部的 `------ XXX ------` 横线分割块的标题。

---

### 8.6 正则写作要点

`pattern` 用 Python `re.search`（不是完全匹配；只要行内有一处匹配就算）。

**常见技巧：**

| 想匹配 | 写法 |
|---|---|
| 多个关键字任一 | `'FATAL\|crash\|panic'` |
| 大小写不敏感 | 写正则前缀 `(?i)`，例如 `'(?i)error'` |
| 数值大于 N | `'underrun=[1-9][0-9]*'`（≥10）；用字符类不要用 `+` 量词 |
| 进程名 + 错误 | `'audioserver.*FATAL'` |
| 整词 | 用 `\b`，例如 `'\bANR\b'` |
| 转义元字符 | `\.` 匹配 `.`，`\(` 匹配 `(` |

**避免的坑：**

- `.+` 在很长的行上会炸——优先用具体字符类
- 不要在 YAML 里用裸字符串写包含 `:` 或 `#` 的正则——用单引号包起来
- pattern 错误时 bra 会跳过该规则并打 warning，但不会让分析失败

**调试规则：**

```bash
# 只看你新加的规则的命中
bra ./bugreport.zip --rules ./my-rules/ -o /tmp/test
cat /tmp/test/data/findings.json | jq '.[] | select(.rule_id=="my_custom_audio_glitch")'
```

---

### 8.7 实战例子

**例 1：在内置 camera 模块上加一条规则**

文件：`~/Library/Application Support/bugreport-analyzer-app/rules/camera.yaml`

```yaml
module: camera
known_issues:
  - id: camera_hal3_session_failed
    pattern: 'CameraDeviceSession.*configureStreams.*returned -[0-9]+'
    severity: HIGH
    summary: 'Camera HAL3 会话配置失败'
    context_lines: 30
```

**例 2：完全自定义一个新模块**

文件：`~/.config/bugreport-analyzer-app/rules/wifi_p2p.yaml`

```yaml
module: wifi_p2p
display_name: Wi-Fi 直连
dumpsys_sections:
  - DUMPSYS::wifip2p
processes:
  - wpa_supplicant
log_tags:
  - WifiP2pService
  - SupplicantP2pIfaceCallback
properties:
  - 'persist.wifi.p2p.*'
known_issues:
  - id: p2p_group_failed
    pattern: 'P2P_GROUP_FORMATION_FAILURE|p2p.*group.*creation.*failed'
    severity: HIGH
    summary: 'Wi-Fi P2P 组创建失败'
    context_lines: 25
```

重启 app 或点 Re-analyze 后，左侧 module tree 会出现「Wi-Fi 直连」节点。

**例 3：跨模块的常见规则**

文件：`~/.config/bugreport-analyzer-app/rules/my_common.yaml`

```yaml
- id: my_oom_pattern
  name: OOM in our app
  severity: CRITICAL
  module: memory
  source: SYSTEM LOG
  pattern: 'OutOfMemoryError.*com\.mycompany\.myapp'
  context_lines: 60
  summary: '我们的 app 发生 OOM'

- id: my_native_assert
  name: Native assert
  severity: CRITICAL
  module: crash
  source: SYSTEM LOG
  pattern: 'libmycompany\.so.*ASSERT_FAILED'
  context_lines: 40
  summary: 'libmycompany.so 触发断言'
```

---

### 8.8 内置规则一览

`bra` 自带的内置规则覆盖：

| 模块 | 文件 | 主要内容 |
|---|---|---|
| audio | `audio.yaml` | AudioFlinger / AudioPolicy / underrun |
| battery | `battery.yaml` | 电量异常、充电问题 |
| bluetooth | `bluetooth.yaml` | BT 连接、配对、HFP/A2DP |
| camera | `camera.yaml` | Camera HAL、流配置失败 |
| display | `display.yaml` | SurfaceFlinger、合成器 |
| kernel | `kernel.yaml` | panic、OOM、I/O error |
| memory | `memory.yaml` | LMK、OOM、内存压力 |
| network | `network.yaml` | 连接失败、DNS、proxy |
| sensors | `sensors.yaml` | 传感器 HAL、SensorService |
| storage | `storage.yaml` | I/O 错误、挂载失败 |
| usb | `usb.yaml` | USB 模式、AOAP |
| **common** | `common.yaml` | 跨模块：FATAL EXCEPTION / ANR / kernel panic / WTF / watchdog 等 |

源码：[bugreport-analyzer/bra/rules/builtin/](https://github.com/chen-jackon/bugreport_Analyze)（在 public 仓里我没放，但你能在分析输出的 HTML 报告里看到所有规则）。

---

## 9. 高亮（🎨 Highlights）

文件查看器里的颜色高亮也能自定义：

- 默认 4 条：error / fail|failed|failure / fatal|crash|exception / warn|warning
- 点 🎨 → 加新规则 → 选颜色 + 输入文字或正则 + 是否区分大小写
- 改完立刻在打开的文件里生效

---

## 10. 面板布局

右侧 Search & Findings 可以挪位置：
- 在面板头部点 ◧ / ◨ / ⬓ 选 dock 位置
- 拖动面板边缘的灰条调整大小
- 头部右侧的 ◀ ▶ ▼ 折叠/展开
- 不想要某个面板？头部 ⚙ View → 取消勾选

所有布局选择都会持久化（每个 dock 位置单独记宽/高）。

---

## 11. 命令行用法

也支持纯 CLI（在 `bugreport-analyzer/` 装好后）：

```bash
# 分析 + 生成 HTML 报告
bra ./bugreport.zip -o ./bra-out

# 用自定义规则
bra ./bugreport.zip --rules ./my-rules/

# 看版本
bra --version
```

`bra-out/` 里会有：
- `data/{meta,modules,findings,tombstones}.json` ← 桌面 app 加载的就是这些
- `report.html` ← 静态 HTML 报告，直接浏览器打开
- `FS/`、`raw/` 等原始抽取出来的文件

---

## 12. 常见问题

**Q: 拖进 zip 没反应？**
检查文件是不是真的 `.zip`。其他格式（`.tar.gz` 等）目前不支持，可以解压后拖文件夹进来。

**Q: 分析进度卡在「Running …」很久？**
大的 bugreport 可能要 30 秒以上。看进度框里有没有错误输出。

**Q: 搜索结果点进去没跳到对应行？**
此问题已修复（v0.1.0 之后）。如果仍遇到请检查文件是否过大（>50MB 时 Monaco 加载会慢一点）。

**Q: macOS 报「文件已损坏」？**
未签名导致的。终端运行：
```bash
xattr -cr "/Applications/Bugreport Analyzer.app"
```
然后重新打开即可。

**Q: 我改了规则，但分析结果没变？**
点提示里的「Re-analyze」，或手动按 Cmd+N 重新分析当前 zip。规则只在分析时读取。

**Q: 想看具体的内置规则定义？**
源码在 `bugreport-analyzer/bra/rules/builtin/*.yaml`。

---

## 13. 数据存储位置

| 类型 | 路径（macOS / Linux / Windows） |
|---|---|
| 用户规则覆盖 | `~/Library/Application Support/bugreport-analyzer-app/rules/` <br> `~/.config/bugreport-analyzer-app/rules/` <br> `%APPDATA%\bugreport-analyzer-app\rules\` |
| 分析结果 | `$TMPDIR/bra-out-<timestamp>/`（系统临时目录，重启不保证保留） |
| 高亮规则、UI 设置 | 浏览器 localStorage（在 app 内部） |
| 搜索历史 | localStorage `search-history`（最多 50 条） |

要永久保留某次分析结果，把 `bra-out-*` 目录手动 copy 出来；下次拖文件夹进来即可重新加载。

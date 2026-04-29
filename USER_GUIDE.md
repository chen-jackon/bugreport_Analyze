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

`bra` 自带一套规则覆盖 audio / battery / camera / kernel 等常见模块。要扩展或覆盖：

1. 头部点 📜 Rules 打开编辑器
2. 看左侧列表：「Built-in」是只读的内置规则，「Override」是你的覆盖
3. 点任意一条 → 在右侧表单里改 → 点 Save
4. 改过的会保存到：
   - macOS: `~/Library/Application Support/bugreport-analyzer-app/rules/`
   - Windows: `%APPDATA%\bugreport-analyzer-app\rules\`
   - Linux: `~/.config/bugreport-analyzer-app/rules/`

> **不需要写 YAML**——表单里能直接配 pattern / severity / source 等字段。

### 两种规则类型（自动识别）

- **Module pack**：定义一个模块（dumpsys 段、processes、log_tags、known_issues 等）
- **Common rules**：跨模块的通用规则列表，每条带 `pattern + severity + source`（如 `SYSTEM LOG`/`KERNEL LOG`/`TOMBSTONE`）

改完规则会弹一个 toast：「Rules changed. Re-run analysis?」点 Re-analyze 就用新规则重跑当前 bugreport。

「↺ Revert to built-in」会删除你的覆盖，恢复内置版本。

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

# Bugreport Analyzer

Android bugreport 可视化分析工具的桌面应用（macOS / Windows / Linux）。

源代码在私有仓库，这个仓库只用来分发安装包。

## 下载

到 [Releases](https://github.com/chen-jackon/bugreport_Analyze/releases) 选最新版本下载对应平台的安装包：

| 平台 | 文件 |
|---|---|
| macOS (Apple Silicon) | `Bugreport Analyzer-x.y.z-arm64.dmg` |
| macOS (Intel) | `Bugreport Analyzer-x.y.z.dmg` |
| Windows 安装版 | `Bugreport Analyzer Setup x.y.z.exe` |
| Windows 便携版 | `Bugreport Analyzer x.y.z.exe` |
| Linux AppImage | `Bugreport Analyzer-x.y.z.AppImage` |
| Linux Debian | `bugreport-analyzer-app_x.y.z_amd64.deb` |

## 使用文档

完整使用说明见 [**USER_GUIDE.md**](./USER_GUIDE.md)：

- 安装与首次启动
- 拖拽分析 bugreport
- 主界面、Dashboard、文件查看器
- 全局搜索（含历史记录）
- Findings、自定义规则
- 高亮配置、面板布局
- CLI 用法
- 常见问题

## 主要功能

- 拖拽 `bugreport.zip` 自动分析，几秒出结果
- 模块化视图（audio / battery / camera / kernel / …）
- ripgrep 全局搜索 + 搜索历史（↑/↓ 切换）
- Monaco 编辑器查看大日志，自动识别 logcat 着色
- Tombstones 解析（pid / 信号 / 崩溃库 / 符号）
- 自定义 YAML 规则，UI 表单编辑无需手写

## 反馈

问题或建议请通过私有源码仓库的 issue（需要权限）或邮件联系。

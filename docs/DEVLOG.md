# DEVLOG

## 2026-04-08

### 项目初始化
- 明确产品方向：macOS Markdown 编辑器（Typora 风格 + 代码块折叠）
- 选择技术路线：Tauri v2
- 安装技能：`tauri-v2`、`macos-design-guidelines`、`asc-notarization`
- 初始化 Git 仓库
- 建立文档体系：`README.md`、`docs/DEVLOG.md`、`docs/CHANGELOG.md`

### 备注
- 后续每次功能开发结束后追加日志，记录设计决策、实现方法、测试结果和遗留问题。

## 2026-04-09

### 接口文档
- 基于 PRD 输出 Tauri v2 接口文档：`docs/API_SPEC.md`
- 设计 command 接口（文件读写、编辑内容同步、代码块折叠、查找替换、设置管理）
- 设计 event 接口（自动保存状态、脏状态、折叠状态变化）
- 统一错误码与返回结构，便于前端一致处理

### 设计决策
- 采用 `invoke` + `#[tauri::command]` 作为主交互模式
- 折叠状态按 `documentId + blockId` 持久化，支持单块与全量折叠
- 自动保存通过事件回传状态，前端只负责 UI 呈现

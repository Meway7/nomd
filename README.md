# nomd

一个面向 macOS 的 Markdown 编辑器项目，目标体验为：

- 像 Typora 一样「单栏所见即编辑」
- 支持像 Obsidian/语雀一样的代码块折叠
- UI 风格简洁，内容优先

## 当前阶段

- 阶段：需求确认 / PRD 已完成
- 技术路线：Tauri v2（桌面）
- 已安装技能：`tauri-v2`、`macos-design-guidelines`、`asc-notarization`

## 开发流程约定

1. 需求确认 -> 任务拆分 -> 实现 -> 验证 -> 文档更新 -> 提交
2. 每次开发都更新：
   - `README.md`（项目现状）
   - `docs/DEVLOG.md`（开发过程日志）
   - `docs/CHANGELOG.md`（变更摘要）
3. 每个里程碑至少一个可运行版本与对应 git commit

## 仓库规范

- 分支命名：`codex/<feature-name>`（默认）
- 提交信息：`type(scope): summary`
- 常用 type：`feat` `fix` `docs` `refactor` `chore`

## 下一步

- 初始化 Tauri 项目骨架
- 基于 `docs/API_SPEC.md` 实现首批 command 与事件
- 搭建 Markdown 编辑器 MVP（单栏编辑预览 + 代码块折叠）

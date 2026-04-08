# API 接口文档（Tauri v2）

## 1. 范围与目标

本文档定义 `nomd` 在 MVP 阶段的前后端接口，采用 Tauri v2 command + event 模式，覆盖：

- Markdown 文件打开/保存
- 单栏编辑内容读写
- 代码块折叠状态持久化
- 编辑器基础能力（查找替换、字数统计）

说明：当前为桌面端（macOS）优先设计，但接口保持跨平台可扩展。

## 2. 技术约定

- 前端调用：`@tauri-apps/api/core` 的 `invoke`
- 后端实现：Rust `#[tauri::command]`
- 事件机制：`emit` / `listen`
- 返回结构：统一 `Result<T, ApiError>`（前端映射为异常或标准错误对象）
- 时间格式：ISO 8601（UTC）
- 路径：绝对路径字符串（macOS）

## 3. 通用数据结构

### 3.1 ApiError

```ts
interface ApiError {
  code: string;
  message: string;
  detail?: string;
}
```

### 3.2 EditorDocument

```ts
interface EditorDocument {
  id: string;               // 文档会话 ID（UUID）
  path: string | null;      // 未保存新文档时为 null
  title: string;            // 展示标题
  content: string;          // Markdown 原文
  isDirty: boolean;         // 是否有未保存改动
  updatedAt: string;        // ISO 时间
}
```

### 3.3 CodeBlockFoldState

```ts
interface CodeBlockFoldState {
  blockId: string;          // 代码块稳定 ID（编辑器解析层生成）
  collapsed: boolean;
}
```

### 3.4 DocumentStats

```ts
interface DocumentStats {
  charCount: number;
  wordCount: number;
  lineCount: number;
  estimatedReadMinutes: number;
}
```

## 4. Command 接口定义

## 4.1 文件与文档会话

### `new_document`

创建空文档会话。

- Request: `{}`
- Response: `EditorDocument`

### `open_document`

打开指定 Markdown 文件。

```ts
interface OpenDocumentRequest {
  path: string;
}
```

- Response: `EditorDocument`
- 失败场景：文件不存在、无权限、编码无法解析

### `save_document`

保存当前文档到现有路径（path 不能为空）。

```ts
interface SaveDocumentRequest {
  id: string;
  content: string;
}
```

- Response:

```ts
interface SaveDocumentResponse {
  id: string;
  path: string;
  updatedAt: string;
}
```

### `save_document_as`

另存为新路径。

```ts
interface SaveDocumentAsRequest {
  id: string;
  content: string;
  targetPath: string;
}
```

- Response: `SaveDocumentResponse`

### `list_recent_documents`

读取最近文档列表（最多 20 条）。

```ts
interface RecentDocumentItem {
  path: string;
  title: string;
  lastOpenedAt: string;
}
```

- Response: `RecentDocumentItem[]`

## 4.2 编辑器内容

### `update_document_content`

更新文档内容到内存会话（用于输入过程中状态同步）。

```ts
interface UpdateDocumentContentRequest {
  id: string;
  content: string;
}
```

- Response:

```ts
interface UpdateDocumentContentResponse {
  id: string;
  isDirty: boolean;
  updatedAt: string;
}
```

### `get_document_stats`

计算当前文档统计信息。

```ts
interface GetDocumentStatsRequest {
  id: string;
}
```

- Response: `DocumentStats`

## 4.3 代码块折叠

### `set_code_block_fold_state`

设置单个代码块折叠状态。

```ts
interface SetCodeBlockFoldStateRequest {
  documentId: string;
  blockId: string;
  collapsed: boolean;
}
```

- Response: `CodeBlockFoldState`

### `set_all_code_blocks_fold_state`

批量折叠/展开当前文档全部代码块。

```ts
interface SetAllCodeBlocksFoldStateRequest {
  documentId: string;
  collapsed: boolean;
}
```

- Response:

```ts
interface SetAllCodeBlocksFoldStateResponse {
  documentId: string;
  affected: number;
  collapsed: boolean;
}
```

### `get_code_block_fold_states`

读取当前文档代码块折叠状态。

```ts
interface GetCodeBlockFoldStatesRequest {
  documentId: string;
}
```

- Response: `CodeBlockFoldState[]`

## 4.4 查找替换（MVP 基础）

### `find_in_document`

```ts
interface FindInDocumentRequest {
  id: string;
  query: string;
  caseSensitive?: boolean;
  regex?: boolean;
}

interface FindMatch {
  start: number;
  end: number;
}
```

- Response: `FindMatch[]`

### `replace_in_document`

```ts
interface ReplaceInDocumentRequest {
  id: string;
  query: string;
  replaceWith: string;
  replaceAll?: boolean;
  caseSensitive?: boolean;
  regex?: boolean;
}

interface ReplaceInDocumentResponse {
  replacedCount: number;
  content: string;
}
```

- Response: `ReplaceInDocumentResponse`

## 4.5 设置项

### `get_settings`

```ts
interface AppSettings {
  autoSave: boolean;              // 默认 true
  autoSaveIntervalMs: number;     // 默认 2000
  theme: 'system' | 'light' | 'dark';
  editorMaxWidth: number;         // Typora 风格居中阅读宽度
}
```

- Response: `AppSettings`

### `update_settings`

- Request: `Partial<AppSettings>`
- Response: `AppSettings`

## 5. Event 事件定义

### `document://autosave-status`

自动保存状态变更事件。

```ts
interface AutosaveStatusEvent {
  documentId: string;
  status: 'idle' | 'saving' | 'saved' | 'error';
  at: string;
  message?: string;
}
```

### `document://dirty-changed`

文档脏状态变化。

```ts
interface DirtyChangedEvent {
  documentId: string;
  isDirty: boolean;
}
```

### `editor://fold-state-changed`

代码块折叠状态变化。

```ts
interface FoldStateChangedEvent {
  documentId: string;
  blockId: string;
  collapsed: boolean;
}
```

## 6. 错误码规范

| Code | 含义 |
| --- | --- |
| `DOCUMENT_NOT_FOUND` | 文档或会话不存在 |
| `FILE_NOT_FOUND` | 文件不存在 |
| `FILE_PERMISSION_DENIED` | 文件无读写权限 |
| `FILE_ENCODING_UNSUPPORTED` | 文件编码不支持 |
| `SAVE_PATH_REQUIRED` | 调用保存但文档无路径 |
| `INVALID_ARGUMENT` | 参数非法 |
| `INTERNAL_ERROR` | 未分类内部错误 |

## 7. 前端调用示例（Tauri v2）

```ts
import { invoke } from '@tauri-apps/api/core';

export async function openDocument(path: string) {
  return invoke<EditorDocument>('open_document', { path });
}

export async function saveDocument(id: string, content: string) {
  return invoke<SaveDocumentResponse>('save_document', { id, content });
}

export async function toggleBlock(documentId: string, blockId: string, collapsed: boolean) {
  return invoke<CodeBlockFoldState>('set_code_block_fold_state', {
    documentId,
    blockId,
    collapsed,
  });
}
```

## 8. Rust Command 注册清单（建议）

`src-tauri/src/lib.rs` 中至少注册以下命令：

- `new_document`
- `open_document`
- `save_document`
- `save_document_as`
- `list_recent_documents`
- `update_document_content`
- `get_document_stats`
- `set_code_block_fold_state`
- `set_all_code_blocks_fold_state`
- `get_code_block_fold_states`
- `find_in_document`
- `replace_in_document`
- `get_settings`
- `update_settings`

## 9. 与 PRD 的映射

- PRD 核心需求「单栏所见即编辑」-> `update_document_content` + `save_document*` + `document://autosave-status`
- PRD 核心需求「代码块折叠」-> `set_code_block_fold_state` / `set_all_code_blocks_fold_state` / `get_code_block_fold_states`
- PRD 基础能力「查找替换、字数统计」-> `find_in_document` / `replace_in_document` / `get_document_stats`
- PRD 文件工作流 -> `open_document` / `save_document` / `save_document_as` / `list_recent_documents`

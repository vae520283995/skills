---
name: confluence-sync
description: 在本地 Markdown 文件和 Confluence 页面之间同步文档。支持 push（上传）和 pull（下载）。
---

# Confluence 文档同步

使用 Confluence MCP 工具在本地 Markdown 文件和 Confluence 页面之间同步文档。

## 核心命令

### push - 本地 → Confluence

将本地 `.md` 文件推送到 Confluence，支持创建新页面或更新现有页面。

**用法**：
```
/confluence-sync push <file_path> --space <space_key> [--parent <page_id>] [--update]
```

**参数**：
- `file_path`: 本地 Markdown 文件路径（必填）
- `space`: Confluence space key，如 `ORDER`（必填）
- `parent`: 父页面 ID，可选（不指定则创建为顶级页面）
- `update`: 可选，指定此参数则更新已存在的页面

**示例**：
```bash
# 创建新页面（顶级）
/confluence-sync push ~/docs/guide.md --space ORDER

# 创建为某页面的子页面
/confluence-sync push ~/docs/guide.md --space ORDER --parent 4397691

# 更新已存在的页面
/confluence-sync push ~/docs/guide.md --space ORDER --parent 4397691 --update
```

### pull - Confluence → 本地

从 Confluence 下载页面内容保存为本地 Markdown 文件。

**用法**：
```
/confluence-sync pull <page_id> --output <dir_path>
```

**参数**：
- `page_id`: Confluence 页面 ID（必填）
- `output`: 本地输出目录路径（必填）

**示例**：
```bash
# 下载到指定目录
/confluence-sync pull 4397691 --output ~/docs

# 下载并以页面标题命名文件
/confluence-sync pull 4397691 --output ~/docs
```

### list - 列出子页面

列出某个页面下的所有子页面，方便查找页面 ID。

**用法**：
```
/confluence-sync list <page_id>
```

**参数**：
- `page_id`: 父页面 ID（可选，不指定则列出 space 根页面）

**示例**：
```bash
# 列出 Team Org 下的子页面
/confluence-sync list 4397691

# 列出 ORDER space 的顶级页面
/confluence-sync list
```

## 工作流程

### push 执行流程

1. **读取本地文件**：读取 `.md` 文件内容，提取文件名作为页面标题
2. **检查父页面**：验证指定的 space 和 parent page 是否存在
3. **判断操作类型**：
   - 无 `--update`：创建新页面
   - 有 `--update`：更新现有页面
4. **执行操作**：调用 `confluence_create_page` 或 `confluence_update_page`
5. **返回结果**：输出创建/更新的页面链接

### pull 执行流程

1. **获取页面内容**：调用 `confluence_get_page` 获取页面
2. **转换格式**：将 Confluence 内容转换为 Markdown
3. **保存文件**：以页面标题为文件名保存到指定目录
4. **返回结果**：输出本地文件路径

## 注意事项

- **文件格式**：仅支持 Markdown 格式
- **图片处理**：本地图片链接会保持原样，如需上传图片到 Confluence 附件需额外处理
- **权限**：需要 Confluence 空间的编辑/创建权限
- **标题提取**：从文件名提取标题（去掉 `.md` 后缀）

## 使用场景

1. **本地编写文档**：在本地用 Markdown 编辑文档
2. **一键同步**：推送到 Confluence 与团队共享
3. **协作编辑**：从 Confluence 下载修改后的文档继续编辑

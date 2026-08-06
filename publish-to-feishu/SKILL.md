---
name: publish-to-feishu
description: >-
  发布本地 Markdown（含相对路径图片）到飞书文档，或将飞书文档导出为本地 Markdown。
  支持新建/覆盖、图片自动上传、OAuth 授权、发布到云盘文件夹/Wiki 根目录/Wiki 指定页面、
  GitHub Alert 与飞书高亮块互转、飞书画板与 JSON 代码块双向同步。
  当用户说「把 xxx.md 发布到飞书」「发布到飞书」「同步到飞书文档」「导出飞书文档」，
  或在项目发布流程中需要把 Markdown 写入飞书时，都应使用本技能。
---

# 发布 Markdown 到飞书（publish-to-feishu）

通过 **`publish-to-feishu`** CLI 在本地 Markdown 与飞书文档之间双向同步：
发布时把 Markdown（含相对路径图片）写入飞书文档；导出时把飞书文档拉回本地 Markdown。

## 何时使用本技能

- 用户说「把 `xxx.md` 发布到飞书」「发布到飞书」「同步到飞书文档」。
- 用户说「导出飞书文档」「把飞书文档 `xxx` 拉下来」。
- 在 `publish-prd` 等组合流程中，需要把单个 PRD 写入飞书的步骤。

## 工具安装与配置

### 安装

要求 Node.js **18+**。源码在 [publish-to-feishu 仓库](https://github.com/cyrasafia/publish-to-feishu)。

推荐方式：克隆仓库后 `npm link`，获得全局 `publish-to-feishu` 命令：

```bash
cd <publish-to-feishu 源码目录>
npm install
npm link
```

之后在任意目录均可使用 `publish-to-feishu "path/to/doc.md"`。取消全局链接：`npm unlink -g publish-to-feishu`。

其他安装方式：

```bash
# 从 Git 仓库全局安装
npm install -g git+ssh://git@github.com:cyrasafia/publish-to-feishu.git
```

未安装且仅在本仓库内使用时，可回退到 `node publish-to-feishu.js`，但需工作目录能访问 `publish-config.json` 与脚本。

### 配置

1. 复制示例配置并填写应用凭证与默认发布位置：

   ```bash
   cp publish-config.example.json publish-config.json
   # 全局命令场景，放到默认目录：
   mkdir -p ~/.config/publish-to-feishu
   cp publish-config.example.json ~/.config/publish-to-feishu/publish-config.json
   ```

   关键字段：
   - `default_publish_path`：默认发布路径，优先级高于 `default_folder_token`。
     - `drive:<folder_token>`：发布到云盘文件夹
     - `wiki-root:<space_id>`：发布到指定 Wiki 空间根目录
     - `wiki-under:<node_token>`：发布到指定 Wiki 页面下
   - `default_folder_token`：**已废弃**，仅兼容旧逻辑。
   - `app_id`、`app_secret`：不配置时使用内置默认值。

2. 首次使用前完成 OAuth 授权（获取并缓存 `user_access_token`）：

   ```bash
   # 推荐：自动拉起浏览器 + 本地回调自动获取 code
   publish-to-feishu auth

   # 备用：手动粘贴回调 URL
   publish-to-feishu auth --manual-url "<callback_url>"

   # 兼容：直接传 code
   publish-to-feishu auth --code <oauth_code>
   ```

   `--oauth-redirect-uri` / `--oauth-scopes` 仅对当次授权生效，不会持久化。

3. 配置文件查找顺序：
   - 当前目录下 `publish-config.json`（仓库内运行 `node publish-to-feishu.js` 时优先）。
   - `~/.config/publish-to-feishu/publish-config.json`（全局命令且当前目录无配置时）。
   - 缓存：`~/.config/publish-to-feishu/.uat-cache.json`。
   - 可用 `XDG_CONFIG_HOME` 改变 `~/.config` 基准目录。

4. 敏感文件（`publish-config.json`、`.uat-cache.json`）已列入 `.gitignore`，请勿提交。

## 执行步骤

1. **解析 Markdown 路径**：从用户文本中提取 Markdown 文件路径（支持相对路径，如 `sample/酒店详情页.md`）。路径缺失时向用户询问。

2. **选择执行命令**，优先使用全局命令：
   - `publish-to-feishu "<markdown_path>"`（CLI 在 PATH 上时）
   - 回退：`node publish-to-feishu.js "<markdown_path>"`（仅当该文件存在于仓库中）

3. **仅在用户明确要求时**追加以下参数：
   - `--doc-id <id>`：覆盖指定文档（否则按 doc-id 自动绑定规则处理，见下文）。
   - `--publish-path "drive:<folder_token>"` / `"wiki-root:<space_id>"` / `"wiki-under:<node_token>"`：新建时指定发布位置，优先级高于配置。
   - `--folder-token <token>`：**已废弃**，仅兼容旧逻辑，推荐用 `--publish-path`。
   - `--dry-run`：仅校验、不写入飞书。

4. **执行后报告关键输出**：
   - 飞书文档 URL（优先返回可访问分享地址；查询失败时回退 `https://www.feishu.cn/docx/<doc_id>`）。
   - 图片上传数量。
   - 任何 API/token 错误。

## doc-id 自动绑定

发布成功后，脚本会把文档 ID 写回源 Markdown 文件末尾：

```html
<!-- feishu-doc-id: <doc_id> -->
```

下次再发布时：
- 未显式传 `--doc-id` → 优先从该注释读取 `doc_id`，覆盖同一飞书文档（即使文件被移动/复制）。
- 在实际发布（convert）前会自动移除这条尾部注释，避免它进入飞书正文。

## 发布路径与优先级

- 覆盖发布（传入 `--doc-id` 或文件末尾有 `<!-- feishu-doc-id: ... -->`）**忽略**发布路径配置，直接覆盖。
- 新建发布时路径优先级：
  1. `--publish-path`
  2. `publish-config.json` 中 `default_publish_path`
  3. 兼容旧逻辑（已废弃）：`--folder-token` / `default_folder_token`
- 全部未指定时，自动发布到「我的文档库（Wiki）根目录」，无需额外配置。

## 导出飞书文档到本地 Markdown

```bash
publish-to-feishu export <doc_id> [output.md | output-dir]
```

- 不传第二参数：在当前目录写入「飞书文档标题.md」。
- 第二参数为目录：在该目录下写入「飞书文档标题.md」（目录不存在会自动创建）。
- 第二参数为 `.md` 文件路径：写入该文件。
- 飞书高亮块导出为 GitHub `> [!NOTE]` 等 Alert 语法。
- 图片下载到输出 Markdown 同目录下的 `assets/`，引用写回 `![](assets/xxx.png)`。
- 文件末尾追加 `<!-- feishu-doc-id: <doc_id> -->`。

## 支持的图片语法

发布时识别三种图片引用（路径均为相对 Markdown 文件的本地路径）：

| 语法 | 示例 | 说明 |
| --- | --- | --- |
| 标准 Markdown | `![alt](assets/xxx.png)` | 最常用 |
| Obsidian | `![[assets/xxx.png]]` | 自动转换为标准语法 |
| HTML `<img>` | `<img src="assets/xxx.jpg" width="525">` | `width` 作为飞书图片块显示宽度上限 |

三者可混用，均会被提取并上传为飞书图片块。导出时统一写回标准 Markdown 语法。

## GitHub Alert 与飞书高亮块

- 发布：连续引用块首行若为 `[!NOTE]`/`[!TIP]`/`[!IMPORTANT]`/`[!WARNING]`/`[!CAUTION]`（不区分大小写），转为飞书高亮块（callout）。可用 `> [!NOTE] 可选标题` + 后续 `>` 正文行。
- 导出：高亮块写回上述 GitHub Alert 语法。
- 限制：在飞书手动改过高亮块颜色时，导出按背景色反推类型，可能与原始不完全一致；无法匹配时按 `NOTE` 输出。

## 飞书画板与 JSON 代码块

- 导出：飞书画板导出为带 `"feishu_whiteboard": "1.0"` 标记的 ` ```json ` 代码块，含节点原始 JSON。
- 发布：Markdown 中含上述 schema 的 ` ```json ` 块会创建飞书画板并写入节点；普通 JSON 代码块不受影响。
- 权限：需应用具备 `board:whiteboard:node:read` 与 `board:whiteboard:node:create`，升级后请重新 `publish-to-feishu auth`。
- 导入策略：envelope 含 `syntax` 节点时，先用 plantuml API 导入源码，再追加其余 raw 节点（自动排除 syntax 展开的 mind_map 子树）。
- 限制：
  - 画板 image 节点跨文档还原可能失效，需手动重传。
  - 节点数量大时分批写入（每批最多 50 个）。
  - 经 plantuml 导入的思维导图再导出通常不会带回 `syntax` 包裹节点，属预期。
  - 复杂 raw 节点 roundtrip 后可能有字段漂移。

## 常见问题

| 现象 | 处理 |
| --- | --- |
| `publish-to-feishu` 命令不存在 | 按上文「安装」`npm link`，或回退 `node publish-to-feishu.js`（需在仓库目录） |
| token 过期 / 401 | 重新执行 `publish-to-feishu auth` |
| 发布到错误文档 | 检查文件末尾 `<!-- feishu-doc-id: ... -->`，或用 `--doc-id` 显式指定 |
| 图片未上传 | 确认图片为相对 Markdown 文件的本地路径，三种语法之一 |
| 覆盖发布不生效 | 确认未误删尾部 doc-id 注释，或显式传 `--doc-id` |

## 关联技能

| 技能 | 关系 |
| --- | --- |
| [publish-prd](../publish-prd/SKILL.md) | 在项目发布流程的「PRD 发布到飞书」步骤调用本技能 |
| [feishu-project-create-requirement](../feishu-project-create-requirement/SKILL.md) | 从已发布的飞书文档创建需求工作项 |
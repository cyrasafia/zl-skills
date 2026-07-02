---
name: publish-prd
description: >-
  一键发布项目到飞书文档、原型服务器和文档仓库（Codeup Git）。
  按顺序完成 Git 初始化/关联、原型发布、PRD 补全与飞书发布、README 生成与推送。
  当用户说"发布项目"、"同步到飞书"、"发布PRD"、"publish"时触发。
---

# 发布项目（publish-prd）

将项目**同步发布**到三个目标：**原型服务器**、**飞书文档**、**Codeup Git 仓库**。

## 何时使用本技能

- 用户说"发布项目"、"同步到飞书"、"发布PRD"、"publish"、"推送到远程"。
- 用户说"把项目上线到原型服务器"。
- 用户说"初始化并关联远程仓库"。
- 用户说"补全 PRD 并发布"。

## 前置条件

| 工具/技能 | 用途 | 检查方式 |
|-----------|------|----------|
| `git` | 版本管理与推送 | `git --version` |
| `codeup` CLI | Codeup 仓库管理 | `codeup --help` |
| `publish-rp` | 原型发布到原型服务器 | `publish-rp --help` |
| `publish-to-feishu` | Markdown 发布到飞书文档 | `publish-to-feishu --help` |

缺失工具时提示用户安装或跳过对应步骤。

## 执行步骤

按顺序执行以下四个步骤。每个步骤执行前先进行条件检查，不满足时**跳过并记录**，不中断后续流程。全部完成后输出**执行结果摘要**。

---

### 步骤一：Git 初始化与远程关联

**目标**：确保项目已初始化 Git 且已关联 Codeup 远程仓库。

#### 1.1 检查 Git 初始化

```bash
git rev-parse --is-inside-work-tree 2>/dev/null
```

- 返回 `true` → 已初始化，跳到 1.2
- 返回非 `true` 或报错 → 执行 `git init`

#### 1.2 检查远程仓库

```bash
git remote get-url origin 2>/dev/null
```

- 有输出 → 远程已关联，跳到步骤二
- 无输出 → 需创建并关联远程仓库

#### 1.3 创建远程仓库并关联

1. 确定 repo 名称：优先使用**项目根目录文件夹名称**（`basename` of CWD）。
   - 名称包含中文、特殊字符、空格时，根据项目内容选择合适的英文 repo 名称（如 `hotel-detail-page`）。
2. 使用 **`codeup-repo`** 技能创建远程仓库：

```bash
codeup repo create <repo-name> --json
```

3. 从 `--json` 输出中提取 `sshUrlToRepo`。
4. 关联远程并推送：

```bash
git remote add origin <sshUrlToRepo>
git push -u origin master   # 或 main，以本地分支名为准
```

**注意**：`git push` 需要访问 SSH 密钥，必须在沙箱外执行。

---

### 步骤二：发布原型到原型服务器

**目标**：将 `prototype/` 目录下的 HTML 原型发布到原型服务器。

#### 2.1 条件检查

检查项目根目录下是否存在 `prototype/` 文件夹且其中包含 `.html` 文件：

```bash
find prototype/ -name "*.html" -type f 2>/dev/null | head -1
```

- 有输出 → 存在原型，继续 2.2
- 无输出 → 无原型，**跳过本步骤**，记录「无原型，已跳过」

#### 2.2 执行发布

```bash
publish-rp ./prototype
```

- 记录输出中的**预览链接**（通常为 `http://rp.histar.com/<repo-name>/`），后续步骤三需要用到。

---

### 步骤三：补全 PRD 并发布到飞书

**目标**：为 `prd/` 下的所有 Markdown PRD 补全开头 NOTE 及原型预览链接，然后发布到飞书。

#### 3.1 条件检查

检查项目根目录下是否存在 `prd/` 文件夹且其中包含 `.md` 文件：

```bash
find prd/ -name "*.md" -type f 2>/dev/null | head -1
```

- 有输出 → 存在 PRD，继续 3.2
- 无输出 → 无 PRD，**跳过本步骤**，记录「无 PRD，已跳过」

#### 3.2 补全 PRD

对 `prd/` 下**所有** `.md` 文件逐个执行以下操作：

1. **补全 NOTE**：使用 **`write-prd`** 技能的逻辑，为每个 PRD 补全标题下方的 `> [!NOTE]` 文档仓库说明（如果有远程仓库的话）。
2. **补全原型预览链接**：如果步骤二成功获取到了预览链接，在每个 PRD 的「参考链接」章节中添加原型预览链接。若没有「参考链接」章节则新增。

#### 3.3 发布到飞书

使用 **`publish-to-feishu`** 技能，逐个发布 `prd/` 下的 `.md` 文件：

```bash
publish-to-feishu "<markdown_path>"
```

- 记录每个文件的**飞书文档 URL**。
- 若发布失败，记录错误信息但不中断其他文件的发布。

---

### 步骤四：生成 README 并推送所有变更

**目标**：生成或更新项目 README，提交并推送全部改动。

#### 4.1 生成或更新 README.md

根据项目当前内容生成 `README.md`，内容应包含：

- **项目名称**：取项目根目录文件夹名称
- **项目简介**：根据 PRD 内容或项目文件结构生成简短描述（1-3 句）
- **文档仓库链接**：若有远程仓库，显示其 HTTPS 地址
- **原型预览链接**：若步骤二成功，显示预览 URL
- **飞书文档链接**：若步骤三成功，列出已发布的飞书文档 URL

如果 `README.md` 已存在，读取现有内容并在保留用户自定义内容的前提下更新上述信息。

#### 4.2 提交并推送

```bash
git add -A
git commit -m "docs: 发布项目 - 同步PRD、原型与文档"
git push origin master   # 或 main
```

- 若没有任何文件变更（`git status` 干净），跳过提交。
- **注意**：`git push` 需要在沙箱外执行，否则无法读取 SSH 密钥。

---

## 执行结果摘要

全部步骤执行完成后，输出如下格式的摘要：

```
## 发布结果

| 步骤 | 状态 | 说明 |
|------|------|------|
| Git 初始化与远程关联 | ✅/⏭️ | ... |
| 原型发布 | ✅/⏭️ | 预览链接：... |
| PRD 补全与飞书发布 | ✅/⏭️ | 飞书链接：... |
| README 与推送 | ✅/⏭️ | ... |
```

- ✅ 已执行
- ⏭️ 已跳过（无相关内容）
- ❌ 执行失败（附错误信息）

## 关联技能

| 技能 | 关系 |
|------|------|
| [codeup-repo](../codeup-repo/SKILL.md) | 仓库创建与远程关联（步骤一） |
| [write-prd](../write-prd/SKILL.md) | PRD 补全 NOTE 与原型链接（步骤三） |
| [publish-to-feishu](../publish-to-feishu/SKILL.md) | Markdown 发布到飞书文档（步骤三） |

## 常见问题

| 现象 | 处理 |
|------|------|
| `codeup` 命令不存在 | 提示安装 codeup-cli 或跳过步骤一 |
| `publish-rp` 命令不存在 | 提示安装或跳过步骤二 |
| `publish-to-feishu` 命令不存在 | 提示安装或跳过步骤三 |
| `git push` 失败（Permission denied） | SSH 密钥未配置，提示在沙箱外执行 |
| 403 创建仓库 | PAT 权限不足，提示检查 codeup config |
| 飞书发布失败（token 过期） | 提示更新飞书 token |

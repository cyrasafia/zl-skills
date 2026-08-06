---
name: publish-prd
description: >-
  一键发布项目到飞书文档、原型服务器和文档仓库（Codeup Git）。
  按顺序完成 Git 初始化/关联、原型发布、PRD 补全与飞书发布、README 生成与推送。
  当用户说"发布项目"、"同步到飞书"、"发布PRD"、"publish"、"把项目上线到原型服务器"、
  "初始化并关联远程仓库"、"补全 PRD 并发布"时，都应使用本技能。
---

# 发布项目（publish-prd）

将项目**同步发布**到三个目标：**原型服务器**、**飞书文档**、**Codeup Git 仓库**。

本技能是一个**编排技能**：它按顺序串联多个子技能，本身不直接操作具体工具。每个步骤的实际执行委托给对应的子技能，由它们负责工具调用、错误处理与安装引导。这样工具的升级与安装说明集中在各自技能里维护，避免重复。

## 何时使用本技能

- 用户说"发布项目"、"同步到飞书"、"发布PRD"、"publish"、"推送到远程"。
- 用户说"把项目上线到原型服务器"。
- 用户说"初始化并关联远程仓库"。
- 用户说"补全 PRD 并发布"。

## 依赖的子技能

| 子技能 | 负责的步骤 | 说明 |
|--------|------------|------|
| [codeup-repo](../codeup-repo/SKILL.md) | 步骤一：Git 初始化与远程关联 | 仓库创建、远程关联、SSH 配置均由该技能处理 |
| `publish-rp`（CLI） | 步骤二：原型发布 | 见下文「publish-rp 安装说明」 |
| [write-prd](../write-prd/SKILL.md) | 步骤三：补全 PRD NOTE 与原型链接 | PRD 文档结构补全由该技能处理 |
| [publish-to-feishu](../publish-to-feishu/SKILL.md) | 步骤三：PRD 发布到飞书 | Markdown → 飞书文档、安装/配置/错误处理均由该技能处理 |

缺失某子技能所依赖的工具时，**不要**自行猜测安装命令——加载对应子技能并按其「工具安装与配置」章节引导用户，或询问是否跳过该步骤。

### 安装依赖的子技能

本技能依赖的子技能需先安装到位，推荐使用 [`skills`](https://github.com/vercel-labs/skills) CLI 按需安装（无需全局安装）：

```bash
# 安装本技能依赖的全部子技能（含本技能自身）
npx skills add https://github.com/cyrasafia/zl-skills \
  --skill publish-prd \
  --skill codeup-repo \
  --skill write-prd \
  --skill publish-to-feishu

# 或安装到全局（所有项目可用）
npx skills add https://github.com/cyrasafia/zl-skills \
  --skill codeup-repo --skill write-prd --skill publish-to-feishu -g
```

仅缺某个子技能时也可单独安装，例如：

```bash
npx skills add https://github.com/cyrasafia/zl-skills --skill codeup-repo
```

安装完成后，AI 助手会根据技能描述自动匹配并加载，无需手动配置。更多用法见 `npx skills --help`。

## publish-rp 安装说明

`publish-rp` 是独立 CLI，将本地原型目录镜像同步到 `prototype-master` 仓库并推送，生成在线预览地址。

- 仓库：https://github.com/cyrasafia/publish-rp
- 环境要求：[Bun](https://bun.sh) 1.1+、系统已安装 `rsync`、`git`（push 需 SSH 可访问 Codeup）

```bash
git clone https://github.com/cyrasafia/publish-rp.git
cd publish-rp
bun link
```

之后在任意目录可用 `publish-rp`。取消链接：`bun unlink publish-rp`。

首次配置（指定 rp_home，即 prototype-master 仓库本地路径）：

```bash
publish-rp config --rp-home "/home/<user>/协作工作区/原型"
```

配置文件查找顺序：

1. 当前目录下的 `publish-rp-config.json`（若存在）
2. 否则 `~/.config/publish-rp/publish-rp-config.json`

示例见仓库内 `publish-rp-config.example.json`。

## 执行步骤

按顺序执行以下四个步骤。每个步骤执行前先进行条件检查，不满足时**跳过并记录**，不中断后续流程。全部完成后输出**执行结果摘要**。

---

### 步骤一：Git 初始化与远程关联

**目标**：确保项目已初始化 Git 且已关联 Codeup 远程仓库。

**委托给 `codeup-repo` 技能**：先做 Git 初始化检查（`git rev-parse --is-inside-work-tree`，否则 `git init`），再检查 `git remote get-url origin`。无远程时，按 `codeup-repo` 的仓库创建 + SSH 关联流程执行（确定 repo 名称 → `codeup repo create` → 取 `sshUrlToRepo` → `git remote add origin` → `git push -u`）。

repo 名称约定：优先用项目根目录文件夹名称；含中文/特殊字符/空格时，按项目内容选合适英文名（如 `hotel-detail-page`）。

**注意**：`git push` 需访问 SSH 密钥，必须在沙箱外执行。

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

若 `publish-rp` 未安装，按上文「publish-rp 安装说明」引导用户安装或询问是否跳过。安装后执行：

```bash
publish-rp ./prototype
```

- 记录输出中的**预览链接**（通常为 `http://rp.histar.com/<repo-name>/`），步骤三需要用到。
- 若需自定义 slug 或提交说明，追加 `--slug <name>` / `--message "<text>"`；预览可用 `--dry-run`。

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

#### 3.2 补全 PRD（委托 write-prd 技能）

对 `prd/` 下**所有** `.md` 文件逐个执行：

1. **补全 NOTE**：使用 **`write-prd`** 技能的逻辑，为每个 PRD 补全标题下方的 `> [!NOTE]` 文档仓库说明（如果有远程仓库的话）。
2. **补全原型预览链接**：如果步骤二成功获取到了预览链接，在每个 PRD 的「参考链接」章节中添加原型预览链接。若没有「参考链接」章节则新增。

#### 3.3 发布到飞书（委托 publish-to-feishu 技能）

使用 **`publish-to-feishu`** 技能，逐个发布 `prd/` 下的 `.md` 文件。具体命令、安装检查、配置与错误处理均按该技能说明执行：

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

## 常见问题

| 现象 | 处理 |
|------|------|
| `codeup` 命令不存在 | 加载 codeup-repo 技能，按其安装说明引导，或跳过步骤一 |
| `publish-rp` 命令不存在 | 按上文「publish-rp 安装说明」引导安装，或跳过步骤二 |
| `publish-to-feishu` 命令不存在 | 加载 publish-to-feishu 技能，按其安装说明引导，或跳过步骤三 |
| `git push` 失败（Permission denied） | SSH 密钥未配置，提示在沙箱外执行 |
| 飞书发布失败（token 过期） | 加载 publish-to-feishu 技能，按其说明重新 `auth` |
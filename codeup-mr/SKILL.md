---
name: codeup-mr
description: Manages Aliyun Codeup merge requests via `codeup` CLI—create, review, merge, list, and update. Default merge uses squash with source branch deletion and commit message `Merge #<id> <中文摘要>`. Use when the user asks to 创建/提/合并/评审 MR, merge request, Codeup CR, 合并请求, or manage open MRs on Codeup.
---

# Codeup MR 管理

通过 **`codeup mr`** 管理云效合并请求。CLI 配置与 PAT 见 [codeup-repo](../codeup-repo/SKILL.md)。

## 语言约定

- **MR 标题、描述、合并提交摘要默认用中文**（除非用户明确要求英文）。
- 标题格式：`类型: 简述`（如 `feat: 新增批量测试脚本`、`文档：新增 best-practices 文件夹`）。

## 创建 MR

### 前置

1. 在 git 仓库内，源分支已 `git push` 到 Codeup（`git push` 需 `required_permissions: ["all"]`）。
2. `codeup config get` 确认 token 含 **合并请求 · 读写**。
3. `codeup mr list --state opened` 查是否已有同分支 MR，避免重复创建。

### `--wip`

| 场景 | `--wip` |
|------|---------|
| 用户要求草稿 / 开发中 / 暂不评审 | 加 |
| 正式提审、功能已完成 | **不加** |

- Codeup 用 `--wip` 在标题前加 `[wip]`，**不要**手写 `[Draft]`。
- 转正式：`codeup mr update <repo> <localId> --no-wip`。

### 目标分支约定

**除非用户明确指定目标分支，否则一律以仓库默认分支（即远程 HEAD 指向的分支，通常为 `master` 或 `main`）作为目标分支（`<target>`）。**

### 流程

```
1. git log <target>..HEAD 与 git diff <target>...HEAD 撰写变更
2. codeup mr create -t "..." -d "..."
3. 从 --json 输出取 localId、detailUrl 反馈用户
```

### 描述模板

```markdown
## 变更说明

- （主要改动，中文）

## 测试计划

- [ ] （可执行验证步骤）
```

### 示例

```bash
git push -u origin "$(git branch --show-current)"

codeup mr create \
  -t "feat: 新增批量 Agent 测试脚本" \
  -d "$(cat <<'EOF'
## 变更说明

- 新增 scripts/batch_agent_test.py

## 测试计划

- [ ] python scripts/batch_agent_test.py --dry-run
EOF
)" --json
```

显式指定仓库/分支：`codeup mr create zlxt/agent/foo --source-branch feat/x --target-branch develop -t "..."`

## 评审 MR

合并前确认 MR 可合并：

```bash
codeup mr get <repo> <localId> --json
# 关注：status、conflictCheckStatus=NO_CONFLICT、allRequirementsPass
```

批准：

```bash
codeup mr review <repo> <localId> --approve -c "LGTM"
```

拒绝：`codeup mr review <repo> <localId> --reject -c "原因说明"`

## 合并 MR

### 默认策略（必须遵守）

| 项 | 默认值 |
|----|--------|
| 合并方式 | **`--type squash`** |
| 源分支 | **合并后删除**：`--remove-source-branch` |
| 提交信息 | **`-m "Merge #<localId> <中文摘要>"`** |

用户未明确要求其他合并方式时，**不要**用 `no-fast-forward` 或 `rebase`。

### 合并提交名

格式（固定）：

```
Merge #<MR编号> <MR内容摘要>
```

- **MR 编号**：`localId`（如 `26`、`27`）。
- **MR 内容摘要**：从 MR **标题**提取的中文简述，去掉类型前缀（`feat:`、`fix:`、`chore:`、`docs:`、`文档：` 等），保留核心中文描述。
- 摘要必须是中文；标题若混有英文类型前缀，只去掉前缀，不整句翻译。

**示例（与本仓库历史一致）：**

| MR 标题 | 合并提交 `-m` |
|---------|----------------|
| `文档：新增 best-practices 文件夹` | `Merge #26 新增 best-practices 文件夹` |
| `feat: 新增批量 Agent 测试脚本与 YAML 用例` | `Merge #27 新增批量 Agent 测试脚本与 YAML 用例` |
| `fix: fix typo`（需结合上下文） | `Merge #24 修复master ab test 配置的笔误` |
| `feat: master agent 拆分AB test chore: ...` | `Merge #23 master agent拆分AB test` |

标题过于笼统时，从 MR 描述「变更说明」提炼一句中文摘要，仍保持 `Merge #N ...` 格式。

### 合并流程

```
1. codeup mr get <repo> <localId> --json   # 无冲突、可合并
2. 据标题撰写中文摘要，构造 -m "Merge #<id> <摘要>"
3. codeup mr review <repo> <localId> --approve -c "LGTM"   # 若尚未评审
4. codeup mr merge <repo> <localId> \
     --type squash \
     --remove-source-branch \
     -m "Merge #<id> <中文摘要>" \
     --json
5. 反馈 mergedRevision、detailUrl；提示本地 git pull origin <target>
```

### 合并示例

```bash
codeup mr review zlxt/agent/plan-travel 27 --approve -c "LGTM"

codeup mr merge zlxt/agent/plan-travel 27 \
  --type squash \
  --remove-source-branch \
  -m "Merge #27 新增批量 Agent 测试脚本与 YAML 用例" \
  --json
```

### 批量合并

按 **localId 从小到大**（或创建时间先后）逐个合并，每合并一个后若下一个 `behind > 0`，先 `codeup mr get` 确认仍 `NO_CONFLICT` 再合并。

## 查询与更新

```bash
codeup mr list --state opened
codeup mr list --state merged --all
codeup mr get <repo> <localId> --json
codeup mr update <repo> <localId> -t "新标题" -d "新描述"
```

在已配置 Codeup SSH `origin` 的目录可省略 `<repo>`。

## 常见错误

| 现象 | 处理 |
|------|------|
| 403 合并请求 | PAT 勾选 **合并请求 · 读写** |
| 冲突 | 源分支 rebase/merge 目标分支后 push，再合并 |
| 源分支 = 目标分支 | 切功能分支或 `--source-branch` |
| 重复 MR | `codeup mr list --state opened` |
| 远程禁止 force push | 用 merge 同步分支，勿 `git push --force` |

## 关联技能

- 仓库与 remote 配置：[codeup-repo](../codeup-repo/SKILL.md)

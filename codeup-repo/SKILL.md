---
name: codeup-repo
description: Manages Aliyun Yunxiao Codeup repositories via the `codeup` CLI (repo list/get/create/update, config). MR create/review/merge 见 codeup-mr 技能. Links local git repos to Codeup remotes using SSH only. Use when the user mentions Codeup/云效代码库, creating or querying repos, `codeup` commands, or associating a local project with a remote Codeup repository.
---

# Codeup 仓库管理（codeup CLI）

通过全局命令 **`codeup`** 操作云效 Codeup（中心版 OpenAPI）。**不要**用网页 API 手写 curl，除非 CLI 能力不足。

## 命令结构

```
codeup config          # 配置（顶层）
codeup repo list|get|create|update
codeup mr create|list|get|update|review|merge
```

## 工具安装与配置

### 安装

要求 Node.js 18+。源码在 [codeup-cli 仓库](https://github.com/cyrasafia/codeup-cli)，未安装时：

```bash
git clone https://github.com/cyrasafia/codeup-cli
cd codeup-cli
npm install
npm link        # 将 codeup 命令安装到全局 PATH
```

确认安装成功：`codeup --help`

### 配置（三项必填，环境变量优先于配置文件）

| 配置项 | 环境变量 | 说明 |
|--------|----------|------|
| `domain` | `CODEUP_DOMAIN` | OpenAPI 接入点，如 `openapi-rdc.aliyuncs.com`（**不是** `codeup.aliyun.com`） |
| `organizationId` | `CODEUP_ORG_ID` | 组织 ID（中心版必需） |
| `token` | `CODEUP_TOKEN` | 个人访问令牌（`pt-xxxx...`） |

可选：`defaultNamespaceId`（默认父路径，数字 ID 或 `zlxt/zl-product` 形式路径）→ 环境变量 `CODEUP_DEFAULT_NAMESPACE`。

**方式一：环境变量**

```bash
export CODEUP_DOMAIN=openapi-rdc.aliyuncs.com
export CODEUP_ORG_ID=<组织ID>
export CODEUP_TOKEN=<pt-令牌>
export CODEUP_DEFAULT_NAMESPACE=zlxt/zl-product   # 可选
```

**方式二：写入配置文件**

```bash
codeup config set domain openapi-rdc.aliyuncs.com
codeup config set org-id <组织ID>
codeup config set token <pt-令牌>
codeup config set default-namespace-id zlxt/zl-product   # 可选

codeup config get    # 查看配置
codeup config path   # 配置文件位置
```

### PAT 获取与权限

1. 登录 [云效工作台](https://devops.aliyun.com/) → 右上角 **用户头像** → **个人设置** → **个人访问令牌** → **新建访问令牌**。
2. 填写名称、到期时间，在 **选择权限** 中勾选对应权限（遵循最小权限）：
   - `codeup repo list/get`：**代码管理 · 代码仓库 · 只读**
   - `codeup repo create/update`：**代码管理 · 代码仓库 · 读写**
   - `codeup mr list/get`：**代码管理 · 合并请求 · 只读**
   - `codeup mr create/update/review/merge`：**代码管理 · 合并请求 · 读写**
   - 用路径（如 `zlxt/zl-product`）作父分组：另需 **代码管理 · 代码组 · 只读**（仅用数字 ID 可不勾选）
3. 创建成功后**立刻复制保存**：云效只在创建时显示一次，之后无法查看原文。

组织 ID 获取：云效 → 用户头像 → 个人设置 → **已加入组织** 中查看。

脚本化时加 **`--json`** 解析输出。

## 仓库查询

```bash
codeup repo list --search <keyword>
codeup repo list --all
codeup repo get <repoId>                # 数字 ID 或 namespace/path
codeup repo get <ref> --json            # 取 sshUrlToRepo、webUrl、id 等
```

## 仓库创建

```bash
codeup repo create <name> [-d "描述"] [--path <slug>] [--visibility private|internal]
```

默认行为：

- **`visibility`**：`internal`
- **`readMeType`**：不传（空库，无平台自动 README）
- **父路径**：若配置了 `defaultNamespaceId`，在其下创建；否则组织根

覆盖示例：

```bash
codeup repo create my-tool --namespace-id zlxt/zl-product
codeup repo create my-tool --org-root
codeup repo create my-tool --readme USER_GUIDE
```

## 仓库更新

至少传一个字段：

```bash
codeup repo update <repoId> --description "..."
codeup repo update <repoId> --visibility internal --default-branch main
```

## 合并请求（MR）

完整流程（创建 / 评审 / 合并、squash 策略、合并提交命名）见 **[codeup-mr](../codeup-mr/SKILL.md)**。

PAT 需 **合并请求 · 只读**（查询）或 **读写**（创建/评审/合并）。

```bash
# 在已配置 Codeup SSH remote 的 git 仓库内
codeup mr create -t "feat: xxx"
codeup mr list --state opened --json
codeup mr get <repoId> <localId> --json
codeup mr review <repoId> <localId> --approve -c "LGTM"
# 合并默认：squash + 删源分支 + Merge #N 中文摘要（见 codeup-mr）
codeup mr merge <repoId> <localId> --type squash --remove-source-branch -m "Merge #N 中文摘要"
```

显式指定仓库与分支：

```bash
codeup mr create zlxt/zl-product/foo \
  --source-branch feature/x --target-branch main -t "标题"
codeup mr list zlxt/zl-product/foo --state opened
```

## 本地 Git 关联远程（必须 SSH）

**规则：设置 `git remote` 时一律使用 `sshUrlToRepo`，禁止使用 `httpUrlToRepo` / HTTPS。**

### 流程 A：先建远程再推送本地

```bash
codeup repo create <name> --json
# 从 JSON 取 sshUrlToRepo

git remote add origin git@codeup.aliyun.com:zlxt/zl-product/foo.git
git push -u origin main
```

### 流程 B：远程已存在

```bash
codeup repo get zlxt/zl-product/foo --json
git remote add origin <sshUrlToRepo>
git push -u origin main
```

### 注意

- 推送需本机已配置 Codeup SSH 公钥；失败时提示用户检查云效 SSH 配置，**不要**改用 HTTPS 作为变通。
- `git push` 在沙箱外执行时需 `required_permissions: ["all"]`（SSH 密钥访问）。
- 默认分支名以远程为准；若远程为 `master` 而本地为 `main`，按实际情况 `git push -u origin main` 或通过 `codeup repo update` 将默认分支改为 `main`。

## 不在 CLI 范围内

删除 / 归档 / 转移 / 模板库列表、关闭 MR：网页操作或后续扩展。日常 clone/pull/push 用 **`git`** + SSH remote。

## 故障速查

| 现象 | 处理 |
|------|------|
| 列表为空 / HTML 登录页 | `domain` 应为 OpenAPI 接入点（如 `openapi-rdc.aliyuncs.com`） |
| 路径解析 403 | PAT 需 **代码组 · 只读**；或改用数字 `namespaceId` |
| `git push` 要用户名密码 | remote 必须是 **SSH** URL，不是 HTTPS |

详细配置与权限说明见 codeup-cli 仓库 [README.md](https://github.com/cyrasafia/codeup-cli)。

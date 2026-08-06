# zl-skills

团队内部 Agent 技能集合，用于扩展 AI 编码助手的能力，覆盖阿里云 Codeup 仓库与 MR 管理、飞书项目需求创建、PRD 编写与发布等场景。

通用技能（如网页搜索、Apifox 等）请直接使用社区维护的版本，不再收录在本仓库。

## 协作约定

- 本仓库为团队内部共享技能库，新增技能需提交到本仓库，由团队共同维护。
- 技能变更会影响团队成员的 AI 助手行为，修改已有技能前请与相关成员沟通确认。

## 技能列表

| 技能                                                                      | 说明                          |
| ----------------------------------------------------------------------- | --------------------------- |
| [codeup-mr](codeup-mr/)                                                 | 阿里云 Codeup 合并请求管理（创建/评审/合并） |
| [codeup-repo](codeup-repo/)                                             | 阿里云 Codeup 仓库管理（创建/查询/关联）   |
| [feishu-project-create-requirement](feishu-project-create-requirement/) | 从 PRD 在飞书项目中创建需求工作项         |
| [publish-prd](publish-prd/)                                             | 一键发布项目到飞书文档、原型服务器和 Git 仓库   |
| [publish-to-feishu](publish-to-feishu/)                                 | 将 Markdown 文档发布到飞书          |
| [write-prd](write-prd/)                                                 | 从原型或草稿编写/扩充 PRD 需求文档        |

## 安装

推荐使用 [`skills`](https://github.com/vercel-labs/skills) CLI（无需全局安装）按需安装技能：

```bash
# 列出本仓库可用技能
npx skills add https://github.com/cyrasafia/zl-skills --list

# 安装单个技能到当前项目（默认符号链接到各 agent 的 skills 目录）
npx skills add https://github.com/cyrasafia/zl-skills --skill <skill-name>

# 安装到全局（所有项目可用）
npx skills add https://github.com/cyrasafia/zl-skills --skill <skill-name> -g

# 一次安装多个技能
npx skills add https://github.com/cyrasafia/zl-skills --skill codeup-mr --skill write-prd

# 非交互式安装（适合 CI/CD）
npx skills add https://github.com/cyrasafia/zl-skills --skill <skill-name> -y
```

常用选项：

| 选项                     | 说明                                            |
| ---------------------- | --------------------------------------------- |
| `-g, --global`         | 安装到用户目录而非当前项目                                 |
| `-s, --skill <names>`  | 指定要安装的技能名，可多次使用；用 `'*'` 安装全部                  |
| `-a, --agent <agents>` | 指定目标 agent（如 `opencode`、`claude-code`），默认交互选择 |
| `-l, --list`           | 仅列出可用技能，不安装                                   |
| `--copy`               | 复制文件而非符号链接（不支持符号链接时使用）                        |
| `-y, --yes`            | 跳过所有确认提示                                      |

安装完成后，OpenCode 等助手会根据技能描述自动匹配并加载对应指令，无需手动配置。

> 如需了解更多用法（搜索、更新、移除技能等），运行 `npx skills --help`。

## 目录结构

```
zl-skills/
├── <skill-name>/
│   ├── SKILL.md              # 技能定义（必含 frontmatter）
│   └── references/           # 技能引用文档（可选）
```

---
name: feishu-project-create-requirement
description: Create story work items in Feishu Project from PRD markdown using feishu-project CLI
---
# 飞书项目创建需求规则

当用户说类似"把 xxx.md 在飞书项目中创建一个需求"时，使用 `feishu-project` 命令行工具创建工作项。

## 执行步骤

1. **读取 PRD 文件**，提取：
   - 需求名称：取 Markdown 一级标题（`# xxx`）
   - 飞书文档地址：从文件末尾注释 `<!-- feishu-doc-id: xxx -->` 拼接 `https://www.feishu.cn/docx/xxx`，或从正文"参考链接"中已有的飞书链接提取
   - 需求描述：根据 PRD 内容总结，简要描述需求背景和主要改动点（2-5 句话，用 Markdown 格式）

2. **角色处理规则（必须执行，优先级从高到低）**：
   - 若用户**明确指定了产品经理**，则使用用户指定的人：
     - `--role role_ad4e41=<用户指定的人>`
   - 若用户**未明确指定产品经理**，则必须自动补充：
     - `--role role_ad4e41=付晨昱`
   - 前端/后端/QA 等**非产品经理角色**仅按用户明确要求添加，不额外猜测。

3. **执行创建命令**：
   ```bash
   feishu-project create \
     --type-key story \
     --name "<需求名称>" \
     --description "<需求描述>" \
     --field wiki="<飞书文档地址>" \
     <产品经理角色参数（按步骤2确定）> \
     <用户要求的其他参数>

4. **字段速查**：
   - `--priority` 优先级：P0/P1/P2/待定
   - `--field wiki=<url>` 需求文档地址
   - `--field description=<text>` 需求描述（也可用 `--description`）
   - `--role role_ad4e41=<用户>` 产品经理
   - `--role role_7838f4=<用户>` 前端开发
   - `--role role_0ef657=<用户>` 后端开发
   - `--role role_500f1b=<用户>` QA
   - `--field field_752faf=前端` / `后端` / `数据算法` 涉及到的端（可多次指定）
   - `--field field_60d56b=<url>` 设计稿链接
   - `--field field_37a269=<url>` 技术文档链接

5. 如果用户指定了额外要求（如优先级、指派开发、涉及端等），通过对应的 `--field` 或 `--role` 参数追加到命令中。

6. 执行完成后，向用户报告创建结果（工作项 ID、链接）。

# opencode-skills

OpenCode 技能集合，用于扩展 AI 编码助手的能力，覆盖 API 测试、代码仓库管理、需求文档编写与发布等场景。

## 技能列表

| 技能 | 说明 |
|------|------|
| [apifox-cli](apifox-cli/) | 通过 Apifox CLI 管理接口自动化测试与项目资源 |
| [codeup-mr](codeup-mr/) | 阿里云 Codeup 合并请求管理（创建/评审/合并） |
| [codeup-repo](codeup-repo/) | 阿里云 Codeup 仓库管理（创建/查询/关联） |
| [feishu-project-create-requirement](feishu-project-create-requirement/) | 从 PRD 在飞书项目中创建需求工作项 |
| [publish-prd](publish-prd/) | 一键发布项目到飞书文档、原型服务器和 Git 仓库 |
| [publish-to-feishu](publish-to-feishu/) | 将 Markdown 文档发布到飞书 |
| [tavily-best-practices](tavily-best-practices/) | Tavily 集成最佳实践参考文档 |
| [tavily-cli](tavily-cli/) | Tavily CLI 网页搜索与内容提取 |
| [tavily-crawl](tavily-crawl/) | 网站多页面爬取 |
| [tavily-dynamic-search](tavily-dynamic-search/) | 动态网页搜索（上下文隔离） |
| [tavily-extract](tavily-extract/) | 从 URL 提取清洁 Markdown 内容 |
| [tavily-map](tavily-map/) | 网站 URL 发现与结构映射 |
| [tavily-research](tavily-research/) | AI 驱动的深度研究与报告生成 |
| [tavily-search](tavily-search/) | LLM 优化的网页搜索 |
| [write-prd](write-prd/) | 从原型或草稿编写/扩充 PRD 需求文档 |

## 使用方式

将本仓库作为 OpenCode 技能目录（`~/.config/opencode/skills`），AI 助手会根据技能描述自动匹配并加载对应指令。

## 目录结构

```
skills/
├── <skill-name>/
│   ├── SKILL.md              # 技能定义（必含 frontmatter）
│   └── references/           # 技能引用文档（可选）
```

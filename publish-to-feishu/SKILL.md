---
name: publish-to-feishu
description: Run publish-to-feishu script from natural language requests
---
# Publish To Feishu Command Rule

When the user asks in Chinese like "把 xxx.md 发布到飞书", treat it as a command-execution request.

- Parse the markdown path from user text (supports relative paths like `sample/酒店详情页.md`).
- If path is missing, ask for the markdown file path.
- Execute in project root (or from a cwd where `publish-config.json` is found), preferring:
  - `publish-to-feishu "<markdown_path>"` when the CLI is on PATH (`npm link` / `npm i -g`), else
  - `node publish-to-feishu.js "<markdown_path>"` (only when that file exists in the repo)
- Only add flags when user explicitly asks:
  - `--doc-id <id>`
  - `--folder-token <token>`
  - `--dry-run`
- After command finishes, report key output:
  - target doc URL
  - image upload count
  - any API/token errors

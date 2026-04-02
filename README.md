# codex-transcripts

把 Codex Desktop 的本地 `.jsonl` 会话记录转换成适合浏览的 HTML 聊天记录页面，整体交互和结构参考 `claude-code-transcripts-main`，但解析逻辑适配 Codex 客户端真实 session 格式。

## 安装

```bash
uv sync
```

也可以直接查看帮助：

```bash
uv run codex-transcripts --help
```

## 用法

直接从本地最近会话里挑选：

```bash
uv run codex-transcripts
uv run codex-transcripts local
```

转换指定会话文件：

```bash
uv run codex-transcripts json ~/.codex/sessions/2026/03/31/rollout-xxxx.jsonl -o ./output
uv run codex-transcripts json ~/.codex/sessions/2026/03/31/rollout-xxxx.jsonl --open
```

也支持从 URL 直接拉取 JSON/JSONL：

```bash
uv run codex-transcripts json https://example.com/session.jsonl -o ./output
```

上传到 GitHub Gist 并拿到预览链接：

```bash
uv run codex-transcripts json ~/.codex/sessions/2026/03/31/rollout-xxxx.jsonl --gist
uv run codex-transcripts --gist
```

批量生成归档：

```bash
uv run codex-transcripts all -o ./codex-archive
uv run codex-transcripts all --dry-run
```

## 命令

- `local` / 默认命令：从 `~/.codex/sessions` 读取最近本地会话
- `json`：转换指定 `.json` / `.jsonl` 文件，或远程 URL
- `all`：转换本地全部会话，生成可浏览 archive

Codex 没有对应 Claude 项目里的 `web` 会话 API 导入能力，所以这里没有实现 `web` 子命令。

## 输出选项

`local` 和 `json` 支持这些选项：

- `-o, --output DIRECTORY`：输出目录；默认写到临时目录
- `-a, --output-auto`：自动按 session 文件名创建子目录
- `--open`：生成后打开 `index.html`
- `--gist`：上传 HTML 到 GitHub Gist 并输出 `gisthost.github.io` 预览链接
- `--json`：把原始 session 文件一起复制到输出目录

## 批量归档

`all` 命令会生成三层结构：

- 总索引页：列出所有 workspace
- 每个 workspace 的索引页：列出该 workspace 下的 sessions
- 每个 session 的 transcript 页面

`all` 支持这些选项：

- `--source DIRECTORY`：源目录，默认 `~/.codex/sessions`
- `-o, --output DIRECTORY`：输出目录，默认 `./codex-archive`
- `--include-agents`：包含 agent / subagent sessions
- `--dry-run`：只预览会转换哪些内容，不写文件
- `--open`：生成后打开 archive 首页
- `-q, --quiet`：安静模式，只输出错误

## 当前支持

- `local`：交互式选择最近本地会话
- `json`：转换本地或远程 `.json` / `.jsonl`
- `all`：按 workspace 分组生成 archive
- `-a / --output-auto`
- `--json`
- `--open`
- `--gist`：上传生成的 HTML 到 GitHub Gist，并输出 `gisthost.github.io` 预览链接
- 多页转写：按 Codex turn 分页
- Gist 预览修复：自动注入相对链接修复脚本，保证分页和锚点在 `gisthost.github.io` 下可用

## 已知限制

- 目前只处理 Codex 本地 `.jsonl` 会话，不读 SQLite 历史库
- `reasoning.encrypted_content` 不会解密，只会显示可见 summary
- 一些 `event_msg` 是 `response_item` 的重复镜像，页面里会去重并优先展示最终可读消息
- `--gist` 依赖 `gh` 已安装且完成 `gh auth login`
- `web` 子命令未实现，因为目前没有对应的 Codex Web session 导入接口
